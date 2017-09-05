---
title: "Localizing enum values in an enterprise WPF application"
date: 2017-09-04T16:46:34+02:00
tags: [ "english-post", "c-sharp",  "wpf", "xaml", "enum"]
---

In a WPF application, you often work with collections of data. This data is then visually represented in list-types (ComboBox, ListView etc.) or data grids. A common design approach may look like this:

* There is a `Model`, which represents a single data record (E.g. a `UserModel`, including atomic data like a name etc.).
* A `View`, which contains a list-type or a data grid. The collection of `UserModel`-objects is then bound to the corresponding `ItemsSource` of the visual element.
* A `ViewModel`, which contains a collection of `UserModel`-objects.

If you are not familiar with this concept, you should read about the [MVVM-Pattern](https://msdn.microsoft.com/en-us/library/hh848246.aspx) first, as I will not go into more detail in this article.

To focus more on the problem, lets keep the `UserModel` as simple as possible.

```csharp
public enum Gender
{
    Female, Male // Let's not start a gender debate here... ;-)
}

public class UserModel
{
    public Gender Gender { get; }
    
    public string Name { get; }

    public UserModel(Gender gender, string name)
    {
        this.Gender = gender;
        this.Name = name;
    }
}
```

A `UserModel` contains a name as string and a gender as enum-type. The required code of our `ViewModel` holds a list of `UserModel`-objects inside a generic `ObservableCollection<>`.

```csharp
private ObservableCollection<UserModel> users;

public ObservableCollection<UserModel> Users
{
    get => this.users ?? (this.users = new ObservableCollection<UserModel>
    {
        // Some sample data
        new UserModel(Gender.Female, "Anna"),
        new UserModel(Gender.Male, "Scott")
    });
    private set
    {
        this.users = value;
        this.OnPropertyChanged(nameof(this.Users));
    }
}
```

When this collection is now bound to a data grid, the result will look like this:

<center>
![datagrid without localization](/img/localize-enum-values/datagrid-without-localization.png)
</center>

When developing a multilingual application, the problem of this approach quickly becomes obvious. In the English-speaking countries there are no problems with the presented data, as the user is correctly shown "Male" and "Female" as gender. But what options do we have if we want to localize these enum values with as little code as possible?


## Solution 1: A naive approach

A naive approach would be to add another property to the `UserModel` that directly converts the enum value into a string.

```csharp
public class UserModel
{
    public Gender Gender { get; }
    
    public string GenderAsString =>
        this.Gender == Gender.Female
            ? LocalizationStrings.GenderFemale;
            : LocalizationStrings.GenderMale;
    
    public string Name { get; }

    public UserModel(Gender gender, string name)
    {
        this.Gender = gender;
        this.Name = name;
    }
}
```

If we now manually specify the columns in the XAML file and apply appropriate column-headers including localization, we get the desired result (in German).

<center>
![datagrid with localization naive approach](/img/localize-enum-values/datagrid-with-localization-naive.png)
</center>

The corresponding XAML-code:

```xml
<DataGrid AutoGenerateColumns="False"
          ItemsSource="{Binding Users}">
    <DataGrid.Columns>
        <DataGridTextColumn Binding="{Binding GenderAsString}"
                            Header="{x:Static loc:LocalizationStrings.Gender}" />
        <DataGridTextColumn Binding="{Binding Name}"
                            Header="{x:Static loc:LocalizationStrings.Name}" />
    </DataGrid.Columns>
</DataGrid>
```

The main disadvantage of this approach is that we no longer use our model class for pure data storage. The property `GenderAsString` is used exclusively in the `View`. No business logic will (hopefully) ever access this property. Therefore, having such a "redundant" property in a model class, is a bad design.

## Solution 2: Encapsulation of the Model

Instead of binding directly to a collection of `Model`-objects, we can bind to a collection another `ViewModel`, whose task is to encapsulate the model. Thus our `UserModel` remains a pure data storage class. This `ViewModel` will hold a private instance of our `Model` and offers additional `View`-specific properties. Such a `ViewModel` could look like this:

```csharp
public class UserViewModel
{
    private readonly UserModel model;

    public UserViewModel(UserModel model)
    {
        this.model = model;
    }

    public string Name => this.model.Name;

    public string GenderAsString =>
        this.model.Gender == Gender.Female
            ? LocalizationStrings.GenderFemale;
            : LocalizationStrings.GenderMale;
}
```

With this approach we get the exact result as shown in solution 1. However, due to encapsulation we have a better separation of concerns. Nevertheless, the same problem exists as in solution 1: The property `GenderAsString` is used exclusively in the `View`. Even though the actual purpose of a `ViewModel` is to create a link between the `View` and the `model`, there is actually *no other need* for our `ViewModel` to hold this data. So what other options do we have?

## Solution 3: Using an IValueConverter

```csharp
public class GenderToStringConverter : IValueConverter
{
    public object Convert(
        object value, 
        Type targetType, 
        object parameter, 
        CultureInfo culture)
    {
        var result = string.Empty;
        if (value is Gender gender)
        {
            result = gender == Gender.Female 
                ? LocalizationStrings.GenderFemale;
                : LocalizationStrings.GenderMale;
        }
        return result;
    }
    
    // ConvertBack() is not required for our sample
    // ...
}
```

Having such an `IValueConverter`, which takes the `Gender`-type as input and returns the corresponding string, will solve our problem as well. Due to this approach, we don't have any unnecessary code in the `Model` or `ViewModel`. The `Gender`-value is bound to the column and the binding gets the required converter as parameter:

```xml
<Grid>
    <Grid.Resources>
        <local:GenderToStringConverter x:Key="GenderToStringConverter" />
    </Grid.Resources>
    <DataGrid AutoGenerateColumns="False"
              ItemsSource="{Binding Users}">
        <DataGrid.Columns>
            <DataGridTextColumn Binding="{Binding Gender,
                                                  Converter={StaticResource GenderToStringConverter}}"
                                Header="{x:Static loc:LocalizationStrings.Gender}" />
            <DataGridTextColumn Binding="{Binding Name}"
                                Header="{x:Static loc:LocalizationStrings.Name}" />
        </DataGrid.Columns>
    </DataGrid>
</Grid>
```

However, one big limitation remains. We now have an `IValueConverter` which only accepts values of the type `Gender`. If you think of enterprise software, you would have to implement the almost identical `IValueConverter` for each individual enum type. This results in unnecessary code duplication. The final solution therefore consists of a generic solution, which I think is the best approach.

## Solution 4: Using an attribute to create generic IValueConverter

So we need a generic way to avoid code duplication. Therefore, we introduce a small attribute class, which gets a localization key as constructor parameter:

```csharp
[AttributeUsage(AttributeTargets.Field)]
public class LocalizationKeyAttribute : Attribute
{
    public string LocKey { get; }

    public LocalizationKeyAttribute(string locKey)
    {
        this.LocKey = locKey;
    }
}
```

Now we can decorate our enum values with this attribute.

```csharp
public enum Gender
{
    [LocalizationKey("GenderFemale")]
    Female,

    [LocalizationKey("GenderMale")]
    Male
}
```

Through this decoration we can now implement a generic `IValueConverter` which finds the appropriate localized resource for any enum type and enum value.

```csharp
public class LocalizedEnumConverter : IValueConverter
{
    public object Convert(
        object value, 
        Type targetType, 
        object parameter, 
        CultureInfo culture)
    {
        var result = string.Empty;
        if (value is Enum enumValue)
        {
            var type = enumValue.GetType();
            
            // Look for our 'LocalizationKeyAttribute' in the field's custom attributes
            var field = type.GetField(enumValue.ToString());
            var key = string.Empty;
            if (field.GetCustomAttributes(typeof(LocalizationKeyAttribute), false) is LocalizationKeyAttribute[] attributes
                && attributes.Length > 0)
            {
                key = attributes[0].LocKey;
            }
            
            result =  string.IsNullOrWhiteSpace(key) 
                ? string.Empty 
                : LocStrings.ResourceManager.GetString(key);
        }
        return result;
    }

    // ConvertBack() ...
}
```

Now we can use this `IValueConverter` for any enum type, as long as the enum values are decorated with the required attribute.

## Summary

The presented solutions illustrate that there are different approaches for localizing enum types depending on the use case. For smaller applications, the implementation from solution 3 may be sufficient for the specific enum type. Anyway you should avoid unnecessary code in the `Model` and `ViewModel`. Therefore, solution 1 and solution 2 should be avoided as well. If you are familiar with value converters and custom attributes, I highly recommend choosing the last solution.

Are you using a completely different approach to localize enum values? If so, I would be happy to hear from it in the comments section below. :-)