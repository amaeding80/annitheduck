# Anni the Duck — Reel Editing Prompt

Nutze diesen Prompt am Anfang jeder neuen Session um einen Reel im etablierten Stil zu erstellen.

---

## Prompt

Ich möchte einen Instagram Reel / YouTube Short aus einem Video von Anni the Duck schneiden.

---

### Technische Basis
- Format: 1080x1920px (9:16 Hochformat)
- Länge: ca. 25 Sekunden
- Output: MP4, libx264, CRF 16, preset fast
- Audio: Original-Stimme + leise Hintergrundmusik (Musik ca. -18dB unter Stimme)
- Python-Script mit PIL, ffmpeg, numpy

---

### Basis-Video Vorbereitung (base_clean.mp4)
ffmpeg-Filter-Chain:
```
crop=608:1030:700:50, scale=1080:1920,
eq=saturation=1.3:brightness=0.03:contrast=1.1:gamma=0.95,
vignette=PI/3.5,
unsharp=5:5:0.6
```
- **Crop**: Gesicht/Oberkörper aus dem Originalvideo ausschneiden — Koordinaten je nach Video anpassen
- **Scale**: auf 1080x1920 hochskalieren
- **eq**: leicht erhöhte Sättigung (+30%), minimale Helligkeit, Kontrast leicht erhöht
- **vignette=PI/3.5**: dunkler, weicher Rand rundherum für Tiefe und Fokus auf die Person
- **unsharp=5:5:0.6**: leichtes Schärfen für knackigeres Bild

### Audio-Mix (audio_mix.aac)
- Originalton: 0dB (Stimme klar und vorne)
- Hintergrundmusik: ca. -18dB (kaum hörbar, nur Atmosphäre)
- Mix mit ffmpeg `amix` oder `volume` filter

---

### Schriften
- **Ankerwörter / Schlüsselwörter**: Montserrat Black, 110px — fett, dominant, alleine auf der Karte
- **Kontextwörter**: Montserrat Regular, 62px — schlanker, kleinere Gruppen
- **Spezielle emotionale Wörter**: Ivy Presto Display, 110px — kursiv-serifig, für besondere Begriffe
- Pfade: `/Volumes/Ext_aM_1TB/Anni_the_Duck/Montserrat Font/static/Montserrat-Black.ttf`
- Pfade: `/Volumes/Ext_aM_1TB/Anni_the_Duck/Montserrat Font/static/Montserrat-Regular.ttf`
- Pfade: `/Volumes/Ext_aM_1TB/Anni_the_Duck/ivy-presto-display.otf`

---

