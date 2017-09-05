---
title: "Auf interne Typen und Member von außen zugreifen"
date: 2017-03-27T00:00:00+02:00
tags: [ "german-post", "c-sharp", "internal", "internalsvisibleto" ]
---
Durch das `internal`-Schlüsselwort kann auf so deklarierte Typen und Member nur von Dateien aus der selben Assembly zugegriffen werden. Wenn man nun aber Unit-Tests für diese Typen und Member schreiben möchte, befindet man sich häufig in einem separaten Testprojekt. Dadurch fehlt die notwendige Zugriffsberechtigung und die zu testende Funktionalität muss ggf. weiter gekapselt werden.

Abhilfe schafft dabei das `InternalsVisibleTo`-Attribut. Wenn wir also ein Projekt `MeinProjekt` haben und zusätzlich ein weiteres Projekt `MeinProjekt.Test`, so können wir in der `AssemblyInfo.cs` von `MeinProjekt` festlegen, dass interne Typen und Member für ein konkretes Projekt von außen zugreifbar sind. Dazu wird der folgende Code in der `AssemblyInfo.cs` hinzugefügt:

```csharp
[assembly: InternalsVisibleTo("MeinProjekt.Test")]
```

Nun können die internen Typen und Member aus `MeinProjekt` direkt in dem `MeinProjekt.Test` verwendet werden.