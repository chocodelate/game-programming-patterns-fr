^title Flyweight
^section Design Patterns Revisited

[ob]The fog lifts, revealing a majestic old growth forest. Ancient hemlocks,
countless in number, tower over you forming a cathedral of greenery. The stained
glass canopy of leaves fragments the sunlight into golden shafts of mist.
Between giant trunks, you can make out the massive forest receding into the
distance.[oe]

La brume se dissipe, révélant une majestueuse forêt ancienne poussante. Des oruches anciennes, innombrables, se dressent au dessus de vous formant une cathédrale de verdure. La verrière de vitraux de feuilles fragmente la lumière du soleil dans des puits de brume d'or. Entre les troncs géants, vous pouvez distinguer le massif forestier s'estompant avec la distance.

[ob]This is the kind of otherworldly setting we dream of as game developers, and
scenes like these are often enabled by a pattern whose name couldn't possibly be
more modest: the humble Flyweight.[oe]

Ceci est le genre de configuration peu réaliste dont nous révons nous les développeurs de jeu, et des scènes comme celles ci sont souvent réalisées avec un patron dont le nom ne pourrait être plus modeste : l'humble Poids Léger.

[ob]## Forest for the Trees[oe]

## Une forêt pour des arbres

[ob]I can describe a sprawling woodland with just a few sentences, but actually
*implementing* it in a realtime game is another story. When you've got an entire
forest of individual trees filling the screen, all that a graphics programmer
sees is the millions of polygons they'll have to somehow shovel onto the GPU
every sixtieth of a second.[oe]

 Je peux décrire une vaste forêt avec seulement quelques phrases, mais l'*implémenter* dans un jeu temps réel est une autre histoire. Quand vous avez une forêt entière d'arbres individuels qui remplissent l'écran, tout ce qu'un programmeur graphique voit est les millions de polygones qu'il aura à enfourner d'une manière ou d'une autre dans le GPU tous les seizièmes de seconde.

[ob]We're talking thousands of trees, each with detailed geometry containing
thousands of polygons. Even if you have enough *memory* to describe that forest,
in order to render it, that data has to make its way over the bus from the CPU
to the GPU.[oe]

Nous parlons de milliers d'arbres, avec chacun une géométrie détaillée contenant des milliers de polygones. Même si vous avez assez de *mémoire* pour décrire la forêt, en vue d'en faire le rendu, cette donnée doit faire son chemin sur le bus du CPU vers le GPU.

[ob]Each tree has a bunch of bits associated with it:[oe]

Chaque arbre a un tas de bits qui lui sont associés :

* [ob] A mesh of polygons that define the shape of the trunk, branches, and greenery.[oe]
* [ob]Textures for the bark and leaves.[oe]
* [ob]Its location and orientation in the forest.[oe]
* [ob]Tuning parameters like size and tint so that each tree looks different.[oe]

* Un maillage de polygones qui définit la forme du tronc, des branches, et du feuillage.
* Des textures pour l'écorce et le feuilles.
* Sa position et son orientation dans la forêt.
* Des paramètres de réglages de la taille et de la teinte pour que chaque arbre soit différents.

[ob]If you were to sketch it out in code, you'd have something like this:[oe]

Si vous étiez en train d'esquisser en dehors du code, vous pourriez avoir ceci :

^code heavy-tree

[ob]That's a lot of data, and the mesh and textures are particularly large. An entire
forest of these objects is too much to throw at the GPU in one frame.
Fortunately, there's a time-honored trick to handling this.[oe]

C'est beaucoup de données, et le maillage et les textures sont particulièrement grand. Une forêt entière de ces objets est bien trop pour l'envoyer au GPU en une image. Heuresement, il y a une astuce éprouvée pour gérer cela.

[ob]The key observation is that even though there may be thousands of trees in the
forest, they mostly look similar. They will likely all use the <span
name="same">same</span> mesh and textures. That means most of the fields in
these objects are the *same* between all of those instances.[oe]

