# Konzept: Codex – systematische Rechtssammlung

Stand: 26.11.2026 · Verantwortlich: Markus Keller · Herkunft: MarkusKeller8200/EduKSX-Architektur#119 und MarkusKeller8200/EduKSX-Architektur#121 (am 26.11.2026 hierher
verschoben, Architektur verweist auf dieses Dokument) · Status: **BESCHLOSSEN** 26.11.2026 (CX1–CX16), **ZU PRÜFEN**:
Funktionsliste §2 · Verbindlichkeit: Stufen nach [KONZEPT-VORLAGE.md](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-VORLAGE.md).

Ergänzt den Abschnitt `codex` der [Modulübersicht](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/MODULE-UEBERSICHT.md). Grundlagen:
[Anmeldung](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-ANMELDUNG.md), [Design](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-DESIGN.md) (DS4, Ausnahme DS14), [Plugins](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-PLUGINS.md) (P2, P7),
[Datenschutz ZH](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-DATENSCHUTZ-ZH.md) (Cloudflare nur öffentliche Inhalte, «anbieten, dann löschen»).

## 1. Ziel

Codex ist die **systematische Rechtssammlung** von Schul- und politischen Gemeinden: Erlasse und Reglemente mit
Systematik, Versionen und Gültigkeit, öffentlich publiziert. Jede Gemeinde kann Codex als **ihre eigene Rechtssammlung**
nutzen und auf ihrer Homepage einbinden. Später erlaubt Codex die **Gegenüberstellung** mehrerer Gemeinden.

**Mandant** ist die einzelne Schul- oder politische Gemeinde (Organisation nach [Rollen](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-ROLLEN.md)).

**Nichtziele:** keine Unterschriften (→ `sign`), keine allgemeine Dokumentablage (→ `intranet`), keine
Vernehmlassungs- oder Genehmigungsabläufe (→ `flow`).

## 2. Öffentlichkeit (CX1 BESCHLOSSEN, Funktionsliste ZU PRÜFEN)

Codex ist **öffentlich wie das Glossary**: Lesen braucht keine Anmeldung. Welche Funktion öffentlich ist, wird vor der
Umsetzung im Repo EduKSX-Codex noch einmal geprüft. Vorschlag (**EMPFOHLEN**):

| Öffentlich, ohne Anmeldung | Mit Anmeldung (Rolle nach [Rollen](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-ROLLEN.md)) |
| --- | --- |
| Rechtssammlung einer Gemeinde lesen, durchsuchen, nach Systematik blättern | Erlasse erfassen, bearbeiten, Versionen anlegen (`editor`) |
| aktuelle und frühere Fassungen, Gültigkeit, Änderungsverlauf | Fassung freigeben und publizieren (`admin` der Gemeinde) |
| Gegenüberstellung mehrerer Gemeinden | Muster-Einträge zuordnen oder deaktivieren (§5) |
| Einbettung auf der Gemeinde-Homepage, lesende API (§4) | CSS und Template für die Einbettung pflegen (§4.2) |
| PDF/Druck einer Fassung | Mandanten, Rollen, Einstellungen (`admin`) |

Öffentliche Inhalte dürfen über Cloudflare ausgeliefert werden ([Datenschutz ZH](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-DATENSCHUTZ-ZH.md)); Konten
und Entwürfe nicht.

## 3. Anmeldung (CX2 BESCHLOSSEN)

**Identisch wie Glossary:** lokales Login als Standard (Passwort plus zweiter Faktor bzw. Passkey), Anbindung an
EduKSX-Ident optional; Entra ID und Google wie in allen Modulen ebenfalls optional. Regeln, Notfallkonten und
Umschalten nach [KONZEPT-ANMELDUNG.md](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-ANMELDUNG.md) (AN1–AN5), umgesetzt mit dem Baustein «Anmeldung» aus
`eduksx/core`. Eine öffentliche Selbstregistrierung gibt es in Codex nicht (kein Modus `mixed`).

## 4. Einbindung bei der Gemeinde

### 4.1 Drei Wege (CX3 BESCHLOSSEN)

