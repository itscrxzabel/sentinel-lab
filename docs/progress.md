# Progress

## 2026-10-29 - Woche 1, Tag 3 (Azure Arc Onboarding)

### Erledigt
- Resource Provider in der Subscription registriert (Portal):
`Microsoft.HybridCompute`, `Microsoft.GuestConfiguration`,
`Microsoft.HybridConnectivity`, `Microsoft.AzureArcData`.
`Microsoft.HybridCompute` war vorher nicht registriert.
- Onboarding-Skript im Portal erzeugt: Public endpoint, SQL-Server-Option aus,
kein Proxy, kein Arc Gateway, Automatic agent upgrade an. Essential Machine
Management nicht aktiviert.
- Skript per Shared Clipboard (nur Host zu Gast) in die VM gebracht. Arc-Agent
installiert ("Installation of azcmagent completed successfully").
- Gerätecode-Login scheiterte (Error 530035). Ursache laut Entra Sign-in-Log:
"Access has been blocked by security defaults".
- Azure CLI auf dem Host installiert, Access Token auf dem Host erzeugt,
`azcmagent connect` in der VM mit `--access-token` statt `--use-device-code`
ausgeführt.
- `VM-SENTINEL-01` steht im Portal unter Azure Arc auf `Connected`.
- Aufgeräumt: Token-Variable und PowerShell-Historie in der VM gelöscht,
Clipboard deaktiviert, `az logout` auf dem Host.
- Snapshot `02-arc-connected` erstellt (VM vorher heruntergefahren).

### Entscheidungen
- Public endpoint statt Private Endpoint: Die VM baut nur ausgehende
HTTPS-Verbindungen auf, Private Endpoint wäre kostenpflichtig und unnötig.
- Security Defaults bleiben aktiv: Kein Abschalten zugunsten des
Gerätecode-Logins, weil das Konto mit hinterlegter Kreditkarte geschützt bleiben
soll.
- Token vom Host statt Browser-Login oder Service Principal Secret in der VM:
Die VM wird später absichtlich angegriffen (u. a. Credential-Dumping), also soll
kein dauerhaftes Passwort oder Secret dort liegen. Das Token ist kurzlebig.
- Clipboard nur Host zu Gast und nach dem Kopieren wieder deaktiviert.
- Snapshot bei ausgeschalteter VM, damit kein Token im gespeicherten
Arbeitsspeicher steckt.

### Offen
- DCRs für Security Events und Sysmon
(`Microsoft-Windows-Sysmon/Operational!*`), Sysmon auf der VM installieren.
- Sentinel erst aktivieren, wenn die Datenpfade stehen.
- Datenprüfung: 4624, 4625, 4688, 4720; Sysmon 1, 3, 10, 11, 13.
- Prüfen, welche Datentypen das Daily Cap nicht begrenzt.
- Ursache der langen Installationsdauer des Agent-Pakets ungeklärt (msiexec lief
nicht, Windows Update ungeprüft).
- Aufräumliste für den 22.11. ergänzen: Workspace, Alert Rule, Action Group,
Arc-Ressource löschen, Azure CLI vom Host deinstallieren, erhöhten Zugriff (User
Access Administrator auf Root-Scope) prüfen und zurücknehmen.

### Naechster Schritt
- Sysmon in der VM installieren und DCR für Security Events anlegen.

### Gelernt
- Welche Resource Provider Arc braucht und wo man den Registrierungsstatus pro
Subscription sieht.
- Public endpoint, Proxy und Arc Gateway: wann man sie braucht und warum nicht
in diesem Lab.
- Security Defaults blockieren den Gerätecode-Login. Das Sign-in-Log (Entra,
Monitoring) zeigt die Ursache.
- Snapshots in VirtualBox bilden einen Baum, `02` hängt unter `01`. Zurück auf
`01`: Arc-Ressource in Azure bleibt verwaist. Zurück auf `02`: Änderungen danach
(Sysmon, Atomic-Tests) gehen verloren.
- Ein Token ist wie ein Passwort: kurzlebig, nicht in Chat, Repo oder Historie.

## 2026-10-28 - Woche 1, Tag 2 (VM aufgebaut)

### Erledigt
- Windows Server Evaluation VM (2022) VirtualBox aufgebaut | 4 GB RAM | 2 CPUs |
50 GB Festplatte | Ablauf: 180 Tage (26. April 2027) innerhalb Projektscope
- Sicherungspunkt erstellt

