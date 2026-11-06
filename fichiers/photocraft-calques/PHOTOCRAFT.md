# Affiches et images en calques PhotoCraft (.pcraft)

Rayane veut **toujours** une version PhotoCraft en calques de chaque image livrée (affiche, montage photo,
planche, mise en situation) : chaque élément séparé, nommé en français, textes modifiables. Il débute :
noms de calques explicites, groupes repliés, un `COMMENT-UTILISER.txt` court et les polices (OFL) dans le zip.

Outil : `montage/photocraft.py` (`Doc` -> `.pcraft`, PhotoCraft 0.9 headless, `./setup.sh --artcraft`).

## Méthode

1. Garder le script qui a fabriqué l'image livrée et rejouer **les mêmes calculs (mêmes graines)**, mais
   déclarer chaque étape comme un calque au lieu de la fusionner.
2. Construire (`D.build`), aplatir (`render_pcraft`) et comparer à l'image livrée (`compare`) :
   viser un écart moyen < 0,01 ; regarder la planche côte à côte et mesurer les décalages des textes.

## Équivalences exactes

| Dans le script | Calque PhotoCraft |
|---|---|
| `apply_ink(img, a, couleur)` | `D.ink(nom, a, couleur)` (Normal) |
| `1-(1-img)(1-a*couleur)` (écran) | `D.ink(nom, a, couleur, blend="Screen")` |
| `img + delta` (delta ≥ 0) | gris `delta`, `"Linear Dodge (Add)"` ; delta signé : + un calque `"Subtract"` |
| `img * gain` (gain ≤ 1) | gris `gain`, `"Multiply"` ; gain > 1 : `"Color Dodge"` avec `1 - 1/gain` |
| bruit additif (grain) | gris `.5 + bruit/2`, `"Linear Light"` (seulement pour du bruit : pas de neutre exact en 8 bits) |
| `roughen(a)` | `Rough(...).edge(a)` par calque + **un seul** voile `veil()` (dans le papier) + trous `holes` |
| densité d'encre, opacité, trous | masque de groupe : `D.group_props[groupe] = dict(mask=...)` (suit les textes modifiés) |
| texte PIL `text_mask` | `D.text(..., box=ibox(masque, x, y))` aligné sur l'encre ; décalage rouge : `shift` |
| grande calligraphie en relief (remplissage calculé) | calque image exact + texte modifiable masqué avec styles (ombre, lueur, biseau) |
| lueur gaussienne d'un titre | style `outerGlow` (deux empilés avec `add=True` : proche + large), régler sur le profil mesuré |
| ombre portée d'un sticker / billet | style `dropShadow` (angle = direction de la lumière, size ≈ 2,7 × sigma) |
| mise en situation | `D.smart` (objet dynamique remplaçable) + calques de gain + masque de groupe (bords déchirés) |

## Pièges

- Taille de texte max 1296 pt (le `Doc` crée à 1200 puis agrandit). Inter : passer `weight` (fait par `Doc`).
- Rotation PhotoCraft positive = sens horaire (PIL/cv2 : anti-horaire) -> `rotate=-angle_pil`.
- Interlettrage : `tracking_px / taille * 1000`. Le crénage diffère un peu de PIL (quelques px sur un titre très espacé).
- Les membres d'un groupe doivent être contigus dans la pile (sinon `build` s'arrête : un groupe éclaté remonterait
  tout le fond au-dessus du reste). Pour poser un calque calculé plus tard : `D.layers.insert(i, D.layers.pop())`.
- Texte mixte arabe + latin avec ponctuation (« إلى أين؟ · VERS OÙ ? ») : l'algorithme bidi déplace le « ? ».
  Un calque par langue. Les dates en chiffres arabes (٠٣ . ١١ . ٢٠٢٦) s'affichent dans l'ordre arabe : normal.
- Chiffre arabe zéro « ٠ » = un point : sur un panneau ou une distance, préférer « 0 ».
- Masques de groupe texturés : limiter la texture à la zone utile (1 ailleurs), sinon le fichier grossit (bruit).
- Poids : un bruit plein cadre pèse lourd (≈ 10 Mo) ; fondre les voiles fixes dans le calque papier.
- Livraison : zips par série (SendUserFile, ≤ 400 Mio) ; `device_commit_files` refuse > 25 Mo par fichier.
