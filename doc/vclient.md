# **vclient -** Client Programm zu Kommunikation mit dem [vcontrold](README-en.md). 

Das Programm wird durch den Aufruf
**make vclient**
übersetzt.


**Aufruf:**
Mit der Integration des P300 Protokolls wurde auch der Aufruf des vclient überarbeitet. Zum einen spielt die Reihenfolge der Parameter keine Rolle mehr. Zum anderen sind jetzt auch lange Versionen der einzelnen Parameter möglich. Neu hinzugekommen sind die Parameter für Cacti und Munin Formatierung der Ausgabe. Weitere Parameter am Ende werden automatisch als Befehle interpretiert.

```

Alte Version:
usage: vclient -h <ip:port> [-c <command1,command2,..>] [-f <commandfile>] [-s <csv-Datei>]
 [-t <Template-Datei>] [-o <outpout Datei> [-x <exec-Datei>] [-k] [-m] [-v]
oder mit langen Parametern:
usage: vclient --host <ip> --port <port> [--command <command1,command2,..>] [--commandfile <commandfile>]
 [--cvsfile <csv-Datei>] [--template <Template-Datei>] [--output <outpout Datei>] [--execute <exec-Datei>]
 [--cacti] [--munin] [--verbose] [command3 [command4] ...]

 -h|--host <IPv4>:<Port> oder <IPv6> des vcontrold
 -p|--port <Port> des vcontrold bei IPv6 (und IPv4)
 -c|--command Liste von auszufuehrenden Kommandos, durch Komma getrennt
 -f|--commandfile Optional Datei mit Kommandos, pro Zeile ein Kommando
 -s|--csvfile Ausgabe des Ergebnisses im CSV Format zur Weiterverarbeitung
 -t|--template Template, Variablen werden mit zurueckgelieferten Werten ersetzt.
 -o|--output Output, der stdout Output wird in die angegebene Datei geschrieben
 -x|--execute Das umgewandelte Template (-t) wird in die angegebene Datei geschrieben und anschliessend ausgefuehrt.
 -m|--munin Munin Datalogger kompatibles Format; Einheiten und Details zu Fehler gehen verloren.
 -k|--cacti Cacti Datalogger kompatibles Format; Einheiten und Details zu Fehler gehen verloren.
 -v|--verbose Verbose Modus zum testen

```


## Template Modus 
Im Template Modus (-t) wird ein Template eingelesen und die dort enthaltenen Variablen ersetzt. Die Ausgabe erfolgt dann auf stdout.
Damit ist es einfach möglich, die Messwerte der Heizung auszulesen und direkt in eine Datenbank zu schreiben.

| **Variable** | **Funktion** |
|----|----|
| $1..$n | Rückgabewert gewandelt in Gleitkommazahl |
| $R1..$Rn | Rückgabewert ungewandelt (Text) |
| $C1..$Cn | aufgerufenes Kommando |
| $E1..$En | Fehlermeldung pro Kommando |


### Einfaches Beispiel: 
Datei sql.tmpl
```

cat sql.tmpl
INSERT INTO messwerte values (CURRENT_DATE,$1,$2);

```

Aufruf:
```

./vclient -h 127.0.0.1:1234 -t sql.tmpl -c getTempA,getTempWWist
INSERT INTO messwerte values (CURRENT_DATE,-2.600000,54.299999);


```


Diese Werte können nun direkt über ein DB-cli in die Datenbank geschrieben werden.
Die Ausgabe von stderr sollte dafür umgeleitet werden:
```

./vclient -h 127.0.0.1:1234 -t sql.tmpl -c getTempA,getTempWWist 2>/dev/null  |mysql -D vito

```

Dies lässt sich mitt der Option -x noch eleganter lösen.
Der generierte Output wird in die angegebene Datei geschrieben und diese wird anschließend ausgeführt. Der Return Wert von vclient wird vom Return Wert des generierten Scriptes übernommen.
```

cat sh-example.tmpl
#!/bin/sh
echo "Das ist ein Beispiel eines ausfuehrbaren Scriptes"
echo "Kommando 1: $C1 Kommando 2: $C2"
echo "rrdb Update machen wir mit: update \$db N:$1:$2"

```

Der \ bewirkt, dass die Variable nicht betrachtet wird.
Nun setzen wir auf Shell Ebene die Variable $db und führen vclient mit dem Schalter -x aus:
```

export db='VITODB'

./vclient -h 127.0.0.1:1234 -c getTempA,getTempWWist -t sh-example.tmpl -x sh-example.sh
Das ist ein Beispiel eines ausfuehrbaren Scriptes
Kommando 1: gettempA Kommando 2: gettempWW
rrdb Update machen wir mit: update VITODB N:-2.600000:54.299999

```

Um Kommunikationsfehler mit der Anlage auszuwerten (z.B. ein vclient Prozess loggt zyklisch Messdaten, in dieser Zeit blockiert ein anderer Client die Kommunikation mit der Schnittstelle), sollte die Fehlervariable des ersten lesenden Befehls abgefangen und ausgewertet werden.
Beispiel:
```

#!/bin/sh

if [ "x$E1" != x ]; then
        echo "es ist ein Fehler aufgetreten: $E1"
else
        echo "Ergebnis: $C1 = $R1"
fi


```

Wird nun die Verarbeitung im *else* Zweig eingebaut, so stellt man sicher, dass Fehler nicht zu falsch geloggten Daten führen.

**Ein ausführliches Beispiel zur Visualisierung der Daten mit der RRDB (Round Robin Database) ist [hier](Datenauswertung-mit-RRDB) beschrieben.**
