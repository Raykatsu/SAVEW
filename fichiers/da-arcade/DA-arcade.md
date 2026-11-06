# DA « HORS CHAMP FIGHTERS » : jeu de combat rétro en pixel art

Rayane adore cette DA (écran de sélection façon jeu de combat des années 90 : sprite, stats, coups spéciaux,
couleurs alternatives, écran cathodique) et a demandé de la développer. Ton : drôle et stylé à la fois (stats absurdes,
coups spéciaux qui jouent sur la photo, citations de victoire). Jeu original : jamais de vrai nom, logo ou réplique
d'une franchise existante (ni « Fatality », ni console réelle).

## Module
```
PYTHONPATH=~/montage-studio python3 -m montage.arcade persos.yaml -o out [--ecrans select,vs] [--formats 4:5,9:16]
```
Chaque écran donne `NN-nom_4x5.pcraft` (calques PhotoCraft) + `.jpg` (2160x2700 ou 2160x3840).
Exemple commenté : `examples/arcade/persos-exemple.yaml`. Rendu : ~1 min par écran (2 vCPU).

## Écrans (`type:`)
| type | contenu | style |
|---|---|---|
| `select` | sprite buste + fiche (stats en segments, coup spécial + manip, super) + roster 8 cases + curseur P1 | 16 bits |
| `vs` | fond rouge/bleu en diagonale, deux bustes (P2 en miroir), VS en flammes | 16 bits façon Neo Geo |
| `combat` / `ko` | décor = photo d'un perso pixelisée, barres de vie, chrono, jauges de super ; K.O. : perdant grisé et penché, combo | 16 bits |
| `challenger` | « UN NOUVEAU CHALLENGER APPROCHE ! », teinte rouge, bandes d'alerte | 16 bits |
| `bonus` | photo pixelisée, score, temps, trajectoire en pointillés (`trajectoire` en coordonnées photo) | 16 bits |
| `gameboy` | 4 verts tramés Bayer, grille LCD, boîte de dialogue (texte ASCII) | console de poche |
| `nes` | palette 8 bits, carton de stage, 1UP / HI, coeurs, vies, flocons (`contour: false` sur la neige) | 8 bits |
| `victoire` | décor teinté coucher de soleil, citation, stats du combat | 16 bits |

## Recettes
- Sprite depuis une photo : détourage rembg, palettes k-means Lab séparées peau (7) / reste (15), filtre de mode,
  contour sombre 1 px, décontamination des bords (érosion 5x5), agrandissement au plus proche (U=5, U=12 en 8 bits).
- Vrai sprite dessiné (perso redessiné, poses animées) : prototype `examples/arcade/sprite-chibi/`.
  Rayane l'a trouvé moche : il préfère son perso fait avec une IA (planche à la chapka grise).
- **Sprites fournis** (planche d'IA, dessin) : `sprites: {buste: png | {"4:5": png, "9:16": png}, taille: png, ko: png}`
  et `sprites_miroir: {buste: false, ko: false}` dans la fiche du perso. Le png est à la taille de la grille
  (1 case = 1 pixel, U=5) : VS (buste collé au bord s'il y est coupé), Round 1 (en pied, pieds au-dessus des jauges),
  K.O. (au sol, sans gris ni rotation). Recette pour passer une planche d'IA en vrai pixel art :
  `examples/arcade/planche-ia/` (détourage, grille, palette commune, énergie en aplats, GIF).
- Ne jamais reprendre le logo d'une vraie franchise présent sur une image fournie (ex. « Street Fighter II »).
- Polices : Press Start 2P (ASCII seulement, ses majuscules accentuées sont ratées) et Silkscreen pour l'accentué
  (`font_for` choisit tout seul). Les livrer dans `polices/` avec un COMMENT-UTILISER.txt.
- Gens dans le décor : effacés (inpaint) ou gardés en spectateurs ; la pixelisation rend la foule méconnaissable.
  Toujours rappeler à Rayane de demander l'accord de ses potes avant de publier.

## Générateurs d'images IA (essai du 04/11/2026)
- Gamma : plan gratuit, 0 crédit restant -> génération refusée.
- Canva `generate-image` : marche, mais l'image reste chez Canva ; le conteneur ne peut pas la télécharger
  (liste blanche réseau). Il faut que le PC soit connecté (téléchargement après accord) ou que Rayane joigne l'image
  à la conversation. Le prompt Canva doit reprendre les mots de Rayane tels quels.
- Les images « pixel art » d'IA ont de faux pixels (tailles inégales, bords flous) : les recaler sur une vraie grille
  (pas dominant par autocorrélation, couleur dominante par case, palette réduite, contour) avant de les intégrer.