### Text-Stil & Glow
- Farbe: Weiß (#FFFFFF)
- **Glow-Effekt** (3-Pass, kein Shadow):
  - GaussianBlur(radius=26, factor=4.0) — weites diffuses Leuchten
  - GaussianBlur(radius=10, factor=3.0) — mittleres Leuchten
  - GaussianBlur(radius=4,  factor=2.0) — scharfe Kante
  - + Original-Text obendrauf
- Ergebnis: Text wirkt leuchtend, hebt sich vom Bild ab ohne harten Schatten

---

### Safe Zones & Positionierung
- CENTER_Y = 1290 (vertikale Textmitte)
- SAFE_TOP = 260 (oben frei für Profilbild/UI)
- SAFE_BOT = 1530 (unten frei)
- MAX_W = 837px (rechte Seite frei für den Instagram-Profilbereich)
- Text horizontal zentriert
- Bei zu breitem Text: Schriftgröße schrittweise verkleinern (×0.92) bis er passt

---

### Untertitel-Karten-System
- **IMMER nur eine Zeile pro Karte** — niemals mehrzeilig
- **Ankerwörter stehen ALLEINE** auf einer Karte (z.B. "öffentlich", "Mowky", "Vorwürfe", "Sexgeschichten")
- **Kontextwörter** in kleinen Gruppen (2–3 Wörter) als eigene Karte
- Timing: exakt an Original-SRT ausrichten
- Ankerwörter bekommen etwas mehr Anzeigezeit
- Alle Wörter aus dem Original müssen erscheinen, nichts weglassen

**Cards-Format:** `(start, end, text, style, do_arc, do_marker)`

Die Cards-Liste wird für jedes Video neu erarbeitet — basierend auf dem Original-SRT und dem Inhalt. Welche Wörter Ankerwörter sind, welche den Arc bekommen und ob ein Marker-Effekt passt, hängt vom jeweiligen Video ab. Das Prinzip bleibt gleich, die Auswahl variiert immer.

---

### Animations-Effekte

**1. Oranger Bogen-Unterstrich (Arc)**
- Farbe: Orange (#D25514)
- Position: direkt unter dem Wort, ARC_YOFF = 16px Abstand
- Höhe: 68px
- Draw-Animation: 14 Frames bei 30fps (~0.47s) — zieht von links nach rechts
- Optik: organischer Pinselstrich mit Sinus-Wellung (Amplitude 7px) und variierender Stärke (4–12px)
- Bleibt bis Ende der Karte sichtbar (eingefroren nach Draw)
- Genutzt bei: Mowky (Sub 3), Sexgeschichten, Vorwürfe

**2. Schwarzer Filzstift-Marker (Durchstreichung)**
- Farbe: Fast-Schwarz (#0C0C0C)
- Höhe: 170px — sehr dick, übermalt das Wort komplett
- Draw-Animation: 5 Frames bei 30fps (~0.18s) — extrem schnell, aggressiv
- Startet: 0.18s nach Kartenstart (MARKER_DELAY = 0.18s)
- Position: vertikal mittig über dem Text, 30px nach unten verschoben
- Breite: Textbreite + 20px (je 10px Überstand links/rechts)
- Optik: dicke Ellipsen entlang Sinus-Wellung, wirkt wie echter Filzstift
- Genutzt bei: "Vergewaltigung" — Wort erscheint kurz, wird dann sofort komplett übermalt

---

### ffmpeg-Strategie
- Alle Untertitel-PNGs via **ffconcat** zu EINER Subtitle-Track-Spur zusammenbauen
- Lücken zwischen Karten: transparentes Blank-PNG einsetzen
- Diesen einen Track mit einem einzigen `overlay=0:0` auf das Basisvideo legen
- Animations-Sequenzen (Arc, Marker) separat als PNG-Sequenz mit `-itsoffset` + `-framerate 30` + `eof_action=pass`
- NICHT: chained overlays für jeden Untertitel einzeln (führt zu Filtergraph-Abbrüchen)

---

### Workflow
1. Basis-Video auf gewünschten Ausschnitt (25s) schneiden → `base_clean.mp4`
2. Audio-Mix erstellen → `audio_mix.aac`
3. Original-SRT als Grundlage nehmen
4. Cards-Liste definieren (Ankerwörter allein, Kontext in Gruppen, Animationen markieren)
5. Python-Script: PNGs + Arc/Marker-Sequenzen generieren
6. ffconcat-Script für Subtitle-Track bauen
7. Finales ffmpeg-Rendering

---

### Datei-Referenz
- Beispiel-Script: `make_reel_v17.py` (im outputs-Ordner der Session)
- Beispiel-Output: `/Volumes/Ext_aM_1TB/Anni_the_Duck/Anni_Reel_v17.mp4`
- Original-SRT: `/Volumes/Ext_aM_1TB/Anni_the_Duck/test_anni.srt`

---

### Stil-Prinzipien
- Einfach und visuell klar — kein Overdesign
- Ankerwörter sollen im Kopf hängen bleiben
- Glow statt Shadow
- Ivy Presto nur für emotional aufgeladene Begriffe
- Pünktliche Lieferung hat Priorität
