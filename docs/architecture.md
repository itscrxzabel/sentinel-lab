# Architektur

Das Lab besteht aus einer lokalen Windows-Server-VM und Microsoft Sentinel in Azure. Die VM sendet Events nach Azure, Sentinel wertet sie aus und erzeugt Incidents.

## Datenfluss

```
 Host-PC (lokal)                          Azure
+---------------------------+            +-----------------------------------------------+
| Windows Server 2022 (VM)  |            |                                               |
|                           |  HTTPS     |  Log Analytics Workspace                      |
|  Atomic Red Team          | ---------> |    SecurityEvent, Sysmon-Events               |
|  (simuliert Angriffe)     |  nur       |        |                                      |
|                           |  ausgehend |        v                                      |
|  Security-Log + Sysmon    |            |  Microsoft Sentinel                           |
|        |                  |            |    4 Analytics Rules (alle 5 Minuten)         |
|  Azure Monitor Agent      |            |        |                                      |
|  (über Azure Arc)         |            |        v                                      |
+---------------------------+            |    Incidents -> Automation Rule (Tag, Owner)  |
                                         +-----------------------------------------------+
```

## Komponenten

| Komponente | Aufgabe | Wichtige Einstellung |
| --- | --- | --- |
| Windows-Server-2022-VM (VirtualBox) | Zielsystem für die Angriffe | 4 GB RAM, 2 CPUs, NAT-Netzwerk |
| Atomic Red Team | Führt Angriffe nach MITRE ATT&CK aus | Ordner in der Defender-Ausschlussliste |
| Sysmon | Zusätzliche Prozess-, Netzwerk- und Registry-Events | Konfiguration eingegrenzt (Event 10) |
| Azure Arc | Verbindet die lokale VM mit Azure | Paket „Essential Machine Management" nicht aktiviert (kostenpflichtig) |
| Azure Monitor Agent | Schickt Events an den Workspace | Läuft auf der VM, gesteuert über Data Collection Rules |
| Data Collection Rules (DCR) | Legen fest, welche Events gesammelt werden | Je eine DCR für Security Events und Sysmon |
| Log Analytics Workspace | Speichert die Logs | Daily Cap 1 GB pro Tag |
| Microsoft Sentinel | SIEM: Rules, Incidents, Automation | Vier Scheduled Analytics Rules, eine Automation Rule |

## Welche Events fließen

| Quelle | Gesammelte Events | Filter |
| --- | --- | --- |
| Windows Security-Log | 4624, 4625, 4688, 4698, 4720 | XPath je Event, z. B. `Security!*[System[(EventID=4698)]]` |
| Sysmon | 1, 3, 10, 11, 13 | Event-IDs in der DCR gefiltert |

Beschreibung der Events: [event-reference.md](event-reference.md)

## Voraussetzungen auf der VM

Damit die Events überhaupt entstehen, musste ich auf der VM zwei Dinge einschalten:

1. **Prozesserstellung überwachen** und die Richtlinie **„Befehlszeile in Prozesserstellungsereignisse einschließen"**. Sonst bleibt `CommandLine` bei 4688 leer.
2. **Audit-Unterkategorie „Other Object Access Events"** (deutsch: „Andere Objektzugriffsereignisse"). Sonst wird 4698 nicht geloggt.

Danach musste 4698 zusätzlich in der DCR ergänzt werden, sonst kam es nicht in Sentinel an.

## Vom Event zum Incident

1. Atomic Red Team führt einen Test aus (z. B. `net user /add`).
2. Windows schreibt ein Event ins Security-Log (4688).
3. Der Azure Monitor Agent schickt es per DCR an den Workspace (Verzögerung etwa 2 bis 5 Minuten).
4. Eine Analytics Rule läuft alle 5 Minuten, schaut 10 Minuten zurück und findet das Event.
5. Sentinel erzeugt einen Alert und daraus einen Incident.
6. Die Automation Rule setzt den Tag `lab-test` und weist den Incident zu.
7. Ein Tier-1-Analyst bearbeitet den Incident (siehe `tickets/`).

## Sicherheit und Kosten

- **Keine eingehenden Verbindungen:** Die VM nutzt NAT ohne Port Forwarding und baut nur ausgehende HTTPS-Verbindungen auf. Sie ist von außen nicht erreichbar.
- **Kein Secret in der VM:** Das Onboarding mit Azure Arc lief über ein kurzlebiges Token vom Host. Es liegt dort nicht dauerhaft.
- **Kostenbremse:** Daily Cap 1 GB im Workspace und eine Alert Rule, die bei Erreichen des Limits warnt.
- **Aufräumen nach dem Projekt:** Workspace, Alert Rule, Action Group und Arc-Ressource werden gelöscht.
