# Unification des noms d'auteurs
## Objectif du projet
L'objectif de ce projet était de résoudre deux problèmes rencontrés dans la création de la base de données BiblioBase, laquelle reprend les informations contenues dans les sections littéraires de la *Bibliographie de la France*. Dans cette base de données, les auteurs ne sont pas toujours évoqués avec les mêmes termes et leur nom peut ainsi être écrit avec des graphies différentes. De plus, ils sont indiqués comme anonyme lorsqu'ils étaient sous pseudonyme ou même anonymes au moment de la parution alors que nous connaissons parfois désormais qui ils sont. 
Le but est donc de désambiguïser chaque auteur au moyen d'un identifiant unique, non soumis aux variations de nom, et de réattribuer cette identifiant aussi lorsqu'ils ne sont pas indiqués comme auteur de leur ouvrage.

## Méthode employée
### Le choix de la technique
L'objectif de désambiguïsation aurait pu être réalisés de plusieurs manières différentes, chacune avec leurs avantages et leurs inconvénients. Il aurait ainsi été possibles de rapprocher les noms proches au moyen d'une distance de Levenshtein par exemple, ou de chercher directement leur nom sur la liste des auteurs d'un catalogue comme la BNF (avec le bémol que le catalogue des auteurs de la BNF nécessite un ordre particulier et a besoin de la forme exacte du nom). En revanche, l'objectif de réattribution demande obligatoirement l'appel à une base de données qui a déjà fait ce travail d'attribution des oeuvres à leurs auteurs en dépit des pseudonymes et de l'anonymat. C'est donc la méthode que nous avons choisie. La base de données à laquelle nous avons fait appel est la BNF. C'est en effet une référecne et c'est elle qui était déjà utilisée pour les données annotées par l'humain que nous avions pour évaluer les performances de notre algorithme.
Par simplicité, nous avons utilisé des modifications de l'adresse url qui permette à notre algorithme d'effectuer directement la recherche dans le catalogue, comme s'il s'agissait d'un usager.
Nous effections d'abord une recherche pour retrouver l'ouvrage tel quel. C'est ensuite à partir de lui, en extrayant les auteurs qui y sont associés, que nous pouvons récupérer des noms uniques pour les ambiguïtés, les pseudonymes ou les anonymats désormais dévoilés.

### Les éléments d'appel
Une difficulté a été de choisir sur quels éléments appuyer notre recherche. Les champs disponibles sur les données incluaient de nombreuses catégories et celles-ci n'étaient pas toute mises en valeur dans le catalogue de la BNF.
#### Le titre
De façon évidente, nous avons d'abord pensé à utiliser le titre. Cependant, nous nous sommes retrouvés face à une difficulté. Le titre indiqué ne correspond pas toujours exactement à celui donné par la BNF. 
Dans nos données, les éventuelles sous-titres sont inclus dans le titre alors que dans le catalogue de la BNF ils peuvent soit y être inclus, soit être placés dans une autre rubrique, soit être simplement absent. Face ce problème, nous avons divisé le titre en deux. Tout ce qui précède la première virgule, le premier point ou le premier point-virgule fait presque toujours partie du grand titre, le reste pas forcément. Ainsi notre algorithme recherche la première partie avec obligation dans la catégorie "titre" du catalogue, tandis qu'il cherche le reste sans obligation dans la totalité de la notice.
Néanmoins, certains titres sont précédés d'un sur-titre plus général qui l'englobe dans un ensemble et qui n'est générélament pas reporté sur la catalogue de la BNF. Nous avons analysé que cela concernait principalement trois sur-titres : "Répertoire du Théâtre-Français.", "Bibliothèque française." et "Classiques français.". Nous les avons donc simplement exclus du titre lors de la recherche (et avant la recherche du premier ,/./;). 
#### L'auteur
Etant donné que nous cherchions à identifier les auteurs, nous avons aussi directement pensé aux auteurs. Cela a été très efficace étant donné que la BNF réperorie les nom d'auteurs sous différentes formes lorsqu'il y en a et avec les pseudonymes lorsque certains sont utilisés. Cependant, cette catégorie a aussi été problématique.
De fait, les auteurs de la BiblioBase sont souvent accompagné d'un titre ou d'un rôle mais celui-ci est rarement présent dans la notice de la BNF. Il est cependant possible l'effacer de façon assez fréquente en enlevant ce qui est placé après la première virgule.
La BNF a aussi des difficultés vis-à-vis de l'emploi des prénoms, en particulier lorsqu'ils sont abbrégés. Pour pallier ce problème, nous n'avons donc gardé que les noms en sélectionnant le deuxième mot des auteurs. Nous avons consience que cela peut séparer en deux le nom d'un auteur dans le cas d'un nom composé. Cependant dans ce cas, la première partie du nom de l'auteur, avec le reste des données, pourrait suffir.
#### La date
L'utilisation de la date est généralement très efficace. BiblioBase nous informe le jour, mois et année de dépôt et la BNF indique l'année du publication. Ces deux données correspondent généralement assez bien mais certains textes peuvent avoir une année de différente entre les deux bases de données, s'il y a eu un délai entre dépôt et publication, c'est pourquoi nous impliquons les années suivante et précédente dans notre recherche.
Il peut aussi arriver que la date ne soit pas données dans la BiblioBase, dans ce cas nous cherchons sur le pannel large des périodes qu'elle couvre de 1818 à 1836.
#### L'imprimeur/Le libraire
Un des critère que nous avons jugé assez efficace pour distinguer le bon ouvrage est la publication. Celle-ci correspond parfois à l'imprimeur ou à la première librairie à l'avoir vendu, deux informations que nous avons dans la BiblioBase. Aussi nous laissons le choix entre les deux possibilités lors de la recherche. 
Cependant, parce que le prénom est souvent abbrégé ou absent dans l'une ou l'autre notice, notre algorithme ne sélectionne que le dernier mot des catégories imprimeur/libraire1, le nom de famille logiquement, suivi potentiellement de "fils", "aîné", "l'aîné" ou "jeune".
Cependant la publication peut parfois être encore autre, ou même n'être pas indiquée, nous autorisons donc une seconde recherche sans si la première n'a rien donné.

