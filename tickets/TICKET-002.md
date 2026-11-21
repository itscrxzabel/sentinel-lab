# TICKET-002: Viele Befehle zur Konten-Abfrage

- **Incident:** 5
- **Rule:** Burst of local account discovery commands
- **Severity:** Low
- **MITRE:** T1087.001 (Discovery)
- **Zeit:** 21.11.2026, 11:00 Uhr (Ortszeit)

## Alert

Innerhalb von 5 Minuten wurden mindestens 3 Befehle ausgeführt, die Konten und Gruppen auflisten (`net user`, `net localgroup`, `cmdkey /list`). Es waren 7 Befehle vom Konto `Administrator`.

## Triage (was ich geprüft habe)

- **Befehle:** Alle lesen nur. Es wurde nichts angelegt oder gelöscht.
- **Konto:** `Administrator` ist das normale Admin-Konto des Servers.
- **Umfeld:** Kein neues Konto, keine fehlgeschlagenen Anmeldungen in diesem Zeitraum.
- **Quelle:** Die Befehle gehören zu meinem Atomic-Red-Team-Test.

## Einstufung

**Benign True Positive.** Die Rule hat richtig erkannt, aber die Aktivität ist harmlos, weil ich sie selbst als Test ausgelöst habe.

## Begründung

Viele Abfragen in kurzer Zeit können eine Aufklärung durch einen Angreifer sein. Hier kommt sie aber von einem bekannten Admin und ist nur lesend. Deshalb ist die Severity auch nur Low.

## Eskalation

**Nein.** Incident schließen als „Benign Positive".

## Empfehlung

- Bekannte Admin-Konten oder Skripte von der Rule ausnehmen.
- Wäre die Aktivität von einem unbekannten Konto gekommen oder kurz vor einer anderen Warnung, müsste ich eskalieren.
