# TP guidé 1 - D'un script fragile à un pipeline testé

Noms : Hadil Bejaoui

## Étape 0

### Q1
On écrit les bibliothèques dans requirements.txt pour que l'environnement soit
reproductible : un coéquipier, un serveur ou la CI installe les mêmes
bibliothèques avec une seule commande (pip install -r requirements.txt). Un
pip install pandas tapé à la main n'est écrit nulle part, donc chacun a des
versions différentes et le code peut marcher chez l'un et casser chez l'autre.

## Étape 1

### Q2
Le rapport affiche 8 lignes au lieu de 5 pays. groupby traite "tunisia",
"MOROCCO" et " Tunisia" (avec un espace devant) comme des valeurs différentes
de "Tunisia" et "Morocco", car il compare les chaînes exactement. Tunisia
apparaît 3 fois et Morocco 2 fois.

### Q3
| order_id | Problème | Dimension |
|---|---|---|
| 1008 | customer_id vide | Complétude |
| 1010 | la ligne apparaît deux fois | Unicité |
| 1004 | quantité = -2 | Validité |
| 1013 | date 2026-13-45 impossible | Validité |
| 1006, 1016, 1023 | "tunisia", "MOROCCO", " Tunisia" écrits différemment | Cohérence |
| 1019 | clavier à 4500 au lieu de 45 | Exactitude |

### Q4
Le total est trop haut : le total du script (12771.10) est supérieur au total
corrigé (7655.90, voir étape 3). Le prix aberrant de 4500 (commande 1019) et
le doublon 1010 le gonflent, alors que la quantité négative (1004) ne le
baisse que de 39,80. On ne pouvait pas le savoir sans recalculer, car les
erreurs agissent dans les deux sens : rien dans le résultat ne signale
laquelle domine.

## Étape 2

### Q5
Deux avantages du découpage en fonctions :
1. Chaque étape peut être testée seule avec un petit tableau (tests
   unitaires, étape 4), ce qui est impossible dans un script monolithique.
2. On peut modifier ou ajouter une règle sans toucher au reste, et quand
   quelque chose casse on sait tout de suite dans quelle étape.

## Étape 3

### Q6
Le bon total est 7655.90. L'ancien script disait 12771.10, soit 5115.20 de
trop : une erreur de +66,8 % par rapport au vrai total.

### Q7
La commande 1019 (clavier à 4500 au lieu de 45) a causé l'essentiel de
l'erreur, environ 4500 des 5115 de trop. Elle a été attrapée par la règle 4
(invalid_unit_price, prix supérieur à MAX_UNIT_PRICE = 1000).

### Q8
Enregistrer les lignes rejetées dans un fichier de quarantaine permet de
comprendre d'où vient le problème, de le signaler à la source, de corriger
les lignes et de les retraiter, et de garder une trace pour l'audit. Les
supprimer en silence cacherait le problème et on ne saurait plus rien sur les
données perdues.

## Étape 4

### Q9
Quand j'ai remplacé > 0 par > -5 dans la règle 3, le test
test_negative_quantity_is_rejected a échoué : la commande 1 (quantité -2) a
été gardée dans clean au lieu d'être rejetée (assert [1, 2] == [2]). Les
autres tests passaient toujours. Après avoir remis > 0, les 5 tests passent.

### Q10
Un test unitaire vérifie que ma logique de code est juste, sur un petit
tableau écrit à la main dont je connais la bonne réponse. Un contrôle de
données vérifie que les vraies données de cette exécution sont acceptables,
et il tourne à chaque exécution du pipeline.