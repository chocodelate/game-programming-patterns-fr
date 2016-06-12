^title Prototype
^section Design Patterns Revisited

[ob]The first time I heard the word "prototype" was in *Design Patterns*. Today, it
seems like everyone is saying it, but it turns out they aren't talking about the
<a href="http://en.wikipedia.org/wiki/Prototype_pattern"
class="gof-pattern">design pattern</a>. We'll cover that here, but I'll also
show you other, more interesting places where the term "prototype" and the
concepts behind it have popped up. But first, let's revisit the <span
name="original">original</span> pattern.[oe]

La premiére fois que j'ai entendu le mot "prototype" était dans *Design Patterns*. Aujourd'hui, il semble que chacun en parle, mais il se trouve qu'ils ne parlent pas du <a href="http://en.wikipedia.org/wiki/Prototype_pattern" class="gof-pattern">patron de conception</a>. Nous couvrirons cela ici, mais je vous montrerez aussi autre chose, de plus intéressant endroits où le terme " prototype " et les concepts derrière ça qui ont surgi. Mais d'abord, revisitons le patron <span
name="original">original</span>.

<aside name="original">

[ob]I don't say "original" lightly here. *Design Patterns* cites Ivan Sutherland's
legendary [Sketchpad](http://en.wikipedia.org/wiki/Sketchpad) project in *1963*
as one of the first examples of this pattern in the wild. While everyone else
was listening to Dylan and the Beatles, Sutherland was busy just, you know,
inventing the basic concepts of CAD, interactive graphics, and object-oriented
programming.[oe]

Je ne dis pas "original" à la légère ici. *Design Patterns* cite le légendaire projet [Sketchpad](http://en.wikipedia.org/wiki/Sketchpad) de Ivan Sutherland en *1963* comme un des premiers exempes de ce patron dans la nature. Pendant que chacun écoutait Dylan et les Beatles, Sutherland était juste occuper, bien, à inventer les concepts basiques de la CAO, des graphismes intéractifs, et de la programmation orientée-objet.

[ob]Watch [the demo](http://www.youtube.com/watch?v=USyoT_Ha_bA) and prepare to be
blown away.[oe]

Regardez [la démo](http://www.youtube.com/watch?v=USyoT_Ha_bA) et preparez vous à être époustouflé.

</aside>

## Le patron de conception Prototype

[ob]Pretend we're making a game in the style of Gauntlet. We've got creatures and
fiends swarming around the hero, vying for their share of his flesh. These
unsavory dinner companions enter the arena by way of "spawners", and there is a
different spawner for each kind of enemy.[oe]

Supposons que nous faisons un jeu dans le style de Gauntlet. Il y a des créatures et des démons grouillant autour du héros, en lice pour leur part de chair. Ces compagnons de table peu recommandables entrent dans l'arène par des " spawners ", et il y a un spawner différent pour chaque sorte d'ennemi.

[ob]For the sake of this example, let's say we have different classes for each kind
of monster in the game -- `Ghost`, `Demon`, `Sorcerer`, etc., like:[oe]

Pour le soucis de l'exemple, nous dirons que nous avons différentes classes pour chaque monstre dans le jeu -- `Ghost`, `Demon`, `Sorcerer`, etc., comme :

^code monster-classes

[ob]A spawner constructs instances of one particular monster type. To support every
monster in the game, we *could* brute-force it by having a spawner class for
each monster class, leading to a parallel class hierarchy:[oe]

Un spawner construit des instances d'un type de monstre particulier. Pour supporter chaque monstre dans le jeu, nous pourrions faire force-brute en ayant une classe spawner pour chaque classe de monstre, menant à la hiérarchie de classe parallèle :

<span name="inherits-arrow"></span>

<img src="images/prototype-hierarchies.png" alt="Parallel class hierarchies. Ghost, Demon, and Sorceror all inherit from Monster. GhostSpawner, DemonSpawner, and SorcerorSpawner inherit from Spawner." />

<aside name="inherits-arrow">

[ob]I had to dig up a dusty UML book to make this diagram. The <img
src="images/arrow-inherits.png" class="arrow" alt="A UML arrow." /> means "inherits from".[oe]

J'ai dû déterrer un livre UML poussiéreux pour faire ce diagramme. La <img
src="images/arrow-inherits.png" class="arrow" alt="A UML arrow." /> veut dire " hérite de ".

</aside>

[ob]Implementing it would look like this:[oe]

L'implémenter devrait ressembler à ceci :

^code spawner-classes

[ob]Unless you get paid by the line of code, this is obviously not a fun way
to hack this together. Lots of classes, lots of boilerplate, lots of redundancy,
lots of duplication, lots of repeating myself...[oe]

A moins que vous soyez payé à la ligne de code, ceci n'est évidemment pas une façon amusante d'organiser les choses. Beaucoup de classes, beaucoup de passe-partout, beaucoup de redondance, beaucoup de duplication, beaucoup de se répéter soi-même...

[ob]The Prototype pattern offers a solution. The key idea is that *an object can
spawn other objects similar to itself*. If you have one ghost, you can make more
ghosts from it. If you have a demon, you can make other demons. Any monster can
be treated as a *prototypal* monster used to generate other versions of
itself.[oe]

Le patron Prototype offre une solution. L'idée clé est qu'*un objet peut générer d'autres objets similaire à lui-même*. Si vous avez un fantôme, vous pouvez faire plus de fantôme à partir de lui. Si vous avez un démon, vous pouvez faire d'autre démons. Tout monstre peut être traité comme un monstre *prototypal* utilisé pour générer d'autre versions de lui-même.

[ob]To implement this, we give our base class, `Monster`, an abstract `clone()`
method:[oe]

Pour implémenter ceci, nous donnons à notre classe de base, `Monster`, une méthode abstraite `clone()` :

^code virtual-clone

[ob]Each monster subclass provides an implementation that returns a new object
identical in class and state to itself. For example:[oe]

Chaque sous-classe de monstre fournit une implémentation qui retourne a un nouvel objet de classe et état identique à lui-même. Par exemple :

^code clone-ghost

[ob]Once all our monsters support that, we no longer need a spawner class for each
monster class. Instead, we define a single one:[oe]

Une fois que tout nos monstres supporte cela, nous n'avons plus besoin d'une classe de spawner pour chaque monstre. Au lieu de ça, nous en définissons une seule : 

^code spawner-clone

[ob]It internally holds a monster, a hidden one whose sole purpose is to be used by
the spawner as a template to stamp out more monsters like it, sort of like a
queen bee who never leaves the hive.[oe]

Cela garde en interne un monstre, un caché pour lequel le but est d'être utilisé par le spawner comme un modèle pour faire plus de monstre comme celui-là, un genre de reine des abeilles qui ne quitte jamais la rûche.

<img src="images/prototype-spawner.png" alt="Un spawner contient un champ prototype référençant un Monster. Il appelle clone() sur le prototype pour créer de nouveaux monstres." />

[ob]To create a ghost spawner, we create a prototypal ghost instance and
then create a spawner holding that prototype:[oe]

Pour créer un spawner de fantôme, nous créons une instance de fantôme et puis un spawner gardant ce prototype :

^code spawn-ghost-clone

[ob]One neat part about this pattern is that it doesn't just clone the *class* of
the prototype, it clones its *state* too. This means we could make a spawner for
fast ghosts, weak ghosts, or slow ghosts just by creating an appropriate
prototype ghost.[oe]

Une partie nette sur ce patron est qu'il ne clone pas juste la *classe* du prototype, il clone son *état* également. Ceci veut dire que nous pourrions faire a spawner pour des fantômes rapides, des fantômes faibles, ou des fantômes lent juste en créant un prototype de fantôme approprié. 

[ob]I find something both elegant and yet surprising about this pattern. I can't
imagine coming up with it myself, but I can't imagine *not* knowing about it now
that I do.[oe]

Je trouve quelque chose d'élégant et encore surprenant sur ce patron. Je ne peux pas imaginer venir avec ça moi-même, mais je ne peux pas imaginer ne *pas* savoir ça maintenant que je connais.

### A quel point ça marche bien ?

[ob]Well, we don't have to create a separate spawner class for each monster, so
that's good. But we *do* have to implement `clone()` in each monster class.
That's just about as much code as the spawners.[oe]

En fait, nous n'avons pas à créer une classe spawner à part pour chaque monstre, donc ça c'est bien. Mais nous *devons*  implémenter `clone()` dans chaque classe de monstre. C'est juste autant de code que de spawners.

[ob]There are also some nasty semantic ratholes when you sit down to try to write a
correct `clone()`. Does it do a deep clone or shallow one? In other words, if a
demon is holding a pitchfork, does cloning the demon clone the pitchfork too?[oe]

Il y a aussi quelques méchantes loutres sémantiques quand vous essayez d'écrire une méthode `clone()`. Dois-je faire un clonage profond ou en surface ? En d'autres termes, si un démon tient une fourche, est ce que le clonage du démon clone la fourche aussi ?

[ob]Also, not only does this not look like it's saving us much code in this
contrived problem, there's the fact that it's a *contrived problem*. We had to
take as a given that we have separate classes for each monster. These days,
that's definitely *not* the way most game engines roll.[oe]

Aussi, de ne pas seulement faire ceci de ne pas ressembler nous évite du code en plus dans ce problème artificiel, il est un fait que c'est un *problème artificiel*. Nous avons pris pour acquis que nous avions des classes séparées pour chaque monstre. De nos jours, Ce n'est *pas* vraiment la manière dont la plupart des moteurs de jeu fonctionne.

[ob]Most of us learned the hard way that big class hierarchies like this are a pain
to manage, which is why we instead use patterns like <a href="component.html"
class="pattern">Component</a> and <a href="type-object.html"
class="pattern">Type Object</a> to model different kinds of entities without
enshrining each in its own class.[oe]

La plupart de nous a appris la dure façon dont les grosses hiérarchies de classe comme celle-ci sont une souffrance à gérer, c'est pourquoi nous utilisons des patrons comme  <a href="component.html" class="pattern">Composant</a> et <a href="type-object.html" class="pattern">Objet de Type</a> à la place pour modéliser différentes sortes d'entité sans consacrer sans propre classe à chacune.

### Fonctions de Spawn

[ob]Even if we do have different classes for each monster, there are other ways to
decorticate this *Felis catus*. Instead of making separate spawner *classes* for
each monster, we could make spawn *functions*, like so:[oe]

Même si nous avons différentes classes pour chaque monstre, il y a d'autres manières de décortiquer ce *Felis catus*. Au lieu de faire des *classes* spawner séparées pour chaque monstre, nous pourrions faire des fonctions de spawn, comme ceci :

^code callback

[ob]This is less boilerplate than rolling a whole class for constructing a monster
of some type. Then the one spawner class can simply store a function pointer:[oe]

Ceci est moins passe-partout que dérouler une classe entière pour construire un monstre d'un type donné. Puis la classe spawner peut simplement stocker un pointeur de fonction :

^code spawner-callback

[ob]To create a spawner for ghosts, you do:[oe]

Pour créer un spawner de fantômes, vous faites :

^code spawn-ghost-callback

### Modèles

[ob]By <span name="templates">now</span>, most C++ developers are familiar with
templates. Our spawner class needs to construct instances of some type, but we
don't want to hard code some specific monster class. The natural solution then
is to make it a *type parameter*, which templates let us do:[oe]

<span name="templates">Maintenant</span>, la plupart de développeurs C++ sont familiers des modèles. Notre classe spawner a besoin de construire des instances d'un certain type, mais nous ne voulons pas coder en dur des classes de monstres spécifiques. La solution naturelle alors est d'en faire un *paramètre de type*, lequel les modèles nous permettent de faire :

<aside name="templates">

Je ne suis pas sûr si les programmeurs C++ ont appris à les aimer ou si les modèles ont juste effrayé certains gens complètement. N'importe comment, chaque personne que je vois utiliser C++ aujourd'hui utilise les modèles aussi.

</aside>

<span name="base"></span>

^code templates

[ob]Using it looks like:[oe]

L'utiliser donne :

^code use-templates

<aside name="base">

La classe `Spawner` ici est telle que du code qui ne s'occupe pas de quel genre de monstre un spawner crée peut juste l'utiliser et fonctionner avec des pointeurs vers des `Monster`.

Si nous avions eu seulement la classe `SpawnerFor<T>`, il n'y aurait aucun supertype pour les instatiations partagent ce modèle, de sorte que tout code qui fonctionnerait avec des spawners de type de monstre devrait lui-même prendre un paramètre de modèle.

</aside>

### Type de Première Classe

[ob]The previous two solutions address the need to have a class, `Spawner`, which is
parameterized by a type. In C++, types aren't generally first-class, so that
requires some <span name="type-obj">gymnastics</span>. If you're using a
dynamically-typed language like JavaScript, Python, or Ruby where classes *are*
 regular objects you can pass around, you can solve this much more directly.[oe]

Les deux précédentes solutions résolvent le besoin d'avoir une classe, `Spawner`, qui soit paramètrée par un type. En C++, les types ne sont pas généralement de première classe, donc ça nécessite une certaine <span name="type-obj">gymnastique</span>. Si vous utilisez un langage dynamiquement typé comme JavaScript, Python, ou Ruby où les classes *sont* des objets réguliers que vous passez, vous pouvez résoudre de façon beaucoup plus directe.
 
<aside name="type-obj">

[ob]In some ways, the <a href="type-object.html" class="pattern">Type Object</a>
pattern is another workaround for the lack of first-class types. That pattern
can still be useful even in languages with them, though, because it lets *you*
define what a "type" is. You may want different semantics than what the
language's built-in classes provide.
[oe]

De certaines façons, le patron <a href="type-object.html" class="pattern">Objet de Type</a> est une autre astuce pour contourner le manque de types de première classe. Ce patron peut tout de même être utile pour des langages possédant ces derniers, malgré tout, car ça *vous* permet de définir ce qu'un "type" est. Vous pourriez vouloir différentes sémantiques que ce que les classes intégrées du langage fournissent.

</aside>

[ob]When you make a spawner, just pass in the class of monster that it should
construct -- the actual runtime object that represents the monster's
class. Easy as pie.[oe]

Quand vous fabriquez un spawner, passez juste dedans  la classe de monstre qu'il devrait construire -- le vrai objet au runtime qui représente la classe de monstre. Simple comme bonjour.

[ob]With all of these options, I honestly can't say I've found a case where I felt
the Prototype *design pattern* was the best answer. Maybe your experience will
be different, but for now let's put that away and talk about something else:
prototypes as a *language paradigm*.[oe]

Avec toutes ces options, Je ne peux pas dire honnêtement que j'ai trouvé un cas où le *patron de conception* Prototype était la meilleur solution. Peut être ton expérience sera différente, mais pour l'heure permettez moi de mettre ça de côté et de parle d'autre chose : les prototypes comme un *paradigme de langage*.

## Le paradigme de langage prototype

[ob]Many people think "object-oriented programming" is synonymous with "classes".
Definitions of OOP tend to feel like credos of opposing religious denominations,
but a fairly non-contentious take on it is that *OOP lets you define "objects"
which bundle data and code together.* Compared to structured languages like C
and functional languages like Scheme, the defining characteristic of OOP is that
it tightly binds state and behavior together.[oe]

Beaucoup de gens pensent que la "programmation orientée-objet" est synonyme de "classes". Les définitions de la POO ressemble un peu à des crédos de dénominations religieuse opposées, mais une simple qui ne cause aucune querelle est que *la POO vous permet de définir des "objets" qui enpaquettent  des données et du code ensemble*. Comparé à des langages structurés comme C et des langages fonctionnels comme Scheme, la définition caractéristique de la POO est que ça relie fermemement état et comportement ensemble.

[ob]You may think classes are the one and only way to do that, but a handful of guys
including Dave Ungar and Randall Smith beg to differ. They created a language in
the 80s called Self. While as OOP as can be, it has no classes.[oe]

Vous pourriez penser que les classes sont le seul moyen de le faire, mais une poignée de gars incluant Dave Ungar et Randall Smith supplient pour être différent. Ils ont créé un langage dans les années 80 appelé Self. Bien qu'il rend possible la POO, il n'a pas de classe.

### Self

[ob]In a pure sense, Self is *more* object-oriented than a class-based language. We
think of OOP as marrying state and behavior, but languages with classes actually
have a line of separation between them.
[oe]

Dans un esprit pur, Self est *plus* orienté-objet qu'un langage basé sur les classes. Nous pensons la POO comme un mariage entre état et comportement, mais les langages avec des classes ont tracé une ligne de séparation entre eux.

[ob]Consider the semantics of your favorite class-based language. To access some
state on an object, you look in the memory of the instance itself. State is
*contained* in the instance.[oe]

Considerez les sémantiques de votre langage de classe favoris. Pour accèder à l'état d'un objet, vous regardez dans la mémoire de l'instance elle-même. L'état est *contenu* dans l'instance.

[ob]To invoke a <span name="vtable">method</span>, though, you look up the
instance's class, and then you look up the method *there*. Behavior is contained
in the *class*. There's always that level of indirection to get to a method,
which means fields and methods are different.[oe]

Pour invoquer une <span name="vtable">méthode</span>, par contre, vous cherchez la classe de l'instance, et puis vous cherchez la méthode *là*. Le comportement est contenu dans la *classe*. Il y a toujours ce niveau d'indirection pour récupérer une méthode, ce qui veut dire que les champs et les méthodes sont différents.

<img src="images/prototype-class.png" alt="A Class contains a list of Methods. An Instance contains a list of Fields and a reference to its Class." />

<aside name="vtable">

[ob]For example, to invoke a virtual method in C++, you look in the instance for the
pointer to its vtable, then look up the method there.[oe]

Par exemple, pour invoquer une méthode virtuelle en C++, vous regardez dans l'instance pour trouver le pointeur vers sa vtable, puis cherchez la méthode là.

</aside>

[ob]Self eliminates that distinction. To look up *anything*, you just look on the
object. An instance can contain both state and behavior. You can have a single
object that has a method completely unique to it.[oe]

Self élimine cette distinction. Pour chercher *quelconque chose*, vous regardez juste l'objet. Une instance peut contenir l'état et le comportement à la fois. Vous pouvez avoir un simple objet qui a une méthode complètement propre à lui.

<span name="island"></span>

<img src="images/prototype-object.png" alt="An Object contains a mixed list of Fields and Methods." />

<aside name="island">

[ob]No man is an island, but this object is.[oe]

Aucun homme n'est une île, mais cette objet l'est.

</aside>

[ob]If that was all Self did, it would be hard to use. Inheritance in class-based
languages, despite its faults, gives you a useful mechanism for reusing
polymorphic code and avoiding duplication. To accomplish something similar
without classes, Self has *delegation*.[oe]

Si c'était tout ce que Self faisait, ça pourrait être dur à utiliser. L'héritage dans des langages basés sur les classes, en dépit ses torts, vous donne un mécanisme utile pour réutiliser du code polymorphique et éviter la duplication. Pour accomplir quelque chose de similaire sans classes, Self a la *délégation*.

[ob]To find a field or call a method on some object, we first look in the object
itself. If it has it, we're done. If it doesn't, we look at the object's <span
name="parent">*parent*</span>. This is just a reference to some other object.
When we fail to find a property on the first object, we try its parent, and its
parent, and so on. In other words, failed lookups are *delegated* to an object's
parent.[oe]

Pour trouver un champ ou appelez une méthode sur un objet, nous cherchons d'abord dans l'objet lui-même. Si il l'a, c'est bon. Si il ne l'a pas, nous cherchons dans l'objet <span name="parent">*parent*</span>. C'est juste une référence à un objet. Quand nous échouons à trouver un propriété sur le premier objet, nous essayons son parent, et son parent, et ensuite de suite. En d'autres termes, des recherches échouées sont déléguées au parent de l'objet.

<aside name="parent">

[ob]I'm simplifying here. Self actually supports multiple parents. Parents are just
specially marked fields, which means you can do things like inherit parents or
change them at runtime, leading to what's called *dynamic inheritance*.[oe]

Je simplifie ici. En fait Self supporte les parents multiples. Les parents sont juste marqués spécialement comme des champs, ce qui signifie que vous pouvez faire des choses comme hériter des parents ou les modifier au temps d'exécution, menant à ce qui est appelé l'*héritage dynamique*.

</aside>

<img src="images/prototype-delegate.png" alt="An Object contains Fields and Methods and a reference to another object that it delegates to." />

[ob]Parent objects let us reuse behavior (and state!) across multiple objects, so
we've covered part of the utility of classes. The other key thing classes do is
give us a way to create instances. When you need a new thingamabob, you can just
do `new Thingamabob()`, or whatever your preferred language's syntax is. A class
is a factory for instances of itself.[oe]

Les objets parents nous permettent de réutiliser le comportement (et l'état !) via de multiples objets, donc nous avons couvert la part utile des classes. L'autre chose clé que les classes font est de nous donner un moyen de créer des instances. Quand vous avez besoin d'un nouveau machin, vous pouvez toujours faire `new Machin()`, ou autre chose selon la syntaxe de votre langage préféré. Une classe est une fabrique pour des instances d'elle-même.

[ob]Without classes, how do we make new things? In particular, how do we make a
bunch of new things that all have stuff in common? Just like the design pattern,
the way you do this in Self is by *cloning*.[oe]

Sans classes, comme faisons nous de nouvelles choses ? En particulier, comment faisons nous un tas de choses qui ont toutes les même choses en commun ? Juste comme le patron de conception, la manière de faire de Self est le *clonage*.

[ob]In Self, it's as if *every* object supports the Prototype design pattern
automatically. Any object can be cloned. To make a bunch of similar objects, you:[oe]

Dans Self, c'est comme si *chaque* objet supportait le patron de conception Prototype automatiquement. Tout objet peut être cloné. Pour faire un tas d'objets équivalents, vous :

[ob]1 Beat one object into the shape you want. You can just clone the base `Object`
   built into the system and then stuff fields and methods into it.[oe]
   
[ob]2 Clone it to make as many... uh... clones as you want.[oe]
   
1. Taillez un objet à la forme que vous souhaitez. Vous pouvez cloner juste l'`Object` de base construit dans le système et puis mettre des champs et des méthodes dedans.
2. Clonez ça pour faire autant de... euh... clones que vous voulez.

[ob]This gives us the elegance of the Prototype design pattern without the tedium of
having to implement `clone()` ourselves; it's built into the system.[oe]

Ceci nous donne l'élégance du patron de conception Prototype sans l'ennui d'avoir à implémenter `clone()` nous-même; c'est intégré au système.

[ob]This is such a beautiful, clever, minimal system that as soon as I learned about
it, <span name="finch">I started</span> creating a prototype-based language to get more experience with it.[oe]

C'est un système tellement beau, intelligent, et minimal que dès que je l'ai appris, <span name="finch">j'ai commencé</span> à créer un langage basé sur les prototypes pour prendre un peu d'expérience là-dedans.

<aside name="finch">

Je réalise que construire un langage du début n'est pas la manière la plus efficace d'apprendre, mais que puis-je dire ? Je suis un peu bizarre. Si vous êtes curieux, le langage s'appelle [Finch](http://finch.stuffwithstuff.com/).

</aside>

### Comment ça s'est passé ?

[ob]I was super excited to play with a pure prototype-based language, but once I had
mine up and running, I <span name="no-fun">discovered</span> an unpleasant fact:
it just wasn't that fun to program in.[oe]

J'étais vraiment excité de jouer avec un pur langage basé sur les prototypes, mais une fois que le mien était en place et fonctionnait, j'ai <span name="no-fun">découvert</span> un fait déplaisant : ce n'était pas si amusant de programmer avec.

<aside name="no-fun">

J'ai depuis entendu dire que beaucoup de programmeurs Self en étaient venus à la même conclusion. Le projet était loin d'être une perte malgré tout. Self était si dynamique que ça nécessitait toutes sortes d'innovations de machines virtuelles en vue qu'il soit assez rapide à l'exécution.

[ob]The ideas they invented for just-in-time compilation, garbage collection, and
optimizing method dispatch are the exact same techniques -- often implemented by
the same people! -- that now make many of the world's dynamically-typed
languages fast enough to use for massively popular applications.[oe]

Les idées qu'ils ont inventé pour la compilation juste-à-temps, le ramassage de miettes, et l'optimisation des appels de méthode sont exactement les mêmes techniques -- souvent implémenté par les mêmes gens ! -- que celles de beaucoup de langages dynamiquement typés assez rapide pour utiliser pour des applications massivement populaires.

</aside>

[ob]Sure, the language was simple to implement, but that was because it punted the
complexity onto the user. As soon as I started trying to use it, I found myself
missing the structure that classes give. I ended up trying to recapitulate it at
the library level since the language didn't have it.[oe]

Bien sûr, le langage était simple à implémenter, mais c'était par ce que il renvoyait la complexité sur l'utilisateur. Dès que j'ai commencé à essayer de l'utiliser, J'ai trouvé absente la structure que les classes donnent. J'ai fini par essayer de la récapituler au niveau de la bibliothèque car le langage n'en avait pas.

[ob]Maybe this is because my prior experience is in class-based languages, so
my mind has been tainted by that paradigm. But my hunch is that most people just
like well-defined "kinds of things".[oe]

Peut-être ceci est par ce que mon expérience précédente est sur langages basés sur les classes, et donc mon esprit a été teinté par ce paradigme. Mais mon pressentiment est que la plupart des gens aime juste le "genre de choses" bien définies.

[ob]In addition to the runaway success of class-based languages, look at how many
games have explicit character classes and a precise roster of different sorts
of enemies, items, and skills, each neatly labeled. You don't see many games
where each monster is a unique snowflake, like "sort of halfway between a troll
and a goblin with a bit of snake mixed in".[oe]

En plus du succès grandissant des langages à classes, regardez combien de jeux ont des classes explicites de personnage et une liste précise des différentes sortes d'ennemis, d'items, et de compétences, chacune proprement étiquetées. Vous ne verrez pas beaucoup de jeux dans lesquels chaque monstre est un unique flocon de neige, comme "une sorte de mix entre un troll et un gobelin avec une pointe de serpent".

[ob]While prototypes are a really cool paradigm and one that I wish more people
knew about, I'm glad that most of us aren't actually programming using them
every day. <span name="telling">The code</span> I've seen that fully embraces
prototypes has a weird mushiness to it that I find hard to wrap my head around.[oe]

Bien que les prototypes soient un paradigme vraiment cool et que je souhaite qu'il soit plus connu, je suis content que le plupart d'entre nous qui ne programme pas vraiment l'utilise tous les jours. <span name="telling">Le code</span> que j'ai vu qui adopte entièrement des prototypes a un étrange tendresse envers ça que je trouve difficile à comprendre.

<aside name="telling">

[ob]It's also telling how *little* code there actually is written in a prototypal
style. I've looked.[oe]

C'est aussi raconter combien est *petit* le code quand il est écrit dans un style prototypal. Je l'ai vu.

</aside>

### Que dire de JavaScript ?

[ob]OK, if prototype-based languages are so unfriendly, how do I explain JavaScript?
Here's a language with prototypes used by millions of people every day. More
computers run JavaScript than any other language on Earth.[oe]

OK, si les langages à classes sont si inamicales, Comment expliquerais-je JavaScript ? Voici un langage avec des prototypes utilisé par des millions de personnes chaque jour. Plus d'ordinateurs font tourner JavaScript que n'importe quel autre langage sur Terre.

[ob]<span name="ten">Brendan Eich</span>, the creator of JavaScript, took
inspiration directly from Self, and many of JavaScript's semantics are
prototype-based. Each object can have an arbitrary set of properties, both
fields and "methods" (which are really just functions stored as fields). An
object can also have another object, called its "prototype", that it delegates
to if a field access fails.[oe]

<span name="ten">Brendan Eich</span>, le créateur de JavaScript, a tiré son inspiration directement de Self, et beaucoup de sémantiques de JavaScript sont basé sur les prototypes. Chaque objet peut avoir un ensemble arbitraire de propriétés, à la fois des champs et des "méthodes" (lesquels sont vraiment juste des fonctions stockées comme des champs). Un objet peut aussi avoir un autre objet, appelé son "prototype", auquel il délègue si un accès à un champs échoue.

<aside name="ten">

[ob]As a language designer, one appealing thing about prototypes is that they are
simpler to implement than classes. Eich took full advantage of this: the first
version of JavaScript was created in ten days.[oe]

Comme un concepteur de langage, une chose attirante au sujet des prototypes est que ils sont plus simple à implémenter que les classes. Eich a pris plein avantage de ceci : la première version de JavaScript fut écrite en dix jours.

</aside>

[ob]But, despite that, I believe that JavaScript in practice has more in common with
class-based languages than with prototypal ones. One hint that JavaScript has
taken steps away from Self is that the core operation in a prototype-based
language, *cloning*, is nowhere to be seen.[oe]

Mais, en dépit de ça, je crois que JavaScript  a en pratique plus de choses en commun avec les langages à classe que les langages à prototypes. Une nuance qui fait que JavaScript est loin de Self est que l'opération coeur dans un langage à prototype, cloner, n'est nulle part à la vue.

[ob]There is no method to clone an object in JavaScript. The closest it has is
`Object.create()`, which lets you create a new object that
delegates to an existing one. Even that wasn't added until ECMAScript 5,
fourteen years after JavaScript came out. Instead of cloning, let me walk you
through the typical way you define types and create objects in JavaScript. You
start with a *constructor function*:[oe]

Il n'y a aucune méthode pour cloner un objet en JavaScript. Le plus proche est `Object.create()`, lequel vous permet de créer un nouvel objet qui délègue à un existant. Bien que ça n'a pas été pas ajouté avant ECMAScript 5, quatorze ans après la naissance de JavaScript. Au lieu de cloner, permettez moi de parcourir la manière typique pour définir des types et créer des objets en JavaScript. Vous commencez avec une *fonction constructeur* :

    :::javascript
    function Weapon(range, damage) {
      this.range = range;
      this.damage = damage;
    }

[ob]This creates a new object and initializes its fields. You invoke it like:[oe]

Ceci crée un nouvel objet et initialise ses champs. Vous invoquez ça comme ceci :

    :::javascript
    var sword = new Weapon(10, 16);

[ob]The `new` here invokes the body of the `Weapon()` function with `this` bound to a
new empty object. The body adds a bunch of fields to it, then the now-filled-in
object is automatically returned.[oe]

Le `new` ici invoque le corps de la fonction `Weapon()` avec `this` limité à un nouvel objet vide. Le corps ajoute un tas de champs à ça, puis l'objet désormais rempli est automatiquement retourné.

[ob]The `new` also does one other thing for you. When it creates that blank object,
it wires it up to delegate to a prototype object. You can get to that object
directly using `Weapon.prototype`.[oe]

Le `new` fait aussi une autre chose pour vous. Quand il crée cet objet vide, il le relie au délégué vers un objet prototype. Vous pouvez obtenir cet objet directement en utilisant `Weapon.prototype`.

[ob]While state is added in the constructor body, to define *behavior*, you usually
add methods to the prototype object. Something like this:[oe]

Bien que l'état est ajouté dans le corps du constructeur, pour définir le *comportement*, vous ajoutez habituellement des méthodes à l'objet prototype. Quelque chose comme ceci :

    :::javascript
    Weapon.prototype.attack = function(target) {
      if (distanceTo(target) > this.range) {
        console.log("Out of range!");
      } else {
        target.health -= this.damage;
      }
    }

[ob]This adds an `attack` property to the weapon prototype whose value is a
function. Since every object returned by `new Weapon()` delegates to
`Weapon.prototype`, you can now call `sword.attack()` and it will call that
function. It looks a bit like this:[oe]

Ceci ajoute une propriété d'attaque au prototype pour lequel la valeur est une fonction. Comme chaque objet retourné par `new Weapon()` délègue à `Weapon.prototype`, vous pouvez appeler `sword.attack()` et il appelera cette fonction. Ca ressemble un peu à ceci :

<img src="images/prototype-weapon.png" alt="A Weapon object contains an attack() method and other methods. A Sword object contains fields and delegates to Weapon." />

[ob]Let's review:[oe]

Récapitulons tout ça :

* [ob]The way you create objects is by a "new" operation that you invoke using an
  object that represents the type -- the constructor function.[oe]

* La manière dont vous créez des objets est avec une opération " new " que vous invoquez en utilisant un objet qui représente le type -- la fonction constructeur.

* [ob]State is stored on the instance itself.[oe]

* l'état est stocké dans l'instance elle-même.

* [ob]Behavior goes through a level of indirection -- delegating to the prototype --
  and is stored on a separate object that represents the set of methods shared
  by all objects of a certain type.[oe]

* Le comportement se met en place via un niveau d'indirection -- délèguant au prototype -- et est stocké dans un objet séparé qui représente l'ensemble des méthodes partagées par tous les objets d'un certain type.

[ob]Call me crazy, but that sounds a lot like my description of classes earlier. You
*can* write prototype-style code in JavaScript (*sans* cloning), but the syntax
and idioms of the language encourage a class-based approach.[oe]

Traitez moi de fou mais ça semble identique a ma description des classes précédente. Vous *pouvez* écrire du code dans un style prototype en JavaScript (*sans* cloner), mais la syntaxe et les idiomes du langage encourage une approche basée sur les classes.

[ob]Personally, I think that's a <span name="good">good thing</span>. Like I said, I
find doubling down on prototypes makes code harder to work with, so I like that
JavaScript wraps the core semantics in something a little more classy.[oe]

Personellement, je pense que c'est une <span name="good">bonne chose</span>. Comme je l'ai dit, je trouve que doubler les prototypes rend le code plus difficile à manipuler, donc j'aime bien que JavaScript enveloppe la sémantique du coeur dans quelque chose de plus classeux.

## Les prototypes pour la modélisation de données

[ob]OK, I keep talking about things I *don't* like prototypes for, which is making
this chapter a real downer. I think of this book as more comedy than tragedy, so
let's close this out with an area where I *do* think prototypes, or more
specifically *delegation*, can be useful.[oe]

OK, je continue de parler des choses pour lesquelles je n'aime pas les prototypes, ce qui fait de ce chapitre un vrai tranquillisant. Je pense ce livre plus comme une comédie qu'une tragédie, donc terminons par un domaine dans lequel je *crois* que les prototypes, ou plus spécifiquement la délégation, peuvent être utiles.

[ob]If you were to count all the bytes in a game that are code compared to the ones
that are data, you'd see the fraction of data has been increasing steadily since
the dawn of programming. Early games procedurally generated almost everything so
they could fit on floppies and old game cartridges. In many games today, the
code is just an "engine" that drives the game, which is defined entirely in
data.[oe]

Si vous comptiez tous les octets dans un jeu qui sont du code comparés à ceux qui sont des données, vous devriez voir la fraction de données a augmentée lourdement depuis 
la naissance de la programmation. Les premiers jeux généraient procéduralement presque tout de telle sorte que ils pouvaient rentrer sur des disquettes et des vielles cartouches de jeu. Dans beaucoup de jeu d'aujourd'hui, le code est juste un " moteur " qui pilote le jeu, lequel est entièrement défini par des données.

[ob]That's great, but pushing piles of content into data files doesn't magically
solve the organizational challenges of a large project. If anything, it makes it
harder. The reason we use programming languages is because they have tools for
managing complexity.[oe]

C'est super, mais empiler du contenu dans des fichiers de données ne résout pas magiquement les challenges d'organisation d'un gros projet. Quoi qu'il en soit, ça le rend plus difficile. La raison pour laquelle nous utilisons des langages de programmation est par ce que ils ont des outils pour gérer la complexité.

[ob]Instead of copying and pasting a chunk of code in ten places, we move it into a
function that we can call by name. Instead of copying a method in a bunch of
classes, we can put it in a separate class that those classes inherit from or
mix in.[oe]

Au lieu de copier/coller un morceau de code en dix endroits, nous le déplaçons dans une fonction que nous pouvons appeler par son nom. Au lieu de copier une méthode dans un tas de classes, nous pouvons la mettre dans un classe à part de sorte que ces classes en héritent.

[ob]When your game's data reaches a certain size, you really start wanting similar
features. Data modeling is a deep subject that I can't hope to do justice here,
but I do want to throw out one feature for you to consider in your own games:
using prototypes and delegation for reusing data.[oe]

Quand vos données de jeu atteignent une certaine taille, vous commencez vraiment à vouloir des fonctionnalités similaires. La modélisation de données est un sujet vaste à laquelle je ne peux espérer rendre justice ici, mais je voudrais vous montrer une fonctionnalité à considerer pour vos propres jeux : utiliser des prototypes et la délégation pour la réutilisation de données.

[ob]Let's say we're defining the data model for the <span name="shameless">shameless
Gauntlet rip-off</span> I mentioned earlier. The game designers need to specify
the attributes for monsters and items in some kind of files.[oe]

Disons que nous définissons le modèle de données pour <span name="shameless">l'arnaque éhontée du Gauntlet</span> que j'ai mentionné précedemment. Les concepteurs de jeu ont besoin de spécifier les attributs des monstres et des items dans des fichiers.

<aside name="shameless">

[ob]I mean completely original title in no way inspired by any previously existing
top-down multi-player dungeon crawl arcade games. Please don't sue me.[oe]

Je voulais dire un titre complètement original d'aucune manière inspiré par des jeux arcades donjon et dragons multijoueurs préexistants. S'il vous plait ne m'intentez pas un procès.

</aside>

[ob]One common approach is to use JSON. Data entities are basically *maps*, or
*property bags*, or any of a dozen other terms because there's nothing
programmers like more than <span name="inventing">inventing</span> a new name
for something that already has one.[oe]

Une approche commune est d'utiliser JSON. Des entités de données sont basiquement des *maps*, ou des *sacs de propriétés*, ou des douzaines d'autres termes par ce qu'il n'y a rien que les programmeurs aiment plus <span name="inventing">qu'inventer</span> un nouveau nom pour quelque chose qui en a déjà un.

<aside name="inventing">

[ob]We've re-invented them so many times that Steve Yegge calls them ["The Universal
Design Pattern"](http://steve-yegge.blogspot.com/2008/10/universal-design-patter
n.html).[oe]

Nous les avons réinventé tant de fois qui Steve Yegge les appelle ["Le patron de conception universel"](http://steve-yegge.blogspot.com/2008/10/universal-design-patter
n.html).

</aside>

[ob]So a goblin in the game might be defined something like this:[oe]

Donc un gobelin dans le jeu pourrait être défini par quelque chose comme ceci :

    :::json
    {
      "name": "goblin grunt",
      "minHealth": 20,
      "maxHealth": 30,
      "resists": ["cold", "poison"],
      "weaknesses": ["fire", "light"]
    }

[ob]This is pretty straightforward and even the most text-averse designer can handle
that. So you throw in a couple of sibling branches on the Great Goblin Family
Tree:[oe]

C'est assez simple et même le designer rebuté par le texte peut gérer ça. Donc vous vous lancez dans une paire de branches frangines sur l'arbre généalogique des super gobelins :

    :::json
    {
      "name": "goblin wizard",
      "minHealth": 20,
      "maxHealth": 30,
      "resists": ["cold", "poison"],
      "weaknesses": ["fire", "light"],
      "spells": ["fire ball", "lightning bolt"]
    }

    {
      "name": "goblin archer",
      "minHealth": 20,
      "maxHealth": 30,
      "resists": ["cold", "poison"],
      "weaknesses": ["fire", "light"],
      "attacks": ["short bow"]
    }

[ob]Now, if this was code, our aesthetic sense would be tingling. There's a lot of
duplication between these entities, and well-trained programmers *hate* that. It
wastes space and takes more time to author. You have to read carefully to tell
if the data even *is* the same. It's a maintenance headache. If we decide to
make all of the goblins in the game stronger, we need to remember to update the
health of all three of them. Bad bad bad.[oe]

Maintenant, si c'était ce code, notre sens de l'esthétique devrait nous donner des frissons. Il y a beaucoup de duplication entre ces entités, et tout programmeur aguerri *déteste* ça. Ca gaspille des l'espace et prend plus de temps à être créé. Vous avez même lu attentivement pour dire que les données *sont* les mêmes. C'est un casse-tête de maintenance. Si nous décidons de rendre plus fort tous les gobelins dans le jeu, nous avons besoin de se rappeler de mettre à jour la vie des trois. Mauvais mauvais mauvais.

[ob]If this was code, we'd create an abstraction for a "goblin" and reuse that
across the three goblin types. But dumb JSON doesn't know anything about that.
So let's make it a bit smarter.[oe]

Si c'était notre code, nous créérions une abstraction pour un "gobelin" et réutiliserions cela pour les trois types de gobelin. Mais cet idiot de JSON ne sait rien de cela. Rendons le un peu plus intelligent.

[ob]We'll declare that if an object has a <span name="meta">`"prototype"`</span>
field, then that defines the name of another object that this one delegates to.
Any properties that don't exist on the first object fall back to being looked up
on the prototype.[oe]

Nous déclarerons que si un objet a un champs <span name="meta">`"prototype"`</span>, alors ça définit le nom d'un autre objet auquel il délègue. Toutes propriétés qui n'existe pas sur le premier objet sera à chercher dans le prototype.

<aside name="meta">

[ob]This makes the `"prototype"` a piece of *meta*data instead of data. Goblins have
warty green skin and yellow teeth. They don't have prototypes. Prototypes are a
property of the *data object representing the goblin*, and not the goblin
itself.[oe]

Ceci fait du `"prototype"` un morceau de *méta*données au lieu de données. Les gobelins ont une peau verruqueuse verte et des dents jaunes. Ils n'ont pas de prototypes. Les prototypes sont une propriété de "l'objet de données représentant le gobelin", et non le gobelin lui-même. 

</aside>

[ob]With that, we can simplify the JSON for our goblin horde:[oe]

Avec ça, nous pouvons simplifier le JSON de notre horde de gobelin :

    :::json
    {
      "name": "goblin grunt",
      "minHealth": 20,
      "maxHealth": 30,
      "resists": ["cold", "poison"],
      "weaknesses": ["fire", "light"]
    }

    {
      "name": "goblin wizard",
      "prototype": "goblin grunt",
      "spells": ["fire ball", "lightning bolt"]
    }

    {
      "name": "goblin archer",
      "prototype": "goblin grunt",
      "attacks": ["short bow"]
    }

[ob]Since the archer and wizard have the grunt as their prototype, we don't have to
repeat the health, resists, and weaknesses in each of them. The logic we've added
to our data model is super simple -- basic single delegation -- but we've
already gotten rid of a bunch of duplication.[oe]

Comme l'archer et le magicien ont le grogneur comme prototype, nous n'avons pas besoin de répéter la vie, les resistances, et la faiblesse dans chacun d'eux. La logique que nous avons ajouté à notre modèle de données est super simple -- une délégation simple et basique -- mais nous nous sommes déjà débarrassé d'un tas de duplications.

[ob]One interesting thing to note here is that we didn't set up a fourth "base
goblin" *abstract* prototype for the three concrete goblin types to delegate to.
Instead, we just picked one of the goblins who was the simplest and delegated to
it.[oe]

Une chose intéressante à noter ici est que nous n'avons pas  mis en place un quatrième abstraction de prototype " gobelin de base  " pour les trois types de gobelins concrets pour déléguer. Au lieu de ça, nous avons juste pris un des gobelins qui était le plus simple et nous avons délégué à lui.

[ob]That feels natural in a prototype-based system where any object can be used as a
clone to create new refined objects, and I think it's equally natural here too.
It's a particularly good fit for data in games where you often have one-off
special entities in the game world.[oe]

Ca semble naturel dans un système à prototype où chaque objet peut être utilisé comme un clone pour créer de nouveaux objets raffinés, et je pense que c'est également naturel ici aussi. C'est un ajustement particulièrement bon  pour des données dans des jeux où vous avez souvent des entités spéciales uniques dans le monde du jeu.

[ob]Think about bosses and unique items. These are often refinements of a more
common object in the game, and prototypal delegation is a good fit for defining
those. The magic Sword of Head-Detaching, which is really just a longsword with
some bonuses, can be expressed as that directly:[oe]

Pensez aux bosses et aux items uniques. Ceux-ci sont souvent des raffinements d'un objet plus commun, et la délégation de prototype est un bon ajustement pour les définir. Le sabre magique détacheur de tête, qui est vraiment juste un long mot avec un peu de bonus, peut être exprimé comme ceci directement :

    :::json
    {
      "name": "Sword of Head-Detaching",
      "prototype": "longsword",
      "damageBonus": "20"
    }

[ob]A little extra power in your game engine's data modeling system can make it
easier for designers to add lots of little variations to the armaments and
beasties populating your game world, and that richness is exactly what delights
players.[oe]

Un petit pouvoir extra dans le système de modélisation de données de votre moteur de jeu rend plus facile aux concepteurs l'ajout de beaucoup de petites variations aux armements et aux bestioles peuplant votre monde, et cette richesse est exactement ce dont se délectent les joueurs.