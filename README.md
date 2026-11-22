# Putrelles Pro

Calcul des **poutrelles continues de plancher à corps creux** (BAEL 91 mod. 99) — version professionnelle de
`Calcul putrelles / Solive.py`.

```
pip install -r requirements.txt     # PyQt5 + python-docx
python main.py                      # lancer l'application
python -m unittest discover tests   # 34 tests du moteur de calcul
```
Python 3.8+ (testé aussi sur 3.13).

## Interface en fichier .ui
Toute la mise en page est dans **`app/ui/main_window.ui`** : ouvrez-le avec Qt Designer pour déplacer, redimensionner
ou ajouter des widgets. Le code (`app/ui/main_window.py`) le charge avec `uic.loadUi` et s'appuie sur les *noms d'objets*
(`edG`, `cbMethod`, `edBarsT`, `tbElu`, …) : gardez ces noms si vous renommez/déplacez des widgets.
Quatre widgets de dessin/tableau sont des « widgets promus » (en-tête `app.ui.views`) : `StaticScheme`, `TSectionView`,
`DiagramView`, `DataTable`. Les couleurs et le style sont dans `app/ui/style.py` (feuille de style Qt).

Exécutable + installateur : lancer `build.bat` (PyInstaller, puis Inno Setup 6 si installé → `installer_output\`).

## Ferraillage
- Vous **saisissez le ferraillage retenu** (onglet Ferraillage) : `2HA10`, `2HA10+1HA12`, `5*10+3*16`…
- **Diamètre minimal HA10** : tout diamètre inférieur est refusé (message + champ en rouge).
- Avertissement si la section choisie est **inférieure à A min** ou **inférieure à la section calculée**, ou si les
  barres ne tiennent pas sur un lit dans bw.
- Les propositions de barres (≥ HA10) sont cliquables : elles remplissent le champ. La **flèche** est vérifiée avec
  le ferraillage choisi en travée.

## Note de calcul
Word **.docx** (python-docx, aucun logiciel requis) sur le modèle de « Note de calcul Flexion T » : hypothèses, section,
schéma statique, sollicitations ELU/ELS + diagrammes, analyse par cas (axe neutre, bras de levier, εb, εs, σs),
ferraillage choisi, cisaillement, flèche, conclusion. Export **PDF** également (menu Fichier / bouton).

## Structure
```
main.py
app/core/     models.py  beam.py (forfaitaire, Caquot)  section.py (flexion, barres)  fleche.py  engine.py   ← calcul pur, sans Qt
app/ui/       main_window.ui  main_window.py  views.py  style.py  icon.py
app/export/   report.py (note Word + PDF)
tests/        test_core.py
```

## Corrections par rapport au programme d'origine
Les résultats ELU de l'original ont été reproduits à l'identique avant toute correction. Anomalies corrigées :

| # | Original | Correction |
|---|----------|-----------|
| 1 | Forfaitaire **3 travées** : `IndexError` | appuis 0,5·max(M0 adjacents) |
| 2 | Tableau **ELS = valeurs ELU** (listes non réinitialisées) | ELU et ELS indépendants |
| 3 | Flèche : plantage si les 3 conditions sont vérifiées | cas géré |
| 4 | Flèche : `fig`, `fip`, `fvg` calculés avec Mj | chaque flèche avec son moment |
| 5 | Flèche : largeur 0,6 m codée en dur | entraxe bf saisi |
| 6 | j = plancher **+ cloisons** | j = poids du plancher seul |
| 7 | A min en travée avec bf | A min = 0,23·bw·d·ft28/fe |
| 8 | Aucun contrôle de μ ; `sqrt` négatif | contrôle μ ≤ μl |
| 9 | Table de compression ignorée | section rectangulaire bf ou en T |
| 10 | Travée unique : Mt = 0,75·M0 | formule générale BAEL |
| 11 | `eval()` sur les portées | analyse sécurisée |
| 12 | Aucun contrôle d'applicabilité | conditions affichées + avertissements ; choix libre de la méthode |

Limites : Caquot avec toutes les travées chargées (sans alternance) ; pas d'armatures transversales ; pas de
vérification des contraintes à l'ELS.
