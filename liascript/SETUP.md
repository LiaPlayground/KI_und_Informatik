# Technischer Ablauf der Session

## Kurs öffnen

Lokal (empfohlen zum Testen): VS Code Extension "LiaScript Preview",
`kurs.md` öffnen, Vorschau starten.

Veröffentlicht:
```
https://liascript.github.io/course/?<RAW-URL-zu-kurs.md>
```

## Wo läuft der Python-Code?

Über das Template `LiaScript/CodeRunner`: Der Kurs schickt den Code per
WebSocket an einen Ausführungsserver, der ihn mit echtem `python3` startet.
Deshalb funktioniert der Netzwerkzugriff auf bip-schulen.de.

Voreingestellter Server (im Template hinterlegt, getestet):
`wss://ancient-hollows-41316.herokuapp.com/`

Verifiziert am 10.09.2026:
- Python 3.10.12 auf Linux (AWS)
- `bs4` 4.14.3 vorhanden
- `requests` NICHT vorhanden — der Kurs nutzt `urllib.request` (Standardlib)
- Live-Abruf von bip-schulen.de: HTTP 200, 108 KB

## Vor der Stunde

Einen Code-Block einmal ausführen. Der Server schläft bei Nichtnutzung ein,
der erste Start dauert dann bis zu 30 Sekunden ("Waking up execution
server..."). Danach antwortet er zügig.

## Wenn der Server ausfällt

Rückfallebene:

Eigener Server: `git clone https://github.com/LiaScript/CodeRunner`,
dann laut dessen README lokal starten und im Kurs-Header
`window.CodeRunner.init("ws://localhost:4000/")` setzen.

Hinweis: Im Template ist auch `wss://coderunner.informatik.tu-freiberg.de/`
auskommentiert hinterlegt. Der war von außen nicht erreichbar — ggf. nur
aus dem Uni-Netz.

## Datum

Schritt 5 nutzt `date.today()`. Die Kernaussage (0 anstehende Termine)
bleibt gültig, solange die Seite keine Zukunftsbeiträge bekommt — der
jüngste Eintrag ist vom 03.07.2026.
