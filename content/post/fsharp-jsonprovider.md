---
title: "Der F-Sharp JSON Type Provider"
date: 2017-02-01T00:00:00+02:00
tags: [ "f-sharp", "typeprovider", "api", "json", "golem" ]
---
Im Laufe seiner Karriere befasst sich jeder Softwareentwickler früher oder später mit der Verarbeitung von JSON- oder XML-Dateien. Das Vorgehen ist dabei immer sehr ähnlich: Man analysiert die Struktur des zu verarbeitenden Dokuments, legt sich passende Model-Klassen an und überführt dann die Daten in die konkreten Objekte.

In diesem Artikel möchte ich zeigen, wie man mit dem F# JSON Type Provider die REST-Schnittstelle von dem IT-Magazin [Golem](http://www.golem.de/) konsumieren kann und warum die Verarbeitung der Daten mit F# so komfortabel ist.

## Die Golem REST-Schnittstelle
Die REST-Schnittstelle von Golem bietet eine Vielzahl von Web-Services an, um z.B. Informationen zu den neusten Artikeln zu bekommen oder um in Kategorien nach bestimmten Begriffen zu suchen. Im folgenden Beispiel konzentriere ich mich auf das Anfragen der Neusten- und der Top-Artikel.

Die URL-Synopsis zur Anfrage der Artikel sieht wie folgt aus:

* ht<span>tp://api.golem.de/api/article/*ARTIKELTYP*/limit/?format=resultformat&key=developerkey
* ht<span>tp://api.golem.de/api/article/*ARTIKELTYP*/?format=resultformat&key=developerkey