| Weg | Für wen | Eigenschaften |
| --- | --- | --- |
| **JS-Baustein** (Standard) | Homepages, die ein Script erlauben | ein `<script>`-Tag plus Element `<eduksx-codex gemeinde="…">` (Web Component mit Shadow DOM, wie im [Plugin-Konzept](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-PLUGINS.md)); Tiefenlinks auf einzelne Erlasse über die Adresse der Homepage; Inhalte für Suchmaschinen über die öffentliche Codex-Seite mit `canonical` |
| **iFrame** (Rückfall) | CMS ohne eigene Scripts | eine Adresse je Gemeinde, Höhe passt sich per `postMessage` an; Optik über das CSS der Gemeinde (§4.2) |
| **Öffentliche API** | Gemeinden bzw. Webdienstleister, die selbst darstellen | lesend, ohne Anmeldung, nach [API-Guideline](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-API-GUIDELINE.md) (`/codex/v1`, OpenAPI); Ratenbegrenzung; nur publizierte Fassungen |

Für den JS-Baustein gelten die Sicherheitsregeln aus [Plugins](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-PLUGINS.md) P6 sinngemäss (Script nur vom
Codex-Host, feste Version im Pfad bzw. Subresource Integrity). Der iFrame ist hier – anders als bei Plugins in
unseren Modulen (P4) – als Rückfall für fremde Homepages erlaubt, weil er nur öffentliche Inhalte ohne Anmeldung zeigt.

Die Einbettung zeigt nur **publizierte** Inhalte. Die Bearbeitung findet immer in Codex selbst statt, nie in der
Einbettung.

### 4.2 Eigenes CSS je Gemeinde (CX4 BESCHLOSSEN, Ausnahme zu DS4)

- Jede Gemeinde kann für **die eingebettete Rechtssammlung** eigenes CSS hinterlegen, damit sie wie ihre Homepage aussieht.
- Für weniger Versierte erstellen wir ein **CSS-Template nach dem Vorbild der Homepage** der Gemeinde (Farben,
  Schrift, Abstände); die Gemeinde übernimmt es oder passt es an.
- **Geltungsbereich:** nur die Einbettung (JS-Baustein im Shadow DOM, iFrame). Codex selbst, seine öffentliche Seite
  und die Administration folgen weiterhin **DS4** (ganzes Theme über bekannte Tokens, Kontrastprüfung) – festgehalten
  als **DS14** im [Design-Konzept](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-DESIGN.md).
- **Prüfung beim Speichern (Pflicht):** kein `@import`, keine `url()` auf fremde Adressen (Schriften und Bilder nur
  von der Homepage-Domain der Gemeinde oder aus Codex), kein `expression`/`behavior`, Grössenbeschränkung; das CSS
  wirkt nur innerhalb des Bausteins.
- **Barrierefreiheit:** Codex führt beim Speichern eine Kontrastprüfung durch und **warnt**; die Verantwortung für das
  eigene CSS liegt bei der Gemeinde. Struktur, Überschriften und Tastaturbedienung liefert der Baustein unverändert.

## 5. Mustersystematik (CX5–CX8 BESCHLOSSEN)

- **Vorschlag (CX5):** Sind alle Gemeinden aufgenommen, stellen wir einen **Vorschlag für Nummerierung und Namen** der
  Dokumente vor (Mustersystematik). Gepflegt wird er zentral von EduKSX. Die Nutzung ist **optional**.
- **Liste je Gemeinde (CX6):** Die Gemeinde geht die Liste der Muster-Einträge durch und entscheidet je Eintrag, ob sie
  das Dokument hat. Hat sie es nicht, **deaktiviert** sie den Eintrag.
- **Zuordnen statt umbenennen (CX7):** Die Gemeinde ordnet ihre Erlasse den Muster-Einträgen zu. Ihre **eigene Nummer
  und Bezeichnung bleiben** – oder sie übernimmt die des Musters. Die Zuordnung ist die Grundlage für die
  Gegenüberstellung.
- **Deaktiviert im Vergleich (CX8):** In der eigenen Rechtssammlung ist ein deaktivierter Eintrag unsichtbar. In der
  Gegenüberstellung erscheint die Gemeinde mit **«kein Erlass»** – das ist selbst eine Aussage.
- Ändert sich die Mustersystematik, bleibt jede Zuordnung erhalten; neue Einträge erscheinen der Gemeinde als
  «noch nicht entschieden».

## 6. Aufnahme und Pflege der Dokumente

### 6.1 Formate (CX9, CX10 BESCHLOSSEN)