## Résumé des recherches de l'algorithme
Notre algorithme émet donc deux recherches parmi les textes imprimés : 
* la première recherche le titre entier, la date approximative, les auteurs par leur nom, l'imprimeur/libraire et à travers toute la notice les mots du sous-titre ;
* si la première est vide, la seconde répète la même recherche mais sans l'imprimeur/libraire.
À partir du résultat, les auteurs ou autres auteurs sont extraits du premier texte de la recherche qui est logiquement le plus probable.

## Résultats
### Résultats de la recherche d'ouvrage
Sur les 2107 ouvrages annotés par l'humain qui nous servi à vérifier les compétences de notre modèle, 232 étaient indiqués ne pas exister dans la BNF, notre recherche ne pouvait donc pas les trouver.
Sur les 1875 autres, 1304 recherches ont trouvé l'ouvrage voulu seul ou au sein d'une liste, 384 n'ont trouvé aucun ouvrage, 187 en ont trouvé d'autres. 
```
predict_ouvrage_verif
Prediction correcte    1304
Non trouvé              384
Hors BNF                232
Pas le même ouvrage     187
```

### Résultats sur la récupération des auteurs de la BNF
Sur la totalité tous les ouvrages pour laquelle n'a pas rien renvoyé ("Non trouvé", 1286 ont reçu les auteurs identifiés, 197 ont manqué d'au moins un de leurs auteurs, 31 des autres ont eu un de leurs auteurs placé en para-auteur et les autres ont reçus des auteurs en plus.
On remarque que les ouvrages correctement identfiés (la majorité d'entre eux) sont en grande partie corrects sur les auteurs qu'ils ont reçu, mais que les autres ont quand même des taux moyens.
```--- Sur tous les textes trouvés : 1723 textes
predict_auteur_verif
ok                             1286
Auteur ajouté                   209
Auteur absent                   197
Auteur placé en para-auteur      31
 
predict_paraauteur_verif
ok                    1370
Para-auteur ajouté     237
Para-auteur absent     116
 
--- Sur les textes correctement identifiés : 1304 textes
predict_auteur_verif
ok                             1068
Auteur ajouté                   133
Auteur absent                    80
Auteur placé en para-auteur      23
 
predict_paraauteur_verif
ok                    1056
Para-auteur ajouté     162
Para-auteur absent      86
 
--- Sur les textes hors BNF (potentiellement identifiés quand même) : 232 textes
predict_auteur_verif
ok                             108
Auteur absent                   66
Auteur ajouté                   57
Auteur placé en para-auteur      1
 
predict_paraauteur_verif
ok                    183
Para-auteur ajouté     33
Para-auteur absent     16
 
--- Sur les textes liés à un autre de la BNF que celui prévu : 187 textes
predict_auteur_verif
ok                             110
Auteur absent                   51
Auteur ajouté                   19
Auteur placé en para-auteur      7
 
predict_paraauteur_verif
ok                    131
Para-auteur ajouté     42
Para-auteur absent     14
```

## Analyse des résultats
### Analyse des résultats de la recherche d'ouvrage
L'analyse de la présence de textes qui ne sont pas celui attendu tend à montrer que ces textes sont généralement d'autres éditions, d'autres versions ou d'autres volumnes du même ouvrage. Cela est renforcé par la non-interopérabilité entre les nombres en lettres (quatrième), en chiffres arabes (4e) et en chiffres romains (IVe) sur la BNF. Ainsi, quand le "IVe volume" est recherché, le "3e volume" et le "4e volume" sont aussi proche l'un et de l'autre de ce qui est demandé. 

Lorsque le texte n'est pas du tout trouvé, cela est généralement pas l'absence d'un des champs dans la notice de la BNF ou d'une différence d'écriture (la BNF étant très sensible aux variations).
Ainsi, lorsque le date ou l'auteur ne sont pas évoqués, une recherche qui les utilise ne pourra pas aboutir. Il pourrait être possible de faire des recherches sans ces champs mais le risque d'erreur serait trop grand. La possibilité d'obtenir des erreurs, en particulier pour les ouvrages qui ne sont pas répertoriés dans le BNF, rendrait les gains minimes.
De la même façon, des variations de noms non-répertoriées suffisent à faire échouer la recherche. Il peut s'agit de variations d'orthographe comme l'auteur Combault/Gombault, ou le titre Thèmes français/Thèmes françois, ou alors de différence d'abbréviation entre les deux bases de données, entre S. A. R. et Son Altesse Royale, ou l'évocation ou non d'un titre d'une personne au sein du titre d'un ouvrage.

### Analyse des résultats de la récupération des auteurs
