# TICKET-001: Neues lokales Konto angelegt

- **Incident:** 6 (Incident 7 ist ein Duplikat)
- **Rule:** Local account created via net.exe
- **Severity:** Medium
- **MITRE:** T1136.001 (Persistence)
- **Zeit:** 21.11.2026, 11:24 Uhr (Ortszeit)

## Alert

Jemand hat mit `net user ... /add` ein neues Benutzerkonto auf dem Server angelegt. Das Konto, das den Befehl ausgeführt hat, ist `Administrator`.

## Triage (was ich geprüft habe)

- **Befehl:** `net user /add "T1136.001_CMD" ...`. Das neue Konto heißt `T1136.001_CMD`.
- **Passwort:** Es steht im Klartext im Befehl. Das ist unsicher.
- **Name:** Der Kontoname passt zu keinem Mitarbeiter und zu keinem Dienstkonto.
- **Quelle:** Im Lab habe ich den Befehl selbst mit Atomic Red Team ausgeführt.

## Einstufung

**True Positive.** Der Alert ist richtig und die Aktivität ist verdächtig.

## Begründung

Ein neues Konto mit seltsamem Namen, das ein Administrator per Kommandozeile anlegt, ist ein typischer Weg, sich Zugang dauerhaft zu sichern. Es gibt keinen Grund wie eine Neueinstellung oder ein Admin-Skript.

## Eskalation

**Ja, an Tier 2.** Wer ein Konto anlegen kann, hat schon Admin-Rechte. Wie der Angreifer das geschafft hat, muss Tier 2 klären.

## Empfehlung

- Neues Konto sperren oder löschen.
- Prüfen, ob sich das Konto schon angemeldet hat.
- Passwort des Kontos `Administrator` ändern.
- Bekannte Admin-Skripte später von der Rule ausnehmen, damit es weniger Fehlalarme gibt.