- Codex nimmt **nur PDF** an, dazu optional je Erlass eine **Markdown-Datei (MD)** mit dem gegliederten Text
  (Titel, Abschnitte, Artikel). Andere Formate (Word, Bilder) weist Codex mit einem Hinweis ab; ein ZIP darf PDF und
  MD enthalten.
- **MD ist die Grundlage für die MCP-Funktionen** (MCP nach [API-Guideline](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-API-GUIDELINE.md) D5, Werkzeuge `codex_<operation>`: Erlasse und Artikel
  finden, zitieren, vergleichen)
  sowie für Volltextsuche, Artikelvergleich zwischen Gemeinden und den Unterschied zwischen Fassungen.
- Liefert die Gemeinde kein MD, **bietet Codex an, es zu erstellen** (Auslesen des PDF, bei Scans mit Texterkennung,
  KI-gestützt). Ein von uns erstelltes MD wird erst nach **Freigabe durch einen Menschen** verwendet (CX11).
- **Massgebend ist die publizierte Originalfassung (PDF)** (CX10). Bei von uns erstellten MD-Dateien zeigt Codex immer
  den Hinweis «Massgebend ist die publizierte Originalfassung» mit Link auf das PDF; bei MD der Gemeinde entscheidet sie.

### 6.2 Einlieferung (CX12, CX13 BESCHLOSSEN)

Alle Wege führen in denselben **Eingangskorb** der Gemeinde; danach gilt 6.3.

| Weg | Wie | Absicherung |
| --- | --- | --- |
| **Ziehen und Ablegen** | einzeln, mehrere oder ZIP in Codex | angemeldet (`editor`) |
| **E-Mail** | eigene Adresse je Gemeinde; Anhänge (PDF, MD, ZIP) landen im Eingangskorb | nur freigegebene Absenderadressen, SPF/DKIM geprüft; nie direkt publiziert |
| **OneDrive / SharePoint** | überwachter Ordner der Gemeinde über Microsoft Graph | Zustimmung durch die Gemeinde (OAuth), nur Leserecht auf den Ordner; Anbindung über EduKSX-Connectors |
| **Dropbox** | überwachter Ordner über die Dropbox-API | wie OneDrive |
| **Übernahme von der Homepage** (Ersterfassung) | Codex holt die PDF von der heutigen Seite der Gemeinde | öffentliche Daten, **keine besondere Freigabe** (CX13); EMPFOHLEN: `robots.txt` beachten, Abfragen drosseln, Gemeinde informieren |
| **Excel-Vorlage** (EMPFOHLEN, Ersterfassung) | eine Zeile je Erlass mit Nummer, Titel, Daten, Dateiname | angemeldet (`editor`) |

**Ersterfassung (CX12):** durch die Gemeinde selbst oder durch uns als Dienstleistung – die Gemeinde wählt. In beiden
Fällen gibt die Gemeinde frei.

### 6.3 Erkennen und Einordnen (CX14–CX16 BESCHLOSSEN)

Jedes eingehende Dokument durchläuft:

1. **Doppelt?** Gleiche Datei (Prüfsumme) wie vorhanden → verworfen mit Hinweis.
2. **Neue Fassung?** Nummer bzw. Titel eines bekannten Erlasses → als neue Fassung angehängt; Codex zeigt den
   Unterschied zur bisherigen Fassung (auf Basis MD) und schlägt das Datum vor, ab dem die alte ausser Kraft tritt.
3. **Neuer Erlass?** Codex liest Titel, Beschlussdatum, Inkrafttreten und beschliessendes Organ aus und **schlägt die
   Kategorie** in der Systematik vor (KI-gestützt, CX14).
4. **Nomenklatur stimmt?** Entspricht Dateiname oder Titel genau der Mustersystematik oder einer Nummer, die die
   Gemeinde bereits zugeordnet hat, wird das Dokument **direkt eingeordnet, ohne Prüfung** (CX15). Sonst bleibt es mit
   Vorschlag im Eingangskorb, bis ein Mensch bestätigt.

**Publikation nach direkter Einordnung (CX16):** je Gemeinde einstellbar – Vorgabe «nur einordnen» (Publikation per
Klick, auch gesammelt «alle freigeben»), wahlweise «direkt publizieren». Auch bei «direkt publizieren» bleibt ein von
uns erstelltes MD bis zur Freigabe unveröffentlicht; publiziert wird dann zuerst das PDF. Jede automatische Einordnung
und Publikation wird protokolliert und lässt sich zurücknehmen.

