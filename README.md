# ioc-feeds

Kuratierte Netzwerk-Indikatoren (Domains und IPs) für den Einsatz als dynamische
Threat-Feeds. Eine Zeile pro Indikator, reiner Text, kommentarfrei — direkt von
einem Feed-Consumer per Raw-URL abrufbar.

## Herkunft und Lizenz

Die Indikatoren stammen aus dem öffentlichen
[Zero-Breach-Toolkit](https://github.com/interplains/Zero-Breach-Toolkit) und
sind dort als **CC0 1.0 (Public Domain)** gewidmet — frei nutzbar, auch
kommerziell, ohne Attributionspflicht.

Veröffentlicht wurden die zugrunde liegenden Indikatoren von: The Citizen Lab,
Confiant, Recorded Future / Insikt Group, Microsoft, ESET, Avast / Gen, Google
Threat Analysis Group und dem Amnesty International Security Lab. Thematisch geht
es um die AdFection-Kampagne (Candiru / DevilsTongue) sowie weitere
Mercenary-Spyware-Infrastruktur.

**Stand der Daten:** 2026-10-18 (AdFection), ergänzt 2026-10-25 (weitere
kuratierte Mercenary-Indikatoren). Siehe "Aktualisierung".

## Inhalt

| Datei | Einträge | Inhalt | Empfohlene Aktion |
|-------|---------:|--------|-------------------|
| `domains/block.txt` | 43 | Angreifer-registrierte Domains, starke Tiers (C&C, Validator, Delivery), Confidence `primary`. Shared-Hosting ausgeschlossen. | Nach Burn-in **Block** |
| `domains/cloud-monitor.txt` | 14 | Vollständige Cloud-FQDNs (CloudFront, Azure, Heroku, ELB). Präzise nur als voller Hostname. | **Monitor** |
| `domains/extended.txt` | 161 | `block` plus ältere Candiru-Kampagnen-Domains (`primary`, aber evtl. abgelaufen/neu registriert). | **Monitor** |
| `ips/monitor.txt` | 119 | IPs (C&C und candiru-ip). Viele cloud-/shared-gehostet. | **Monitor**, nie inline Block |

## Wie man es anbindet

Ein Feed-Consumer, der eine HTTP(S)-URL mit "ein Indikator pro Zeile" abruft,
zeigt auf die **Raw-URL** der jeweiligen Datei (nicht die HTML-Ansicht):

```
https://raw.githubusercontent.com/<org-oder-user>/ioc-feeds/main/domains/block.txt
https://raw.githubusercontent.com/<org-oder-user>/ioc-feeds/main/domains/cloud-monitor.txt
https://raw.githubusercontent.com/<org-oder-user>/ioc-feeds/main/domains/extended.txt
https://raw.githubusercontent.com/<org-oder-user>/ioc-feeds/main/ips/monitor.txt
```

Domain- und IP-Feeds sind getrennt, weil der Indikatortyp beim Consumer in der
Regel separat angelegt wird.

Die Raw-Auslieferung hat einen CDN-Cache von einigen Minuten; Änderungen sind
also nicht sofort live. Die URLs bleiben stabil, solange Branch (`main`), Pfade
und Dateinamen unverändert bleiben — nicht umbenennen, keine History umschreiben.

## Einsatz mit Bedacht

- **Erst Monitor, dann Block.** Jede Liste zunächst beobachtend einbinden, die
  Treffer sichten, und nur `domains/block.txt` nach sauberer Burn-in-Phase auf
  Block umstellen. Die übrigen Listen dauerhaft auf Monitor.
- **Cloud-FQDNs nie verkürzen.** Einträge wie `d147…​.cloudfront.net` oder
  `…​.cloudapp.azure.com` sind nur als *vollständiger* Hostname sicher. Niemals
  auf die Parent-Domain reduzieren — das würde legitime Dienste treffen. Cloud-IDs
  können außerdem an neue, legitime Mieter recycelt werden, daher Monitor.
- **IPs sind schwach.** Für Mercenary-Spyware bringt IP-Blocking wenig Wert und
  viel False-Positive-Risiko. Besser für retrospektives Hunting als für Blocking.
- **Historische Attribution ≠ aktuelle Bösartigkeit.** Aufnahme bedeutet nicht,
  dass eine Domain/IP heute noch dem Akteur gehört oder aktiv ist.
- **Blocking ist ein Stopgap, keine Infektionserkennung.** Ein Treffer heißt
  "etwas wollte dorthin", ein ausbleibender Treffer heißt nicht "sauber". Für die
  eigentliche Geräteprüfung bleibt das Zero-Breach-Toolkit (oder MVT) zuständig.

## Aktualisierung

Mercenary-Infrastruktur rotiert schnell. Die Listen alle paar Wochen neu
erzeugen. Quelle aktualisieren mit dem Toolkit:

```
python scan/zero-breach-toolkit.py --update
```

und anschließend die gefilterten Feeds neu ableiten (starke Tiers + `primary`,
Shared-Hosting raus, Domains/IPs/Cloud-FQDNs getrennt).
