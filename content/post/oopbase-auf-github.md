---
title: "oopbase auf GitHub Pages"
date: 2017-05-22T00:00:00+02:00
tags: [ "announcement", "github" ]
---
Seit der Umstellung von WordPress auf [Hugo](https://gohugo.io) läuft auf meinem vServer lediglich ein Apache HTTP Server. Es existiert keine Datenbank mehr, da Hugo statischen Inhalt generiert. Nun ist die Überlegung durchaus berechtigt, ob man einen kompletten vServer zum Hosten einer statischen Webseite benötigt 😉. Während des Studiums gab es genug Anwendungsfälle, wofür ein vServer tatsächlich ganz nett war. Nun bin ich aber zu dem Entschluss gekommen, meinen vServer zu kündigen und meinen Blog woanders zu Hosten. Meine Domain (oopbase.de) möchte ich aber weiterhin behalten.

## GitHub Pages

Glücklicherweise lassen sich statische Webseiten kostenlos bei [GitHub Pages](https://pages.github.com/) hosten. Dazu legt man lediglich ein neues Repository nach dem folgenden Schema an:

> *username*.github.io

Jeglicher Inhalt, der in dieses Repository gepusht wird, ist nun via *username*.github.io über den Browser erreichbar. Damit aber eigener statischer Inhalt im Browser gerendert wird, muss zusätzlich eine *.nojekyll*-Datei hochgeladen werden. Diese sorgt dafür, dass das Repository nicht mit [Jekyll](https://jekyllrb.com/) automatisch verarbeitet wird (Man könnte sich natürlich seine statische Webseite auch mit Jekyll generieren lassen, jedoch wollte ich bei Hugo bleiben). Der einzige Nachteil, den man durch das Verzichten auf Jekyll hat, ist, dass man zum Verwalten seiner Webseite nun zwei Repositories benötigt:

* Ein Respository, in dem die Hugo-Anwendung (der Code) liegt (also das *input*-Verzeichnis etc.)
* Sowie ein Repository (*username*.github.io), welches das von Hugo generierte HTML etc. beinhaltet

## Funktionieren eigene Domains?

Ja. In den DNS-Einstellungen für eure Domain müssen dafür zwei "A"-Einträge angelegt werden mit den IP's **192.30.252.153** und **192.30.252.154**. Dies sind die zwei relevanten GitHub-Server, auf die eure Domain weiterleiten soll. Zuletzt muss nur noch eine *CNAME*-Datei in eurem *username*.github.io-Repository angelegt werden. In diese schreibt ihr dann eure Domain rein (bei mir also *oopbase.de*). Dadurch können die GitHub Server euer Repository - und somit eure Webseite - eindeutig identifizieren.

Für mich hat sich dieser Umstieg definitiv gelohnt, da mein vServer zuletzt nichts anderes gemacht hat als GitHub Pages - nämlich statische HTML-Seiten anzeigen.

Zur Vollständigkeit halber (und zum eigenen Nachvollziehen) ist [hier der Link zum Code-Repository meines Blogs](https://github.com/oopbase/blog.oopbase) und [hier der Link zum Repository der generierten Webseite, die ihr gerade betrachtet](https://github.com/oopbase/oopbase.github.io).