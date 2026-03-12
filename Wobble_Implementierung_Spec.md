# Wobble Animation — Implementierungs-Spezifikation

Plattformunabhängige Beschreibung für Web- und iOS-Entwickler.
Ziel: Beide Implementierungen sollen visuell identisch aussehen und sich gleich verhalten.

---

## 1. Lottie-Animation

- **Datei:** `WobbleTest1.json` (beiliegend)
- **Originalauflösung:** 1200 × 1200 px
- **Framerate:** 16 fps
- **Loop:** Endlos, autoplay
- **Inhalt:** Drei organische Blob-Shapes (lila, pink, blau), die sich langsam hin- und herbewegen

---

## 2. Layout & Positionierung

- **Hintergrundfarbe:** Weiss (`#FFFFFF`)
- **Viewport:** Fullscreen, kein Scrolling
- **Animationsgrösse:** 200% der Viewport-Breite und -Höhe (also doppelt so gross wie der Bildschirm)
- **Zentrum der Animation:** Horizontale Mitte des Screens, vertikal auf der unteren Bildschirmkante (letzter Pixel-Row)
- **Vertikaler Offset:** Animation ist zusätzlich 5% der Viewport-Höhe nach oben verschoben
- **Clipping:** Alles ausserhalb des sichtbaren Viewports wird abgeschnitten — man sieht nur die obere Hälfte der Animation
- **Skalierungspunkt (transform-origin):** Immer das Zentrum der Animation (= untere Bildschirmkante). Die Skalierung erfolgt von dort aus gleichmässig in alle Richtungen.

---

## 3. Blur-Overlay

- **Typ:** Gaussian Blur über die gesamte Animation
- **Blur-Radius:** 45 px (bezogen auf die Standard-Viewport-Grösse; ggf. proportional skalieren)
- **Methode Web:** `backdrop-filter: blur(45px)` auf einem fullscreen-Overlay über der Animation
- **Methode iOS:** `UIVisualEffectView` mit Custom-Blur oder `CIGaussianBlur` mit Radius 45 auf dem Lottie-Layer
- **Resultat:** Die Shapes sind nur noch als weiche, verlaufende Farbwolken erkennbar — keine scharfen Kanten

---

## 4. Deckkraft

- **Opacity der Animation:** 80% (`0.8`)
- **Blur-Overlay:** 100% deckend (der Blur selbst, nicht eine zusätzliche Farbfläche)

---

## 5. Audio-reaktive Skalierung

### Audioquelle
- **Input:** Mikrofon des Geräts (Echtzeit)
- **Berechtigung:** Muss vom User aktiv ausgelöst werden (Button-Klick), niemals automatisch

### Frequenzanalyse
- **Methode:** FFT-basierte Frequenzanalyse des Mikrofon-Inputs
- **FFT-Grösse:** 512 Bins
- **Relevanter Bereich:** Nur die unteren 60% der Frequenz-Bins auswerten (tragen die meiste Energie)
- **Smoothing:** `0.3` auf dem Analyser (zeitliche Glättung der FFT-Daten)

### Lautstärke-Berechnung
1. Durchschnitt der unteren 60% der Frequenz-Bins bilden
2. Normalisieren auf Bereich 0.0–1.0 (durch 255 teilen)
3. Signal mit Faktor **5×** verstärken
4. Auf Maximum 1.0 clampen

### Skalierung
- **Minimaler Scale (Stille):** `1.0×`
- **Maximaler Scale (volle Lautstärke):** `2.0×`
- **Formel:** `scale = 1.0 + volume × (2.0 − 1.0)`
- **Skalierungspunkt:** Zentrum der Animation (= untere Bildschirmkante minus 5% Offset)

### Smoothing / Easing
- **Attack (Lautstärke steigt):** Schnell, Interpolationsfaktor `0.35`
- **Release (Lautstärke sinkt):** Langsamer, Interpolationsfaktor `0.12`
- **Formel pro Frame:**
  ```
  speed = (targetScale > currentScale) ? 0.35 : 0.12
  currentScale += (targetScale − currentScale) × speed
  ```
- **Update-Rate:** Jeden Frame (requestAnimationFrame / Display-Link, ~60 fps)
- **Übergangszeit CSS/CA:** 80ms ease-out als zusätzliche Glättung

---

## 6. UI-Elemente (optional, für Debug/Demo)

- **Mikrofon-Button:** Zentriert im Screen, verschwindet nach Aktivierung
- **Pegel-Anzeige:** Kleine Leiste am unteren Bildschirmrand mit:
  - Label "Pegel"
  - Farbiger Fortschrittsbalken (Gradient: Lila → Pink → Blau)
  - Numerischer Skalierungswert (z.B. "1.34×")
- Diese Elemente liegen immer über dem Blur-Overlay (höchster z-index)

---

## 7. Zusammenfassung der Werte

| Parameter                  | Wert                          |
|----------------------------|-------------------------------|
| Hintergrund                | `#FFFFFF`                     |
| Animationsgrösse           | 200% × 200% des Viewports    |
| Animations-Zentrum Y       | Untere Bildschirmkante − 5vh  |
| Animations-Zentrum X       | Horizontale Mitte             |
| Clipping                   | Viewport-Grenzen              |
| Blur-Radius                | 45 px                         |
| Deckkraft Animation        | 80%                           |
| Scale bei Stille           | 1.0×                          |
| Scale bei max. Lautstärke  | 2.0×                          |
| FFT-Grösse                 | 512                           |
| Frequenz-Bins genutzt      | Untere 60%                    |
| Signal-Verstärkung         | 5×                            |
| Attack-Speed               | 0.35                          |
| Release-Speed              | 0.12                          |
| Analyser-Smoothing         | 0.3                           |
| Transition-Easing          | 80ms ease-out                 |
| Framerate Lottie           | 16 fps                        |
| Loop                       | Endlos                        |
