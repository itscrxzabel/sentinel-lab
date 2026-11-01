# Azure-Grundlagen
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

# Windows Server 2022 VM
- Windows Server Evaluation VM (2022) VirtualBox aufgebaut | 4 GB RAM | 2 CPUs |
50 GB Festplatte | Ablauf: 180 Tage (26. April 2027) innerhalb Projektscope
- ISO aus dem Microsoft Evaluation Center heruntergeladen | in
VirtualBox eingebunden.
- NAT wurde gewählt weil die VM nur ausgehend ins Internet soll.
- Sicherungspunkt erstellt


# Azure Arc
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
ausgeführt. Die übrigen Parameter (Tenant, Subscription, Resource Group,
Region) stammen aus dem im Portal erzeugten Skript.
    - `az login` auf dem Host, Browser öffnet sich, Anmeldung mit
    Microsoft-Konto.
    - Token vom Host erzeugt, in der VM mit `--access-token` verwendet, danach
    gelöscht.
- `VM-SENTINEL-01` steht im Portal unter Azure Arc auf `Connected`.
- Aufgeräumt: Token-Variable und PowerShell-Historie in der VM gelöscht,
Clipboard deaktiviert, `az logout` auf dem Host.
- Snapshot `02-arc-connected` erstellt (VM vorher heruntergefahren).