L'Observation clé est que même si il y avait des milliers d'arbres dans la forêt, ils sont presque semblables. Ils utiliseront presque tous le <span
name="same">même</span> maillage et les mêmes textures. Cela veut dire que la plupart des champs dans ces objets sont les *mêmes* parmi toutes ces instances.

<aside name="same">

[ob]You'd have to be crazy or a billionaire to budget for the artists to
individually model each tree in an entire forest.[oe]

Vous seriez fou ou milliardaire de bugéter pour que les artistes modélisent individuellement chaque arbre d'une forêt entière.

</aside>

<span name="trees"></span>

<img src="images/flyweight-trees.png" alt="A row of trees, each of which has its own Mesh, Bark, Leaves, Params, and Position." />

<aside name="trees">

[ob]Note that the stuff in the small boxes is the same for each tree.[oe]

Notez que la chose dans les petites boites est la même pour chaque arbre.

</aside>

[ob]We can model that explicitly by splitting the object in half. First, we pull
out the data that all trees have <span name="type">in common</span> and move it
into a separate class:[oe]

Nous pouvons modéliser ça explicitement en découpant l'objet en deux. D'abord, Nous extrayons les données que tous les arbres ont en commun et nous mettons ça dans un classe à part :

^code tree-model

[ob]The game only needs a single one of these, since there's no reason to have the
same meshes and textures in memory a thousand times. Then, each *instance* of a
tree in the world has a *reference* to that shared `TreeModel`. What remains in
`Tree` is the state that is instance-specific:[oe]

Le jeu a seulement besoin d'un seul examplaire de celles-ci, car il n'y a aucune raison d'avoir les mêmes maillages et textures en mémoire un millier de fois. Donc, chaque *instance* d'un arbre dans le monde a une référence vers ce `TreeModel` partagé. Ce qui reste dans le `Tree` est l'état qui est spécifique à l'instance :

^code split-tree

[ob]You can visualize it like this:[oe]

Vous pouvez visualiser ça comme ci :

<img src="images/flyweight-tree-model.png" alt="A row of trees each with its own Params and Position, but pointing to a shared Model with a Mesh, Bark, and Leaves." />

<aside name="type">

[ob]This looks a lot like the <a href="type-object.html" class="pattern">Type
Object</a> pattern. Both involve delegating part of an object's state to some
other object shared between a number of instances. However, the intent behind
the patterns differs.[oe]

Ceci ressemble beaucoup au patron  <a href="type-object.html" class="pattern">Objet de Type</a>. Cela implique à la fois de déléguer une part de l'état d'un objet à un autre objet partagé par plusieurs instances. Cependant, l'intention derrière le patron diffère.

[ob]With a type object, the goal is to minimize the number of classes you have to
define by lifting "types" into your own object model. Any memory sharing you get
from that is a bonus. The Flyweight pattern is purely about efficiency.[oe]

Avec un objet de type, le but est de minimiser le nombre de classes que vous aurez à définir en élevant des "types" dans votre propre modèle objet. Tout partage de mémoire que vous obtenez de cela est un bonus. Le patron Poids Léger est lui purement basé sur l'efficacité.

</aside>

[ob]This is all well and good for storing stuff in main memory, but that doesn't
help rendering. Before the forest gets on screen, it has to work its way over to
the GPU. We need to express this resource sharing in a way that the graphics
card understands.[oe]

Ceci est bien beau pour stocker des choses en mémoire principale, mais ça n'aide pas au rendu. Avant que la forêt arrive sur l'écran, elle doit faire son chemin vers le GPU. Nous avons besoin d'exprimer ce partage de ressource d'une manière que la carte graphique peut comprendre.

[ob]## A Thousand Instances
[oe]

## Un Millier d'Instances

[ob]To minimize the amount of data we have to push to the GPU, we want to be able to
send the shared data -- the `TreeModel` -- just *once*. Then, separately, we
push over every tree instance's unique data -- its position, color, and scale.
Finally, we tell the GPU, "Use that one model to render each of these
instances."[oe]

