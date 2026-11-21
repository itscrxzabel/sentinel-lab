# Event-Referenz

Welche Windows-Events und Sysmon-Events das Lab sammelt, was sie bedeuten und welche Felder dabei wichtig sind. Security Events landen in Sentinel in der Tabelle `SecurityEvent`.

## Windows Security Events

| Event | Bedeutung | Wichtige Felder | Im Lab genutzt |
| --- | --- | --- | --- |
| **4624** | Erfolgreiche Anmeldung | `TargetUserName`, `LogonType`, `IpAddress` | Gesammelt, in keiner Rule genutzt |
| **4625** | Fehlgeschlagene Anmeldung | `TargetUserName`, `LogonType`, `IpAddress` | Gesammelt, in keiner Rule genutzt |
| **4688** | Neuer Prozess gestartet | `NewProcessName`, `ParentProcessName`, `CommandLine`, `SubjectUserName` | T1136.001, T1547.001, T1087.001 |
| **4698** | Geplanter Task erstellt | `TaskName`, `TaskContent` (XML), `SubjectUserName` | T1053.005 |
| **4720** | Benutzerkonto erstellt | `TargetUserName` (neues Konto), `SubjectUserName` (wer es angelegt hat) | Gesammelt, in keiner Rule genutzt |

### Was man dazu wissen muss

- **4688 und die Kommandozeile:** Ohne die Richtlinie „Befehlszeile in Prozesserstellungsereignisse einschließen" bleibt `CommandLine` leer. Außerdem muss die Überwachung „Prozesserstellung" aktiv sein. Beides habe ich auf der VM eingeschaltet.
- **4698 braucht eine Audit-Einstellung:** Die Unterkategorie „Other Object Access Events" (auf der deutschen VM „Andere Objektzugriffsereignisse") ist standardmäßig aus. Ohne sie wird der Task nicht geloggt.
- **4698 und die Data Collection Rule:** Das Event kam erst in Sentinel an, nachdem ich den XPath-Filter `Security!*[System[(EventID=4698)]]` ergänzt hatte.
- **4698: Task-Name und Inhalt liegen im Roh-XML** (`EventData`). Die Spalten `TaskName` und `TaskContent` gibt es nicht fertig, die Query liest sie mit einem regulären Ausdruck aus.
- **`net.exe` und `net1.exe`:** Ein einziger `net user`-Befehl erzeugt zwei 4688-Events, weil `net.exe` den Prozess `net1.exe` startet. Das ist normal und kein doppelter Angriff.
- **`LogonType` bei 4624/4625:** 2 = lokal am Gerät, 3 = über das Netzwerk, 10 = Remote Desktop.

## Sysmon-Events

Sysmon liefert mehr Details als die Windows-Bordmittel. Das Lab sammelt folgende Events über eine eigene Data Collection Rule.

| Event | Bedeutung | Wichtige Felder | Typischer Nutzen |
| --- | --- | --- | --- |
| **1** | Prozess erstellt | `Image`, `CommandLine`, `ParentImage`, `User`, `Hashes` | Wie 4688, dazu Hashes und Parent-Kommandozeile |
| **3** | Netzwerkverbindung | `Image`, `DestinationIp`, `DestinationPort`, `Protocol` | Auffällige ausgehende Verbindungen |
| **10** | Zugriff auf einen anderen Prozess | `SourceImage`, `TargetImage`, `GrantedAccess` | Zugriff auf `lsass.exe` (Credential Dumping) |
| **11** | Datei erstellt | `TargetFilename`, `Image` | Neue Dateien in Temp- oder Autostart-Ordnern |
| **13** | Registry-Wert gesetzt | `TargetObject`, `Details`, `Image` | Run-Key-Persistenz |

### Was man dazu wissen muss

- **Event 10 erzeugt viele Einträge**, selbst wenn nur `lsass.exe` überwacht wird. Ich habe die Sysmon-Konfiguration schrittweise eingegrenzt (z. B. `taskmgr.exe` ausgeschlossen).
- **Im Lab sind alle Sysmon-Events gesammelt, aber keine der vier Rules nutzt sie.** Die Erkennung basiert auf Security Events. Sysmon Event 13 wäre die genauere Quelle für T1547.001, weil es den Registry-Eintrag direkt zeigt und nicht nur den Befehl `reg add`.
