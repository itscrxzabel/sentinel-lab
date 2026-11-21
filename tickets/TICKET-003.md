# TICKET-003: Geplanter Task startet eine Shell

- **Incident:** 8
- **Rule:** Scheduled task created with a script/shell action
- **Severity:** Medium
- **MITRE:** T1053.005 (Execution, Persistence, Privilege Escalation)
- **Zeit:** 21.11.2026, 11:41 Uhr (Ortszeit)

## Alert

Es wurde ein neuer geplanter Task angelegt (Event 4698), der `cmd.exe` startet.

## Triage (was ich geprüft habe)

- **Name:** Der Task heißt `\spawn`. Das ist ein allgemeiner Name und passt zu keiner Software.
- **Aktion:** Er startet `cmd.exe`, also eine Kommandozeile.
- **Ersteller:** Das Konto `Administrator`.
- **Umfeld:** Im selben Zeitraum wurden auch ein neues Konto und ein Run-Key angelegt (siehe TICKET-001).
- **Quelle:** Im Lab habe ich den Task selbst mit Atomic Red Team erstellt.

## Einstufung

**True Positive.** Der Alert ist richtig und die Aktivität ist verdächtig.

## Begründung

Ein Task mit unbekanntem Namen, der eine Kommandozeile startet, kann ein Angreifer nutzen, um später wieder Code auszuführen. Dazu kommen weitere verdächtige Dinge im selben Zeitraum.

## Eskalation

**Ja, an Tier 2.** Tier 1 kann den Task auf dem Server nicht untersuchen oder löschen.

## Empfehlung

- Task prüfen und löschen.
- Prüfen, ob der Task schon gelaufen ist.
- Server auf weitere Spuren prüfen (neue Konten, Run-Keys, andere Tasks).
- Bekannte Update-Tasks später von der Rule ausnehmen.