Als Artikeltyp wird unterschieden zwischen *latest* und *top*. Über ein optionales *limit* kann festgelegt werden, wie viele Artikel maximal angefragt werden sollen. Als Parameter der Anfrage dienen ein Format, in dem das Ergebnis der Anfrage zurückgegeben werden soll (für das Beispiel also JSON) und ein DeveloperKey. Nach der [Registrierung bei Golem](http://api.golem.de/apimanage/index.php) bekommt man zwei Developer-Keys zugesendet. Dies sind zum einen ein Test-Key, mit dem beliebig viele Anfragen geschickt werden können und zum anderen ein Personal-Key, mit dem nur eine gewisse Anzahl von Anfragen pro Stunde erlaubt ist. Bei Anfragen mit dem Test-Key werden immer die gleichen Daten angefragt (Lorem ipsum natürlich 😉), jedoch kann man sich so einen Überblick über die zu verarbeitende Struktur des JSON-Dokuments verschaffen.

## Der F# JSON Type Provider
Über den [JSON Type Provider](http://fsharp.github.io/FSharp.Data/library/JsonProvider.html) bekommt man einen statisch typisierten Zugriff auf JSON-Dokumente. Dadurch müssen für die Verarbeitung eines JSON-Dokuments keine extra Datenhaltungsklassen angelegt werden, da diese durch den Type Provider implizit zur Verfügung gestellt werden. Der Type Provider bekommt ein Beispieldokument als Eingabe, welches die zu verarbeitende Struktur des JSON-Dokuments beinhaltet. Die passenden Typen werden dann vom Type Provider generiert. Was hier ein wenig wie schwarze Magie klingt, fühlt sich bei erster Verwendung auch tatsächlich so an 😁.

## F# Wrapper für Golem Artikel
Da jetzt alle nötigen Komponenten bekannt sind, können wir mit der eigentlichen Implementierung loslegen. Als erstes benötigen wir also für den JSON Type Provider die Struktur des JSON-Dokuments, welches wir bei Anfrage der Artikel zurückbekommen. Dafür rufen wir mit unseren Test-Key die folgende URL auf:

ht<span>tp://api.golem.de/api/article/latest/?format=json&key=GEHEIMERTESTKEY

Das Ergebnis der Anfrage sollte dann in etwa so aussehen:

```json
{
  "data": [
    {
      "articleid": 1,
      "headline": "Lorem ipsum dolor sit amet consectetuer sadipscing elitr sed",
      "abstracttext":
        "Lorem ipsum dolor sit amet, consectetuer sadipscing elitr...",
      "url": "http:\/\/api.golem.de\/",
      "date": 123456789,
      "leadimg": { "url": "http:\/\/api.golem.de\/img\/example_120.png", "height": 90, "width": 120 },
      "comments": 12,
      "images": 5
    }, {
      "articleid": 1,
      "headline": "Lorem ipsum dolor sit amet consectetuer sadipscing elitr sed",
      "abstracttext":
        "Lorem ipsum dolor sit amet, consectetuer sadipscing elitr...",
      "url": "http:\/\/api.golem.de\/",
      "date": 123456789,
      "leadimg": { "url": "http:\/\/api.golem.de\/img\/example_120.png", "height": 90, "width": 120 },
      "comments": 12,
      "images": 5
    }
  ],
  "success": true
}
```

Diesen Inhalt speichern wir nun als *article.json* ab.

Nun endlich der F# Code: Um typsicher zwischen den Neusten- und den Top-Artikeln unterscheiden zu können, legen wir uns für diesen Fall eine Discriminated Union an. Zusätlich implementieren wir eine Funktion, die einen Union Case dieser Discriminated Union entgegen nimmt und eine entsprechende String-Übersetzung zurückgibt.

```fsharp
type ArticleType = | Latest | Top 

let asString = function
    | Latest -> "latest"
    | Top -> "top"
```

Anschließend legen wir einen neuen Typ namens *Article* an und initialisieren diesen als JSON Provider. Der statische Parameter, der an den JSON Provider übergeben wird, beschreibt den relativen Pfad zur Beispiel-JSON-Datei (In unserem Fall also das heruntergeladene Dokument).

```fsharp
type Article = JsonProvider<"article.json">
```

Nun benötigen wir lediglich die anzufragende URL sowie eine Funktion, welche die Anfragen kapselt.

```fsharp
let [<Literal>] BaseUrl = "http://api.golem.de/api/"
let [<Literal>] ArticleUrl = BaseUrl + "article/"

let [<Literal>] RequestSuffix = "/?format=json&key="

let fetchArticles developerKey articleType (limit : option<int>) =
    let articleTypeAsString = articleType |> asString
    let url =
        match limit with
        | Some data -> ArticleUrl + articleTypeAsString + string(data) + RequestSuffix + developerKey
        | None -> ArticleUrl + articleTypeAsString + RequestSuffix + developerKey
    Article.Load url
```

Abhängig davon, ob ein *limit* für die Artikel gesetzt wurde, wird die anzufragende URL zusammengebaut und anschließend über den Type Provider geladen und zurückgegeben (Für die C# Entwickler: das letzte Statement einer F# Funktion wird automatisch zurückgegeben).

Und das war's auch schon. Dies ist der komplette F#-Code, der nötig ist, um Artikel von der golem.de REST-Schnittstelle anzufragen und zu verarbeiten. 

Über die Punktnotation wird typsicher auf das JSON Objekt zugegriffen. Dafür müssen keine Model-Klassen angelegt werden, da dies vom JSON Provider implizit übernommen wird. Ähnliche Type Provider gibt es für [CSV-, XML- oder HTML-Dateien](http://fsharp.github.io/FSharp.Data/index.html). (Oder auch für [MineSweeper](http://pinksquirrellabs.com/post/2014/02/02/The-MineSweeper-Type-Provider.aspx), wenn man denn MineSweeper über die Punktnotation in der IDE via Intellisense spielen möchte 😁)

Das komplette Beispiel habe ich [bei GitHub hochgeladen](https://github.com/oopbase/fsharp-golem). Bei Fragen oder Anregungen, freue ich mich über eure Kommentare.