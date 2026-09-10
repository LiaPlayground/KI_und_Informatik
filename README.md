<!--
author:   Sebastian Zug
email:    sebastian.zug@informatik.tu-freiberg.de
version:  3.0.0
language: de
narrator: Deutsch Female
comment:  Wir lassen eine KI ein Programm schreiben, das die Schul-Webseite
          ausliest - und schauen genau hin, wo das schiefgeht.

import:   https://raw.githubusercontent.com/LiaScript/CodeRunner/master/README.md
-->

# Kann das nicht die KI machen? 

<h3>Oder: Warum sollte ich noch studieren</h3>

**Kreativitätsgymnasium Leipzig | 10. September 2026**

---

> Prof. Dr. Sebastian Zug, Johannes Kohl
>
> **Technische Universität Bergakademie Freiberg**

## Zielstellung

Unsere Aufgabe:

> **"Schreib mir ein Programm, das auf der Schul-Webseite nachschaut, welche Termine anstehen - und mir eine E-Mail schickt."**

![Nachrichtenseite des BIP Kreativitätsgymnasiums Leipzig](bilder/nachrichtenseite.png)<!--
style="max-width: 40%; border: 1px solid #ccc;"
-->

[www.bip-schulen.de/nachrichten-gyl](https://www.bip-schulen.de/nachrichten-gyl)

> [!TIP]
> **Wie gehen wir im Zeitalter von KIs vor? Wir kopieren die Aufgabe in ein LLM und hoffen auf ein schnelles Ergebnis**

## KI Lösung für Noobs

```txt LLM-Anfrage
Schreib mir ein Programm, das auf der Schul-Webseite nachschaut, 
welche Termine anstehen - und mir eine E-Mail schickt. Die URL 
lautet 

https://www.bip-schulen.de/nachrichten-gyl
```


``` python  LLM-script.py
import requests
from bs4 import BeautifulSoup
from datetime import datetime, timedelta
import smtplib
from email.mime.text import MIMEText

URL = "https://www.bip-schulen.de/nachrichten-gyl"

def hole_termine():
  response = requests.get(URL)
  soup = BeautifulSoup(response.text, 'html.parser')
  termine = []
  for item in soup.find_all('div', class_='blog-item'):
    titel = item.find('h2').text.strip()
    datum_str = item.find('time').text.strip()
    datum = datetime.strptime(datum_str, '%d.%m.%Y')
    if datum >= datetime.now() and datum <= datetime.now() + timedelta(days=7):
        termine.append((datum, titel))
  return termine

def sende_mail(termine):
  text = "\n".join(f"{d.strftime('%d.%m.%Y')}: {t}" for d, t in termine)
  msg = MIMEText(text)
  msg['Subject'] = 'Kommende Termine'
  msg['From'] = 'bot@schule.de'
  msg['To'] = 'ich@example.org'
  with smtplib.SMTP('smtp.schule.de', 587) as s:
    s.starttls()
    s.login('bot', 'geheim')
    s.send_message(msg)

if __name__ == '__main__':
  termine = hole_termine()
  if termine:
    sende_mail(termine)
    print(f"{len(termine)} Termine versendet.")
```
@LIA.python

Das sieht überzeugend aus: sinnvoll benannte Funktionen, sauber getrennt in
Holen und Versenden, sogar eine Bedingung, die nur bei Treffern eine Mail
schickt. Wer schon einmal Python gesehen hat, würde das durchwinken.

                                    {{2}}
Also ausprobieren. Und schon die allererste Zeile bringt das Programm zu Fall:

                                    {{2}}
``` text
Traceback (most recent call last):
  File "/tmp/tmp_47w66ol/main.py", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
```

--{{2}}--
Nicht Zeile 20, nicht der komplizierte Teil mit dem Datum - Zeile 1. Das
Programm kommt nicht einmal bis zum ersten Befehl.

<details>
<summary>**Warum passiert das?**</summary>

`requests` ist eine der bekanntesten Python-Bibliotheken für Webzugriffe.
In unzähligen Tutorials, Büchern und Beispielen steht genau diese Zeile -
und deshalb schreibt eine KI sie auch.

Nur: `requests` gehört **nicht** zur Standardausstattung von Python. Es muss
extra installiert werden. Auf dem Server, auf dem dieser Kurs seinen Code
ausführt, ist es nicht vorhanden.

Die KI konnte das nicht wissen. Sie kennt unsere Umgebung nicht - sie hat
nie nachgesehen, was hier installiert ist. Sie hat das Übliche gewählt,
und das Übliche ist hier falsch.

Der Ausweg steht in der Standardbibliothek: `urllib.request` kann dasselbe,
ist etwas umständlicher zu schreiben und **immer** da. Genau das nutzen die
Programme auf den nächsten Seiten.

</details>

                                    {{3}}
> **Frage:** Ist das jetzt ein Fehler der KI?

--{{3}}--
Nein - und das macht es interessant. Auf dem Rechner der meisten Entwickler
wäre `requests` installiert, und der Code liefe. Der Fehler entsteht erst
im Zusammentreffen mit **dieser** Umgebung.

Wer Code übernimmt, übernimmt auch dessen Voraussetzungen. Sichtbar werden
die erst, wenn sie fehlen.


## Der (steinige) Weg 

> [!IMPORTANT] 
> **Was macht ein Profi?**

                {{0-1}}
***********************************************

Ein Profi hätte diesen Code **erst gelesen, dann ausgeführt**. Und beim
Lesen fällt weit mehr auf als die fehlende Bibliothek - sechs Stellen, von
denen keine einzige eine Fehlermeldung erzeugt:

| Stelle im Code | Was daran fragwürdig ist |
| -------------- | ------------------------- |
| `requests.get(URL)` | Kein `timeout`. Antwortet der Server nicht, hängt das Programm auf unbestimmte Zeit. Und wenn die Seite mal nicht erreichbar ist, stürzt es ab. |
| `find('time').text` | Nimmt den **sichtbaren** Text `03.07.2026` - obwohl im selben Tag ein maschinenlesbares `datetime`-Attribut steht. |
| `strptime(..., '%d.%m.%Y')` | Verlässt sich darauf, dass das Datum immer deutsch formatiert ist. |
| `find_all(...)` | Liest genau eine Seite. Ob es weitere gibt, wurde nie geprüft. |
| `timedelta(days=7)` | Woher kommt die 7? Stand nicht in der Aufgabe. Die KI hat geraten. |
| `s.login('bot', 'geheim')` | Passwort im Quelltext. Wandert so in jedes Backup und jedes Repository. |

> Keiner dieser Punkte ist ein Syntaxfehler. Der Code läuft. Genau das macht sie gefährlich: Es gibt nichts, was von allein auffällt.

***********************************************

                {{1-2}}
***********************************************

> [!WARNING]  
>  **Und das Schlimmste steht gar nicht im Code - Die Liste ist unvollständig!** 

Alle sechs Punkte betreffen das *Wie*. Die viel größere Frage ist das *Ob*! **Stehen auf dieser Seite überhaupt Termine?**

***********************************************

                                    {{2}}
Deshalb geht ein Profi anders vor - und zwar **bevor** die KI ins Spiel kommt:

                                    {{2}}
1. **Anforderungen konkretisieren** - was genau ist ein "Termin", welcher
   Zeitraum, wie oft benachrichtigen?
2. **Das Problem verstehen** - wie ist die Seite aufgebaut, wo stehen die
   Daten, gibt es sie überhaupt?
3. **Die Lösung entwerfen** - welche Schritte braucht es, was gehört
   getrennt, was passiert bei Fehlern?
4. **Variabilität berücksichtigen** - was ändert sich absehbar? Layout,
   Datumsformat, Menge der Einträge?
5. **Und dann** die KI anwerfen - jetzt kann man präzise beschreiben, was
   gebraucht wird, und beurteilen, was zurückkommt.

## Frage 1: Was heißt eigentlich "Termin"?

Bevor irgendjemand Code schreibt - auch keine KI - muss klar sein,
was überhaupt gewollt ist.

> **Aufgabe:** Formulieren Sie die Aufgabe so präzise, dass eine fremde
> Person sie ohne Rückfragen umsetzen könnte.
>
> - Was ist ein "Termin"?
> - "Anstehend" - ab wann, bis wann?
> - Was heißt "Bescheid geben"? Wie oft?

### Lösung 1

**Warum ist das so schwer?**

Umgangssprache ist mehrdeutig, ohne dass es auffällt. "Die nächsten
Termine" kann heißen: die nächsten drei, die der nächsten Woche, alle
zukünftigen. Solange nur Menschen miteinander reden, klärt sich das
nebenbei. Ein Programm fragt nicht nach - es macht irgendetwas Bestimmtes.

Diese Präzisierung ist keine Vorarbeit zum Programmieren.
Sie **ist** ein großer Teil des Programmierens.


## Frage 2: Wie können wir den Termin robust auslesen

Um die Seite geht es:
[www.bip-schulen.de/nachrichten-gyl](https://www.bip-schulen.de/nachrichten-gyl)

--{{0}}--
Öffnen Sie die Seite und drücken Sie F12. Damit sehen Sie den Quelltext -
das, was der Browser tatsächlich bekommt und in die hübsche Ansicht
übersetzt.

                                    {{1}}
Ein einzelner Artikel sieht im Quelltext so aus:

``` html
<div class="com-content-category-blog__item blog-item">
  <h2>
    <a href="/nachrichten-gyl/2760-abschluss-meeting">
      Abschluss-Meeting
    </a>
  </h2>
  <dd class="published">
    <time datetime="2026-07-03T00:00:00+02:00">
       03.07.2026
    </time>
  </dd>
</div>
```

--{{1}}--
Das ist echter Quelltext von der Seite, nur gekürzt. Jeder Artikel steckt in
einem div mit der Klasse blog-item, der Titel in einem h2, das Datum in einem
time-Tag.

> **Schauen Sie genau hin:** Das Datum steht in diesem Ausschnitt
> **zweimal**. Finden Sie beide? Welches würden Sie im Programm benutzen?


### Lösung 2

         {{0-1}}
*************************************

Das Attribut des Eintrages ist die die deutlich bessere Wahl:

- Es ist normiert (ISO 8601) - Jahr-Monat-Tag, immer gleich aufgebaut
- Es lässt sich direkt sortieren, sogar als Zeichenkette
- Der sichtbare Text kann sich ändern, wenn jemand die Spracheinstellung
  der Webseite umstellt. Aus `03.07.2026` würde dann `July 3, 2026`,
  und ein Programm, das auf das Format baut, geht kaputt.

Der KI-Entwurf von vorhin hat genau hier die schlechtere Variante gewählt:
`find('time').text` und dann `strptime(..., '%d.%m.%Y')`.

Ehrlicherweise: **Das funktioniert heute.** Der sichtbare Text steht zwar
zwischen Tabs und Zeilenumbrüchen, aber `.strip()` räumt das weg, und
`strptime` kommt mit `03.07.2026` klar. Es ist kein Fehler, der jetzt
knallt - es ist einer, der wartet.

*************************************

         {{1-2}}
*************************************

Das liefert die KI, wenn wir uns auf das Attribut fokussieren (so oder so ähnlich).

``` python
import urllib.request
from bs4 import BeautifulSoup

URL = "https://www.bip-schulen.de/nachrichten-gyl"
seite = urllib.request.urlopen(URL, timeout=30).read().decode("utf-8")

suppe = BeautifulSoup(seite, "html.parser")
artikel = suppe.find_all("div", class_="blog-item")

for a in artikel:
    titel = a.find("h2").get_text(strip=True)
    datum = a.find("time")["datetime"][:10]
    print(datum, "|", titel)

print()
print("Gefunden:", len(artikel), "Einträge")
```
@LIA.python

*************************************

## Frage 3: Alle Nachrichten

> **Frage:** Wie viele Nachrichten hat die Seite eigentlich insgesamt?

--{{0}}--
Scrollen Sie auf der Webseite ganz nach unten. Dort steht eine Zahl, die
unser Programm nie beachtet hat.

### Lösung 3

                                    {{1}}
Zählen wir nach - und holen gleich drei Seiten statt einer:

``` python
import urllib.request, re
from bs4 import BeautifulSoup

BASIS = "https://www.bip-schulen.de/nachrichten-gyl"

def hole(url):
    return urllib.request.urlopen(url, timeout=30).read().decode("utf-8")

erste = hole(BASIS)
seiten = int(re.search(r"Seite \d+ von (\d+)", erste).group(1))
print("Die Seite hat", seiten, "Unterseiten")
print("Geschätzte Einträge:", seiten * 16)
print()

alle = []
for nr in range(3):
    url = BASIS if nr == 0 else BASIS + "?start=" + str(nr * 16)
    treffer = BeautifulSoup(hole(url), "html.parser").find_all("div", class_="blog-item")
    alle.extend(treffer)
    print("Seite", nr + 1, "->", len(treffer), "Einträge")

print()
print("Zusammen:", len(alle), "von", seiten * 16)
print("Ältester davon:", min(a.find("time")["datetime"][:10] for a in alle))
```
@LIA.python

--{{1}}--
37 Unterseiten, also rund 592 Einträge. Unser erstes Programm hat 16 davon
gesehen. Und schon drei Seiten reichen über ein Jahr zurück.

<details>
<summary>**Wie kommt man an die anderen Seiten?**</summary>

Klicken Sie auf der Webseite auf "2" und schauen Sie in die Adresszeile:
`...?start=16`. Bei Seite 3 steht dort `?start=32`.

Der Server erwartet also nicht die Seitennummer, sondern wie viele Einträge
übersprungen werden sollen. Deshalb im Code `nr * 16`.

Solche Muster stehen in keiner Dokumentation - man findet sie, indem man
die Seite benutzt und hinschaut. Genau das kann eine KI nicht: Sie hat die
Seite nie geöffnet.

</details>

                                    {{2}}
> **Frage:** Wessen Fehler ist das - der KI oder unserer?

--{{2}}--
Es ist unserer. Wir haben nie gesagt, dass alle Seiten gemeint sind. Die KI
kannte nur unsere Beschreibung und hat daraus das Übliche erzeugt: den Code,
der zu "lies eine Webseite aus" am besten passt.

Entscheidend ist etwas anderes: Das Programm hat **keinen Fehler gemeldet**.
Es hat genau das getan, was im Code stand. Falsch war die Aufgabe.

## Frage 4: Blick in die Zukunft

Wir wollten "anstehende Termine". Filtern wir also nach der Zukunft.

``` python
import urllib.request
from bs4 import BeautifulSoup
from datetime import date

URL = "https://www.bip-schulen.de/nachrichten-gyl"
seite = urllib.request.urlopen(URL, timeout=30).read().decode("utf-8")
suppe = BeautifulSoup(seite, "html.parser")

heute = date.today()
kommend = []

for a in suppe.find_all("div", class_="blog-item"):
    titel = a.find("h2").get_text(strip=True)
    j, m, t = map(int, a.find("time")["datetime"][:10].split("-"))
    if date(j, m, t) >= heute:
        kommend.append((date(j, m, t), titel))

print("Heute ist der", heute.strftime("%d.%m.%Y"))
print("Anstehende Termine:", len(kommend))
print()

for wann, titel in kommend:
    print(wann, "|", titel)
```
@LIA.python

--{{0}}--
Führen Sie den Code aus. Das Ergebnis ist überraschend: null Termine. Keine
Fehlermeldung, keine Warnung - einfach nichts.

                                    {{1}}
> **Frage:** Null Treffer. Ist das Programm kaputt?
>
> Überlegen Sie erst selbst, bevor Sie weiterlesen.

--{{1}}--
Die naheliegende Vermutung ist ein Bug: falsches Datumsformat, falscher
Vergleich, ein Tippfehler. Prüfen wir es nach - schauen wir uns an, was
überhaupt auf der Seite steht.

### Lösung 4

Schauen wir uns die sechs neuesten Einträge an - ohne jeden Filter:

``` python
import urllib.request
from bs4 import BeautifulSoup

URL = "https://www.bip-schulen.de/nachrichten-gyl"
seite = urllib.request.urlopen(URL, timeout=30).read().decode("utf-8")
suppe = BeautifulSoup(seite, "html.parser")

for a in suppe.find_all("div", class_="blog-item")[:6]:
    titel = a.find("h2").get_text(strip=True)
    datum = a.find("time")["datetime"][:10]
    print(datum, "|", titel)
```
@LIA.python

--{{0}}--
Lesen Sie die Titel: Abschluss-Meeting. Woche der Wissenschaft. Abitur 2026.
Studienfahrt nach München. Sportfest.

Das sind Rückblicke - Berichte über Dinge, die stattgefunden haben, meist
mit Fotos. Und alle liegen in der Vergangenheit.

                                    {{1}}
**Der Code war nie kaputt.**

Auf dieser Seite stehen überhaupt keine Termine.
Es ist eine Nachrichtenseite - ein Archiv, kein Kalender.

Unsere ganze Aufgabenstellung beruhte auf einer falschen Annahme.

--{{1}}--
Das Programm lief fehlerfrei. Die KI hat sauberen Code geliefert. Trotzdem
war das Ergebnis wertlos - weil die Voraussetzung nicht stimmte.

<details>
<summary>**Warum konnte die KI das nicht merken?**</summary>

Weil sie die Seite nie gesehen hat. Sie kannte nur unsere Beschreibung:
"Termine von der Schul-Webseite".

Eine KI erzeugt Code, der zu dieser Beschreibung passt - und der Code war
dafür korrekt. Ob es die beschriebenen Daten überhaupt gibt, steht nicht
im Prompt und lässt sich daraus auch nicht erschließen.

Der Unterschied ist wichtig:

- **Syntaxfehler** meldet Python sofort
- **Logikfehler** fällt oft beim Testen auf
- **Falsche Annahme über die Wirklichkeit** meldet niemand.
  Das Programm läuft, gibt "0" aus und sieht dabei aus wie Erfolg.

</details>

## Frage 5: Und die E-Mail?

Angenommen, wir hätten Termine gefunden. Der Versand sähe so aus:

``` python
import smtplib
from email.message import EmailMessage

nachricht = EmailMessage()
nachricht["From"] = "schulbot@example.org"
nachricht["To"] = "klasse12@example.org"
nachricht["Subject"] = "Kommende Termine"
nachricht.set_content("Hier stünden die Termine.")

with smtplib.SMTP("smtp.example.org", 587) as server:
    server.starttls()
    server.login("benutzer", "passwort")
    server.send_message(nachricht)
```

--{{0}}--
Dieser Block hat bewusst keinen Ausführen-Knopf: Die Zugangsdaten sind
erfunden. Zum Ausprobieren bräuchten Sie ein echtes Postfach - und hätten
dann die Frage, wo das Passwort sicher liegt, statt im Quelltext.

                                    {{1}}
> **Frage:** Was passiert, wenn dieses Programm jeden Morgen um 7 Uhr läuft
> - und die Webseite drei Wochen lang unverändert bleibt?

--{{1}}--
Überlegen Sie einen Moment, bevor Sie weiterblättern.

### Lösung 5

Es kommen **21 identische Mails**. Mit unserem Filter sogar 21 leere.
Ab Tag drei liest die niemand mehr.

                                    {{1}}
**Was fehlt dem Programm?**

Es müsste sich merken, was es beim letzten Mal verschickt hat, und nur
Neues melden. Dafür braucht es einen Zustand, der Programmläufe
überdauert - eine Datei oder Datenbank.

--{{1}}--
Das ist kein exotischer Sonderfall, sondern der Normalfall: Sobald ein
Programm regelmäßig läuft statt einmal, kommen Fragen dazu, die im
Prompt nie auftauchen. Was, wenn die Seite nicht erreichbar ist? Was, wenn
ein Eintrag nachträglich geändert wird?

Diese Fragen fallen in keinem Test auf. Sie fallen im Betrieb auf.

## Was haben wir gesehen?

Zwei Fehler haben sich sofort gemeldet: die fehlende Bibliothek und der
erfundene Mailserver. Beide waren in Minuten behoben - laute Fehler sind
die harmlosen.

                                    {{1}}
Die interessanten Fehler waren die anderen. Denn ansonsten hat die KI alles
richtig gemacht:

- Der Code war syntaktisch korrekt
- Er lief am Ende ohne Fehlermeldung durch
- Er war sauber geschrieben

                                    {{2}}
Trotzdem war das Ergebnis unbrauchbar. Warum?

| Was schiefging | Wer hätte es merken können? |
| -------------- | ----------------------------- |
| Nur 16 von 592 Einträgen | Nur wer die Seite anschaut |
| Termine gesucht, wo keine sind | Nur wer den Inhalt liest |
| 21 gleiche Mails | Nur wer den Betrieb mitdenkt |

--{{2}}--
In allen drei Zeilen steht dasselbe: Es braucht jemanden, der das Ergebnis
beurteilen kann. Eine KI kann Code erzeugen. Ob er das richtige Problem
löst, kann sie nicht wissen.

                                    {{3}}
> **Die Frage, um die es hier ging:**
>
> Woher wusste ich, dass die Ausgabe falsch war?

--{{3}}--
Nicht "kann die KI programmieren" - sie kann es, oft besser als ein
Anfänger. Sondern: Wer merkt, dass "0 Termine" nicht "Fehler" bedeutet,
sondern "falsche Annahme"?

Ein Programm, das statt 592 nur 16 Einträge findet, wirft keine
Fehlermeldung. Es gibt keinen roten Text. Es sieht aus wie Erfolg.

<details>
<summary>**Was heißt das für ein Informatikstudium?**</summary>

Ein verbreitetes Missverständnis: Im Studium lerne man Programmiersprachen.
Syntax ist aber der kleinste Teil - und der, den eine KI zuverlässig
übernimmt.

Der größere Teil ist das, was in dieser Stunde dreimal gefehlt hat:

- Eine unklare Aufgabe präzise machen, bevor Code entsteht
- Annahmen über die Wirklichkeit prüfen, statt sie zu glauben
- Beurteilen, ob eine Ausgabe plausibel ist
- Mitdenken, was im Dauerbetrieb passiert

Das lässt sich nicht wegautomatisieren, weil es kein Übersetzungsproblem
ist. Es ist ein Urteilsproblem: Man muss wissen, wie die Sache aussieht,
wenn sie stimmt.

</details>

## Ausblick

Falls Sie selbst weitermachen wollen - die Code-Blöcke lassen sich direkt
im Kurs ändern und neu ausführen:

1. Lassen Sie sich alle 37 Seiten geben statt drei. Wie lange dauert das?
   Und wäre es höflich, das oft zu tun?
2. Sortieren Sie die Einträge nach Monat und zählen Sie, wie viele es je
   Monat gibt.
3. Suchen Sie alle Einträge, in deren Titel "Studienfahrt" vorkommt.
4. Überlegen Sie, wo auf der Schul-Webseite echte Termine stehen könnten -
   und ob die genauso aufgebaut sind wie diese Nachrichten.

## Material und Kontaktdaten

**Dieser Kurs zum Mitnehmen:**

![QR-Code zum Kurs](bilder/qrcode-kurs.png)<!--
style="max-width: 220px;"
-->

[liascript.github.io - Kurs öffnen](https://liascript.github.io/course/?https://raw.githubusercontent.com/LiaPlayground/KI_und_Informatik/main/README.md)

Der Quelltext liegt offen auf GitHub:
[github.com/LiaPlayground/KI_und_Informatik](https://github.com/LiaPlayground/KI_und_Informatik)

Alle Code-Blöcke lassen sich im Kurs direkt ändern und neu ausführen -
probieren Sie die Aufgaben aus dem Ausblick gern selbst aus.

---

**Kontakt**

Prof. Dr. Sebastian Zug
[sebastian.zug@informatik.tu-freiberg.de](mailto:sebastian.zug@informatik.tu-freiberg.de)

Johannes Kohl
[johannes.kohl@informatik.tu-freiberg.de](mailto:johannes.kohl@informatik.tu-freiberg.de)

Institut für Informatik
**Technische Universität Bergakademie Freiberg**

---

Fragen zum Informatikstudium in Freiberg beantworten wir gern -
auch nach dieser Stunde.