## 7. Daten und Aufbewahrung

- Personendaten: wenig (Konten der Bearbeitenden, Namen in Erlassen nur, wo sie zum Rechtstext gehören).
- Fassungen werden nicht gelöscht, sondern ausser Kraft gesetzt; der Verlauf bleibt öffentlich.
- Beim Vertragsende gilt «anbieten, dann löschen» mit Export PDF/A plus CSV ([Datenschutz ZH](https://github.com/MarkusKeller8200/EduKSX-Architektur/blob/main/docs/KONZEPT-DATENSCHUTZ-ZH.md)).

## 8. Entscheide

| Nr. | Frage | Entscheid | Status |
| --- | --- | --- | --- |
| CX1 | Öffentlichkeit | öffentlich lesbar wie Glossary; Funktionsliste §2 vor der Umsetzung prüfen | BESCHLOSSEN 26.11.2026, Liste ZU PRÜFEN |
| CX2 | Anmeldung | wie Glossary: lokales Login Standard, Ident optional (AN1–AN5), keine Selbstregistrierung | BESCHLOSSEN 26.11.2026 |
| CX3 | Einbindung | JS-Baustein (Standard), iFrame (Rückfall) und öffentliche lesende API | BESCHLOSSEN 26.11.2026 |
| CX4 | Eigenes CSS | je Gemeinde, nur für die Einbettung, geprüft beim Speichern; Template nach Homepage; Ausnahme DS14 zu DS4 | BESCHLOSSEN 26.11.2026 |
| CX5 | Mustersystematik | Vorschlag für Nummerierung und Namen, sobald alle Gemeinden aufgenommen sind; optional | BESCHLOSSEN 26.11.2026 |
| CX6 | Liste je Gemeinde | Gemeinde entscheidet je Muster-Eintrag; nicht vorhanden → deaktivieren | BESCHLOSSEN 26.11.2026 |
| CX7 | Eigene Dokumente | Zuordnung zum Muster; eigene Nummer und Bezeichnung bleiben oder Muster übernehmen | BESCHLOSSEN 26.11.2026 |
| CX8 | Deaktiviert im Vergleich | eigene Sammlung: unsichtbar; Gegenüberstellung: «kein Erlass» | BESCHLOSSEN 26.11.2026 |
| CX9 | Formate | nur PDF, optional MD; fehlt MD, bieten wir die Erstellung an; MD ist Grundlage für MCP, Suche und Vergleich | BESCHLOSSEN 26.11.2026 |
| CX10 | Massgebende Fassung | publiziertes Original-PDF; Hinweis mit Link immer bei von uns erstelltem MD | BESCHLOSSEN 26.11.2026 |
| CX11 | KI beim Auslesen und Zuordnen | ja, Freigabe durch einen Menschen | BESCHLOSSEN 26.11.2026 |
| CX12 | Ersterfassung | durch die Gemeinde oder durch uns als Dienstleistung, wählbar | BESCHLOSSEN 26.11.2026 |
| CX13 | Übernahme von der Homepage | ohne besondere Freigabe (öffentliche Daten) | BESCHLOSSEN 26.11.2026 |
| CX14 | Erkennen beim Hochladen | Doppelte, neue Fassungen und neue Erlasse erkennen; Kategorie vorschlagen | BESCHLOSSEN 26.11.2026 |
| CX15 | Nomenklatur stimmt | direkte Einordnung ohne Prüfung | BESCHLOSSEN 26.11.2026 |
| CX16 | Publikation nach direkter Einordnung; Einlieferung | je Gemeinde einstellbar (Vorgabe «nur einordnen»); Einlieferung per Ziehen und Ablegen, E-Mail, OneDrive/SharePoint, Dropbox | BESCHLOSSEN 26.11.2026 |

## 9. Verlauf

- 26.11.2026: Angelegt (MarkusKeller8200/EduKSX-Architektur#119) mit den Vorgaben von Markus Keller; CX1–CX8 im Chat entschieden.
- 26.11.2026: §6 Aufnahme und Pflege der Dokumente (MarkusKeller8200/EduKSX-Architektur#121); CX9–CX16 im Chat entschieden.
- 26.11.2026: Aus EduKSX-Architektur ins Repo EduKSX-Codex verschoben.