### Entscheidungen
- NAT gewählt, weil alle Verbindungen im Lab von der VM ausgehen (Arc-Agent und
Azure Monitor Agent nach Azure, Updates, Atomic-Red-Team-Download).
- Eingehende Verbindungen zur VM bräuchten Port Forwarding in VirtualBox. Das
richte ich nicht ein: Die VM wird später absichtlich angegriffen und soll aus
dem Heimnetz und dem Internet nicht erreichbar sein.
- Hostname: VM-SENTINEL-01
- Sicherungspunkt `01-baseline-clean`: sauberer Ausgangszustand, auf den ich bei
Fehlern (Konfiguration, Angriffssimulation) zurückgehe.

### Offen
- Azure Arc Onboarding (Essential Machine Management nicht aktivieren).
- DCRs fuer Security Events und Sysmon.
- Sentinel erst nach VM und Arc aktivieren.
- Datenpruefung: 4624, 4625, 4688, 4720; Sysmon 1, 3, 10, 11, 13.
- Pruefen, welche Datentypen das Daily Cap nicht begrenzt.
- Alert Rule auf die Aufraeumliste fuer den 22.11. setzen (Workspace, Alert
Rule, Action Group, Arc-Ressource loeschen).

### Naechster Schritt
- Azure Arc Onboarding (Essential Machine Management nicht aktivieren).

### Gelernt
- Wie man in VirtualBox eine VM aufbaut und konfiguriert.
- Dass man bei Windows Server Evaluation "Desktop Experience" statt Server Core
wählen muss, wenn man eine GUI will. Server Core zeigt nur die Konsole (sconfig)
und lässt sich unter 2022 nicht nachträglich umstellen.
- Dass die unbeaufsichtigte Installation in VirtualBox die Meldung "Microsoft
Software License Terms nicht gefunden" auslöste. Lösung: "Skip Unattended
Installation" und die Installation manuell durchführen.
- Die unterschiedlichen Netzwerkmodi (NAT, Bridged, Host-only) und deren Vor-
und Nachteile.
- NAT: Die VM baut Verbindungen nach außen auf, die NAT-Tabelle ordnet die
Antworten der VM zu. Von außen kann niemand eine Verbindung zur VM starten,
solange kein Port Forwarding existiert. Für Arc genügt das, weil der Agent die
Verbindung (HTTPS) nach Azure aufbaut, nicht umgekehrt.

## 2026-10-27 - Woche 1, Tag 1 (Plumbing)

### Erledigt
- Azure Free Account angelegt, nicht auf Pay-as-you-go hochgestuft.
- Resource Group `rg-sentinel-lab-gwc-001` angelegt (Region Germany West
Central).
- Log-Analytics-Workspace `log-sentinel-lab-gwc-001` angelegt, Pricing Tier
Pay-as-you-go.
- Daily Cap auf 1 GB pro Tag gesetzt.
- Action Group `ag-sentinel-lab-001` mit E-Mail-Benachrichtigung angelegt.
- Alert Rule "Daily data limit reached" (Severity 2, Warning) angelegt: Custom
Log Search auf `_LogOperation`, Table rows, Count, Schwelle 0, Auswertung alle 5
Minuten. Preis laut Anzeige ca. 1,50 $.

### Entscheidungen
- Namensmuster: `<Typkuerzel>-sentinel-lab-<Region>-<Nummer>`.
- Daily Cap 1 GB: Schaetzung fuer eine Windows-Server-VM mit Security Events und
Sysmon. Nach einigen Tagen mit echten Nutzungsdaten pruefen und anpassen.
- Severity Warning, weil die Doku es so vorgibt.

### Offen
- Windows Server Evaluation VM einrichten.
- Azure Arc Onboarding (Essential Machine Management nicht aktivieren).
- DCRs fuer Security Events und Sysmon.
- Sentinel erst nach VM und Arc aktivieren.
- Datenpruefung: 4624, 4625, 4688, 4720; Sysmon 1, 3, 10, 11, 13.
- Pruefen, welche Datentypen das Daily Cap nicht begrenzt.
- Alert Rule auf die Aufraeumliste fuer den 22.11. setzen (Workspace, Alert
Rule, Action Group, Arc-Ressource loeschen).

### Naechster Schritt
- Lokale Windows Server VM vorbereiten (4-6 GB RAM).

### Gelernt (selbst ergaenzen)
- Wie man eine Resource Group und Log-Analytics-Workspace in Azure erstellt.
- Wie man eine Daily Cap fuer einen Log-Analytics-Workspace setzt.
- Wie man ein Alert Rule mit einer Action Group erstellt.
    - Nachricht über Mail
