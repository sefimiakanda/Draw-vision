# 🎨 Draw-vision

![Python](https://img.shields.io/badge/Python-3.10-blue)
![OpenCV](https://img.shields.io/badge/OpenCV-green)
![MediaPipe](https://img.shields.io/badge/MediaPipe-orange)
![License](https://img.shields.io/badge/License-MIT-yellow)

**Dessine avec ta main.** Draw-vision est une application de dessin sans contact : une webcam et une main. Pas de souris, pas de clavier, pas d'écran tactile.

Le projet s'appuie sur la **vision par ordinateur** (Computer Vision), le **suivi de la main** (Hand Tracking) et la **reconnaissance de gestes** (Hand Gesture Recognition) pour transformer des mouvements naturels en interface utilisateur.


## ✨ Fonctionnalités

- Détection de la main en temps réel avec **21 points de repère** (landmarks) via MediaPipe
- Dessin au doigt sur un canvas virtuel superposé au flux vidéo
- Menu de couleurs sélectionnable par geste (Bleu, Vert, Rouge, Jaune) et bouton **Clear All**
- Gomme contrôlée par la paume de la main
- Lissage du tracé par **Exponential Moving Average (EMA)** pour réduire les tremblements
- Sauvegarde du dessin en **PNG** (fond blanc) par un simple pouce levé
- Affichage en direct du geste détecté, pratique pour déboguer

---

## 🖐️ Gestes

| Geste | Action |
|-------|--------|
| ☝️ Index seul levé | **Dessiner** |
| ✌️ Index + majeur levés | **Sélectionner** une couleur ou *Clear All* (pointe le bouton dans le menu) |
| 🖐️ Paume ouverte (4 doigts levés) | **Gommer** |
| 👍 Pouce levé, autres doigts repliés | **Sauvegarder** le dessin dans `output/` |

### Raccourcis clavier

| Touche | Action |
|--------|--------|
| `s` | Sauvegarder le dessin |
| `c` | Effacer le canvas |
| `q` ou `Échap` | Quitter |

---

## ⚙️ Comment ça marche

```
Webcam
  ↓  camera.py       ouvre la webcam (essaie les index 0, 1, 2)
  ↓  app.py          orchestre tout le pipeline : lit chaque image et la retourne comme un miroir
MediaPipe Hands
  ↓  hand_tracker.py détecte la main et ses 21 landmarks
gestures.py
  ↓  analyse la position des doigts → DRAW / SELECT / ERASE / SAVE / NONE
app.py
  ↓  transforme le geste en action (trait, couleur, gomme, sauvegarde)
canvas.py (NumPy)
  ↓  calque de dessin fusionné avec la vidéo
ui.py
  ↓  menu, barre d'aide et statut, puis affichage avec OpenCV
output/drawing_AAAAMMJJ_HHMMSS.png
```

1. **Capture** : OpenCV lit la webcam et l'image est retournée comme un miroir.
2. **Détection** : MediaPipe Hands localise la main et ses 21 points de repère.
3. **Reconnaissance** : `gestures.py` compare la position des extrémités des doigts à leurs articulations pour déterminer quels doigts sont levés, puis en déduit le geste.
4. **Action** : en mode dessin, la position de l'index est lissée (EMA) puis reliée au point précédent pour former un trait sur le canvas.
5. **Affichage** : le canvas (fond noir = transparent) est superposé à la vidéo, avec le menu et les informations d'état.

---

## 📁 Structure du projet

```
draw-vision/
├── main.py            # Point d'entrée : lance l'application
├── app.py             # Boucle principale : lit la webcam, associe chaque geste à une action
├── config.py          # Toutes les constantes (tailles, seuils, couleurs, textes)
├── camera.py          # Ouverture de la webcam
├── hand_tracker.py    # Wrapper MediaPipe : détection de la main et des landmarks
├── gestures.py        # Classification des gestes + lissage EMA
├── canvas.py          # Calque de dessin : traits, gomme, fusion avec la vidéo, sauvegarde PNG
├── ui.py              # Interface : menu de couleurs, barre d'aide, statut, détection du survol
├── run.sh             # Script de lancement
├── pyproject.toml   # Dépendances Python
├── .gitignore
├── README.md
└── output/            # Dessins sauvegardés (créé automatiquement, ignoré par Git)
```

Chaque fichier a une seule responsabilité : `main.py` reste minimal, `app.py` orchestre, et les autres modules (`camera`, `hand_tracker`, `gestures`, `canvas`, `ui`) se concentrent chacun sur une tâche précise. Les valeurs réglables sont toutes regroupées dans `config.py`.

---

## 🚀 Installation

**Prérequis** : Python 3.10 recommandé (MediaPipe ne prend pas en charge toutes les versions de Python, en particulier les plus récentes) et une webcam.


# 1. Cloner le dépôt
```bash
git clone https://github.com/irwinjessk/air-draw.git
cd Draw-vision
```

# 2. Installer les dépendances avec uv

Depuis la racine du projet :
```bash
uv sync
```
(ou `uv add <package>` si vous ajoutez des dépendances au fur et à mesure)


# 3. Lancer l'application.

```bash
uv run main.py
```

Sous Linux / macOS, tu peux aussi utiliser le script de lancement :

```bash
chmod +x run.sh   # une seule fois
./run.sh
```

---

## 🎮 Utilisation

1. Lance `python main.py` et place ta main devant la webcam, bien éclairée.
2. Lève l'**index** pour dessiner.
3. Lève **index + majeur** et pointe une couleur du menu pour la choisir.
4. Ouvre la **paume** pour gommer.
5. Fais un **pouce levé** (poing fermé, pouce bien vertical) pendant environ 1 seconde pour sauvegarder. Le fichier apparaît dans `output/`.

Un délai de 1,5 s entre deux sauvegardes évite de créer plusieurs fichiers à la suite.

---

## 🔧 Paramètres modifiables

Tous les réglages se trouvent dans **`config.py`** :

| Paramètre | Rôle |
|-----------|------|
| `BRUSH_SIZE` | Épaisseur du pinceau |
| `ERASER_RADIUS` | Rayon de la gomme |
| `SMOOTHING_ALPHA` | Lissage : plus petit = plus fluide mais plus lent |
| `MIN_DETECTION_CONFIDENCE`, `MIN_TRACKING_CONFIDENCE` | Sensibilité de MediaPipe |
| `SAVE_COOLDOWN` | Délai minimum entre deux sauvegardes par geste |
| `FINGER_MARGIN`, `THUMB_TIP_MARGIN`, `THUMB_FIST_MARGIN` | Seuils de reconnaissance des doigts |
| `PALETTE` | Couleurs du menu |
| `CAMERA_INDICES`, `FRAME_WIDTH`, `FRAME_HEIGHT` | Choix et résolution de la webcam |

---

## 🩺 Dépannage

- **« Impossible d'ouvrir la webcam »** : ferme les applications qui l'utilisent (Chrome, Meet, Zoom...).
- **Un geste n'est pas reconnu** : regarde la ligne `Geste: ...` affichée en bas de la fenêtre pour voir ce que le programme comprend, puis ajuste ton geste ou les seuils dans `config.py`.
- **Avertissements Qt / Wayland / protobuf dans le terminal** : ils sont sans gravité et n'empêchent pas l'application de fonctionner.

---

## 🛠️ Technologies

- [Python](https://www.python.org/)
- [OpenCV](https://opencv.org/) : capture vidéo, dessin, interface
- [MediaPipe](https://developers.google.com/mediapipe) : Hand Tracking
- [NumPy](https://numpy.org/) : manipulation du canvas

---

## 🗺️ Pistes d'amélioration

- [ ] Réglage de l'épaisseur du pinceau par geste
- [ ] Annuler / rétablir (undo / redo)
- [ ] Prise en charge de deux mains
- [ ] Export en SVG
- [ ] Reconnaissance de gestes personnalisables

---

## 👤 Auteur

Projet réalisé par Fidèle Miakanda.


---

## 📄 Licence

Ce projet est distribué sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.