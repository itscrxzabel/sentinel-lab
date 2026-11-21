# SOC-Report: Sentinel Detection Lab

**Zeitraum:** 20.11.2026 bis 21.11.2026 (Ortszeit)
**Umgebung:** Windows-Server-Testsystem mit Microsoft Sentinel, Angriffe simuliert mit Atomic Red Team
**Hinweis:** Alle Alerts stammen aus eigenen, bewusst ausgelösten Tests. Es gab keinen echten Angriff.

## Zusammenfassung

Vier Angriffstechniken wurden simuliert. Alle vier wurden von einer Analytics Rule in Sentinel erkannt (Erkennungsrate 4 von 4). Dabei entstanden 8 Incidents. Sie gehören zu 5 Aktivitäten (der Run-Key-Test lief zweimal), die übrigen 3 Incidents sind Duplikate.

## Zahlen

| Kennzahl | Wert |
| --- | --- |
| Simulierte Techniken | 4 |
| Analytics Rules | 4 |
| Erkannte Techniken | 4 von 4 |
| Incidents gesamt | 8 |
| Davon Duplikate (gleiche Aktivität, mehrfach gemeldet) | 3 |
| Zeit von der Aktivität bis zum Incident | ca. 6 bis 14 Minuten |
| Automation Rules | 1 (Tag `lab-test` und Zuweisung) |

Die Zahlen gelten für die Detection-Tests am 20. und 21.11.2026. Beim späteren Test der Automation Rule entstanden drei weitere Incidents (Nr. 9 bis 11), die nicht mitgezählt sind.

## Einstufung der triagierten Incidents

| Ticket | Incident | Technik | Einstufung | Eskalation |
| --- | --- | --- | --- | --- |
| TICKET-001 | 6 | T1136.001 Konto angelegt | True Positive | Ja, Tier 2 |
| TICKET-002 | 5 | T1087.001 Konten-Abfrage | Benign True Positive | Nein |
| TICKET-003 | 8 | T1053.005 Geplanter Task | True Positive | Ja, Tier 2 |

Der Run-Key-Incident (T1547.001) wurde als erste Rule bewiesen, aber nicht als Ticket ausgearbeitet.

## Top-Techniken

1. **T1547.001, Run-Key-Persistenz:** 4 Incidents (2 Testläufe, jeder wurde doppelt gemeldet).
2. **T1136.001, neues lokales Konto:** 2 Incidents (1 Testlauf, doppelt gemeldet).
3. **T1053.005 und T1087.001:** je 1 Incident.

Drei der vier Techniken gehören zur **Persistence**. Das bedeutet, dass der Angreifer sich dauerhaft Zugang sichern will.

## Empfehlungen

1. **Duplikate verringern:** Die Rules laufen alle 5 Minuten und schauen 10 Minuten zurück. Dadurch entstehen doppelte Incidents. Alert Grouping nach Konto und Host würde sie zusammenfassen.
2. **Bekannte Admin-Aktivität ausnehmen:** Konten und Skripte, die regelmäßig Konten abfragen oder Tasks anlegen, sollten in einer Ausschlussliste stehen. So gibt es weniger Fehlalarme.
3. **Lücken schließen:** Die Rules erkennen nur bestimmte Wege, z. B. `net user`, `reg add` und Tasks mit Shell-Aktion. Dieselben Aktionen über PowerShell (z. B. `New-LocalUser`) werden nicht erkannt.
4. **Eskalationsregel festlegen:** Wenn mehrere Persistence-Alerts am selben Host in kurzer Zeit auftreten, immer an Tier 2 geben.