Pour minimiser le montant de données que nous avons à envoyer au GPU, nous voulons être capable d'envoyer les données partagées -- le `TreeModel` -- juste *une fois*. Puis, séparément, nous poussons chaque données uniques des instances d'arbre -- Sa position, couleur, et son échelle. Enfin, nous disons au GPU, " Utilise ce modèle pour rendre chacune de ces instances. "

[ob]Fortunately, today's graphics APIs and <span name="hardware">cards</span>
support exactly that. The details are fiddly and out of the scope of this book,
but both Direct3D and OpenGL can do something called [*instanced
rendering*](http://en.wikipedia.org/wiki/Geometry_instancing).[oe]

Heuresement, les APIs et les <span name="hardware">cartes</span> graphiques d'aujourd'hui supportent exactement cela. Les détails sont minutieux et hors du champs de ce livre, mais Direct3D et OpenGL peuvent faire quelque chose appelé l'[*instanciation géométrique*](http://en.wikipedia.org/wiki/Geometry_instancing).

[ob]In both APIs, you provide two streams of data. The first is the blob of common
data that will be rendered multiple times -- the mesh and textures in our
arboreal example. The second is the list of instances and their parameters that
will be used to vary that first chunk of data each time it's drawn. With a
single draw call, an entire forest grows.[oe]

Dans les deux APIs, vous fournissez deux flux de données. Le premier est le morceau de données communes qui sera rendu plusieurs fois -- le maillage et les textures dans notre exemple arboricole. Le second est la liste des instances et leurs paramètres qui seront utilisés pour faire varier le premier morceau de données chaque fois que c'est dessiné. Avec un simple appel de dessin, une forêt entière pousse.

<aside name="hardware">

[ob]The fact that this API is implemented directly by the graphics card means the
Flyweight pattern may be the only Gang of Four design pattern to have actual
hardware support.[oe]

Le fait que cette API est implantée directement par la carte graphique signifie que le patron Poids Léger pourrait être le seul patron de conception du Gang of Four à avoir un vrai support matériel. 

</aside>

[ob]## The Flyweight Pattern[oe]

## Le Patron Poids Léger

[ob]Now that we've got one concrete example under our belts, I can walk you through
the general pattern. Flyweight, like its name implies, comes into play when you
have objects that need to be more lightweight, generally because you have too
many of them.[oe]

Maintenant que nous avons vu un exemple concret, je peux passer au patron en général. Poids Léger, comme son nom l'indique, entre en jeu quand nous avons des objets qui ont besoin d'être plus léger, généralement par ce que vous en avez trop.

[ob]With instanced rendering, it's not so much that they take up too much memory as
it is they take too much *time* to push each separate tree over the bus to the
GPU, but the basic idea is the same.[oe]

Avec l'instanciation géométrique, ce n'est pas tant que ça prend trop de mémoire mais plutôt que ça prend trop de *temps* pour envoyer chaque arbre individuellement sur le bus vers le GPU, mais l'idée basique est la même.

[ob]The pattern solves that by separating out an object's data into two kinds. The
first kind of data is the stuff that's not specific to a single *instance* of
that object and can be shared across all of them. The Gang of Four calls this
the *intrinsic* state, but I like to think of it as the "context-free" stuff. In
the example here, this is the geometry and textures for the tree.[oe]

Le patron résoud cela en séparant les données d'un objet en deux sortes. La première sorte de données est celle qui n'est pas spécifique à une simple *instance* de cet objet et peut être partagée par tous les objets. Le Gang of Four appelle ceci l'état *intrinsèque*, mais j'aime à penser que c'est une chose " non-contextuelle  ". Dans l'exemple présent, c'est la géométrie et les textures pour l'arbre.

[ob]The rest of the data is the *extrinsic* state, the stuff that is unique to that
instance. In this case, that is each tree's position, scale, and color. Just
like in the chunk of sample code up there, this pattern saves memory by sharing
one copy of the intrinsic state across every place where an object appears.[oe]

Le reste des données est l'état extrinsèque, la chose qui est unique à cette instance. Dans ce cas, c'est chaque position, échelle, et couleur de l'arbre. Juste comme dans le morceau de code au-dessus, ce patron économise de la mémoire en partageant une copie de l'état intrinsèque à chaque endroit où un objet apparait.

[ob]From what we've seen so far, this seems like basic resource sharing,
hardly worth being called a pattern. That's partially because in this example
here, we could come up with a clear separate *identity* for the shared state:
the `TreeModel`.[oe]

A partir de ce qu'on a vu jusqu'ici, ceci semble un partage de ressources basique, qui n'a pas besoin d'être appelé un patron. Cela est partiellement vrai par ce que dans l'exemple présent, nous pouvions nous avancer avec une identité claire séparée pour l'état partagé : le `TreeModel`.

[ob]I find this pattern to be less obvious (and thus more clever) when used in cases
where there isn't a really well-defined identity for the shared object. In those
cases, it feels more like an object is magically in multiple places at the same
time. Let me show you another example.[oe]

Je trouve ce patron moins évident (et donc plus intélligent) quand il est utilisé dans des cas où il n'y a pas une identité bien définie pour l'objet partagé. Dans ces cas, il semble plus comme un objet qui est magiquement en de multiples endroits en même temps. Permettez moi de vous montrer un autre exemple.

[ob]## A Place To Put Down Roots[oe]

## Un endroit pour s'enraciner

[ob]The ground these trees are growing on needs to be represented in our game too.
There can be patches of grass, dirt, hills, lakes, rivers, and whatever other
terrain you can dream up. We'll make the ground *tile-based*: the surface of the
world is a huge grid of tiny tiles. Each tile is covered in one kind of terrain.[oe]

Le sol de ces arbres sont grandissant en besoin pour être représenté dans notre jeu également. Il peut y avoir des pièces de d'herbe, de terre, de collines, de lacs, de rivière, et n'importe quel terrain que tu peux imaginer. Nous rendrons le sol *basé-tuile* : la surface du monde est une immense grille de petites tuiles. Chaque tuile est couverte d'une sorte de terrain.

[ob]Each terrain type has a number of properties that affect gameplay:[oe]

Chaque type de terrain a un certain nombre de propriétés qui affectent le gameplay :

* [ob] A movement cost that determines how quickly players can move through it.[oe]
* [ob]A flag for whether it's a watery terrain that can be crossed by boats.[oe]
* [ob]A texture used to render it.[oe]

* Un coût de mouvement lequel détermine à quelle vitesse les joueurs peuvent se déplacer dessus.
* Un drapeau pour que si le terrain est de l'eau il puisse être traversé en bateau.
* Une texture utilisé pour le rendre.

[ob]Because we game programmers are paranoid about efficiency, there's no way we'd
store all of that state in <span name="learned">each</span> tile in the world.
Instead, a common approach is to use an enum for terrain types:[oe]

Par ce que nous programmeurs de jeu sommes paranoïaque au sujet de l'efficacité, d'aucune façon nous devrions stocker tout de cet état dans <span name="learned">chaque</span> tuile dans le monde. A la place, une approche commune est d'utiliser une énumération pour les types de terrain :

<aside name="learned">

[ob]After all, we already learned our lesson with those trees.[oe]

Après tout, on a déjà appris notre leçon avec ces arbres.

</aside>

^code terrain-enum

[ob]Then the world maintains a huge grid of those:[oe]

Puis le monde maintient une immense grille de ceux ci :

<span name="grid"></span>

^code enum-world

<aside name="grid">

[ob]Here I'm using a nested array to store the 2D grid. That's efficient in C/C++
because it will pack all of the elements together. In Java or other memory-
managed languages, doing that will actually give you an array of rows where each
element is a *reference* to the array of columns, which may not be as memory-
friendly as you'd like.[oe]

J'utilise ici un tableau imbriqué pour stocker la grille 2D. Cela est efficace en C/C++ car ça empaquètera tous les éléments ensemble. En Java ou dans d'autres langages à mémoire managée, faire la même chose vous donnera un tableau de lignes où chaque élément est une *référence* vers un tableau de colonnes, ce qui n'est pas très sympatique niveau mémoire comme vous l'imaginez.

[ob]In either case, real code would be better served by hiding this implementation
detail behind a nice 2D grid data structure. I'm doing this here just to keep it
simple.[oe]

Dans tous les cas, le vrai code devrait être mieux servi en cachant ce détail d'implémentation derrière une structure de donnée sympa de grille 2D. Je fais juste comme ça pour garder les choses simples.

</aside>

[ob]To actually get the useful data about a tile, we do something like:[oe]

Pour vraiment obtenir les données utiles sur une tuile, nous faisons quelque chose comme ceci : 

^code enum-data

[ob]You get the idea. This works, but I find it ugly. I think of movement cost and
wetness as *data* about a terrain, but here that's embedded in code. Worse, the
data for a single terrain type is smeared across a bunch of methods. It would be
really nice to keep all of that encapsulated together. After all, that's what
objects are designed for.[oe]

Vous avez compris l'idée. Cela fonctionne, mais je trouve ça terrible. Je pense au coût de mouvement et à l'humidité comme *donnée* du terrain, mais ici c'est embarqué dans le code. Pire, les données pour un simple type de terrain est barbouillé en un ensemble de méthodes. Cela pourrait être bien de garder tout ça encapsulé ensemble. Après tout, c'est ce pourquoi les objets sont faits.

[ob]It would be great if we could have an actual terrain *class*, like:[oe]

il serait bien d'avoir une vrai *classe* de terrain, comme ceci :

<span name="const"></span>

^code terrain-class

<aside name="const">

[ob]You'll notice that all of the methods here are `const`. That's no coincidence.
Since the same object is used in multiple contexts, if you were to modify it,
the changes would appear in multiple places simultaneously.[oe]

Vous noterez que toutes les méthodes ici sont `const`. Ceci n'est pas une coïncidence. Comme le même objet est utilisé dans de multiples contextes, si vous le modifiez, les changements devraient apparaître en de multiples endroits simultanément.

[ob]That's probably not what you want. Sharing objects to save memory should be an
optimization that doesn't affect the visible behavior of the app. Because of
this, Flyweight objects are almost always immutable.[oe]

Cela n'est probablement pas ce que vous voulez. Partager des objets pour économiser de la mémoire devrait être une optimisation qui n'affecte pas le comportement visible de l'application. Par ce fait, les objets Poids Léger sont presque toujours immuables.

</aside>

[ob]But we don't want to pay the cost of having an instance of that for each tile in
the world. If you look at that class, you'll notice that there's actually
*nothing* in there that's specific to *where* that tile is. In flyweight terms,
*all* of a terrain's state is "intrinsic" or "context-free".[oe]

Mais nous ne voulons pas payer le cout d'avoir une instance pour chaque tuile dans la monde. Si vous regardez bien cette classe, vous noterez qu'il n'y a véritablement *rien* la dedans qui est spécifique à *où* cette tuile se situe. En terme poids léger, *tout* d'un état du terrain est "intinsèque" ou "non-contextuelle".

[ob]Given that, there's no reason to have more than one of each terrain type. Every
grass tile on the ground is identical to every other one. Instead of having the
world be a grid of enums or Terrain objects, it will be a grid of *pointers* to
`Terrain` objects:[oe]

Ceci dit, Il n'y a aucune raison d'en avoir plus qu'une pour chaque type de terrain. Chaque tuile d'herbe sur le sol est identique aux autres du même type. Au lieu d'avoir le monde comme une grille d'énumération ou d'objets `Terrain`, il sera une grille de *pointeurs* vers des objets `Terrain` :

^code world-terrain-pointers

[ob]Each tile that uses the same terrain will point to the same terrain instance.[oe]

Chaque tuile qui utilise le même terrain pointera vers la même instance de terrain.

<img src="images/flyweight-tiles.png" alt="A row of tiles. Each tile points to either a shared Grass, River, or Hill object." />

[ob]Since the terrain instances are used in multiple places, their lifetimes would
be a little more complex to manage if you were to dynamically allocate them.
Instead, we'll just store them directly in the world:[oe]

Comme les instances de terrain sont utilisées en de multiples endroits, leur durée de vie devrait être un petit plus complexe à gérer que vous les allouiez dynamiquement. Plutôt, nous les stockerons directement dans le monde :

^code world-terrain

[ob]Then we can use those to paint the ground like this:[oe]

Puis nous pouvons les utiliser pour peindre le sol comme ceci :

<span name="generate"></span>

^code generate

<aside name="generate">

[ob]I'll admit this isn't the world's greatest procedural terrain generation
algorithm.[oe]

J'admet que ce n'est pas le meilleur algorithme de génération procédural de terrain du monde.

</aside>

[ob]Now instead of methods on `World` for accessing the terrain properties, we can
expose the `Terrain` object directly:[oe]

Maintenant au lieu des méthodes sur `World` pour accéder au propriétés du terrain, nous pouvons exposer l'objet `Terrain` directement :

^code get-tile

[ob]This way, `World` is no longer coupled to all sorts of details of terrains. If
you want some property of the tile, you can get it right from that object:[oe]

De cette manière, `World` n'est plus couplé à toutes sortes de détails du terrain. Si vous voulez une propriété d'une tuile, vous pouvez récupérer ça juste à partir de cet objet :

^code use-get-tile

[ob]We're back to the pleasant API of working with real objects, and we did this
with almost no overhead -- a pointer is often no larger than an enum.[oe]

Nous sommes revenus à la plaisante API qui travaille avec des vrais objets, et nous avons fait ceci avec presque aucun surcoût -- un pointeur n'est souvent pas plus grand qu'une énumération.

[ob]## What About Performance?[oe]

## Quid de la Performance ?

[ob]I say "almost" here because the performance bean counters will rightfully want
to know how this compares to using an enum. Referencing the terrain by pointer
implies an indirect lookup. To get to some terrain data like the movement cost,
you first have to follow the pointer in the grid to find the terrain object and
then find the movement cost there. Chasing a pointer like this can cause a <span
name="cache">cache miss</span>, which can slow things down.[oe]

Je dis "presque"  par ce que le comptable de la performance  voudra légitimement savoir comment ceci se compare à l'utilisation d'une énumération. Référencer le terrain par pointeur implique une recherche indirecte. Pour obtenir des données de terrain comme le coût de mouvement, vous devez d'abord suivre le pointeur dans la grille pour trouver l'objet terrain et alors trouver le coût de mouvement. Accéder à un pointeur comme celui-ci peut causer un <span
name="cache">cache-miss</span>, lequel peut ralentir les choses.

<aside name="cache">

[ob]For lots more on pointer chasing and cache misses, see the chapter on <a href
="data-locality.html" class="pattern">Data Locality</a>.[oe]

Pour en savoir plus sur la recherche de pointeur et les cache-miss, voyez le chapitre sur la <a href
="data-locality.html" class="pattern">Localité des Données</a>.

</aside>

[ob]As always, the golden rule of optimization is *profile first*. Modern computer
hardware is too complex for performance to be a game of pure reason anymore. In
my tests for this chapter, there was no penalty for using a flyweight over an
enum. Flyweights were actually noticeably faster. But that's entirely dependent
on how other stuff is laid out in memory.[oe]

Comme toujours, la règle d'or de l'optimisation est de *profiler en premier*. Le matériel d'ordinateur moderne est trop complexe pour que la performance soit un jeu de pure raison désormais. Dans mes tests pour ce chapitre, il n'y avait aucune pénalité d'utiliser un poids léger avec une énumération. Les poids légers étaient vraiment plus rapide. Mais ceci est entièrement dépendant de comment l'autre chose est agencée en mémoire.

[ob]What I *am* confident of is that using flyweight objects shouldn't be dismissed
out of hand. They give you the advantages of an object-oriented style without
the expense of tons of objects. If you find yourself creating an enum and doing
lots of switches on it, consider this pattern instead. If you're worried about
performance, at least profile first before changing your code to a less
maintainable style.[oe]

Ce en quoi je *suis* confiant est qu'utiliser des objets poids léger ne devrait pas être rejeté du revers de la main. Ils vous donnent les avantages d'un style orienté-objet sans la dépense de tonnes d'objets. Si vous trouvez vous même en train créer une énumération et faire plein de switch dessus, considerez plutôt ce patron. Si vous vous inquiétez au sujet de la performance, au moins profilez d'abord avant de changer votre code dans un style moins maintenable.

[ob]## See Also[oe]

## Voir aussi

* [ob]In the tile example, we just eagerly created an instance for each terrain
    type and stored it in `World`. That made it easy to find and reuse the
    shared instances. In many cases, though, you won't want to create *all* of
    the flyweights up front.[oe]

	[ob]If you can't predict which ones you actually need, it's better to create
    them on demand. To get the advantage of sharing, when you request one, you
    first see if you've already created an identical one. If so, you just return
    that instance.[oe]
	
	[ob]This usually means that you have to encapsulate construction behind some
    interface that can first look for an existing object. Hiding a constructor
    like this is an example of the <a
    href="http://en.wikipedia.org/wiki/Factory_method_pattern" class="gof-
    pattern">Factory Method</a> pattern.[oe]
	
*  	Dans l'exemple des tuiles, nous avons créé une instance pour chaque type de terrain et stocké ça dans `World`. Cela a rendu   		facile à trouver et réutiliser les instances partagées. Dans bien des cas, malgré tout, vous ne voudrez pas *tous* les poids léger d'avance.

	Si vous ne pouvez pas prédire laquelle vous avez vraiment besoin, c'est mieux de les créer sur demande. Pour obtenir l'avantage du partage, quand vous en demandez une, vous regardez d'abord vous  en avez déjà créé une identique. Si c'est le cas, vous retournez juste cette instance.

	Ceci veut dire qu'habituellement vous aurez à encapsuler la construction dérrière une interface qui peut d'abord rechercher un objet existant. Cacher un constructeur comme ceci est un exemple du patron <a
    href="http://en.wikipedia.org/wiki/Factory_method_pattern" class="gof-
    pattern">Méthode de Fabrique</a>.

* [ob]  In order to return a previously created flyweight, you'll have to keep track
    of the pool of ones that you've already instantiated. As the name implies,
    that means that an <a href="object-pool.html" class="pattern">Object
    Pool</a> might be a helpful place to store them.[oe]
	
* En vue de retourner un poids léger créer précédemment, vous aurez à garder une trace de la piscine qui contient ceux que vous avez déjà instanciés. Comme le nom l'indique, cela signifie qu'une <a href="object-pool.html" class="pattern">Piscine d'Objet</a> pourrait être un endroit utile pour les stocker.
	
* [ob]When you're using the <a class="pattern" href="state.html">State</a>
    pattern, you often have "state" objects that don't have any fields specific
    to the machine that the state is being used in. The state's
    identity and methods are enough to be useful. In that case, you can apply
    this pattern and reuse that same state instance in multiple state machines
    at the same time without any problems.[oe]

* Quand vous utilisez le patron <a class="pattern" href="state.html">Etat</a>, vous avez souvent des objets "état" qui n'ont pas de champs spécifique à la machine que l'état utilise. L'identité et les méthodes de l'état sont suffisants pour être utile. Dans ce cas, vous pouvez appliquer ce patron et réutiliser cette même instance d'état dans de multiples machines à états en même temps sans aucun problème.


