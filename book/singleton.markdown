^title Singleton
^section Design Patterns Revisited

[ob]This chapter is an anomaly. Every other chapter in this book shows
you how to use a design pattern. This chapter shows you how *not* to use
one.[oe]

Ce chapitre est une anomalie. Chacun des autres chapitres vous montrent comment utiliser un patron de conception. Ce chapitre vous montre comment ne *pas* en utiliser un.

[ob]Despite noble intentions, the <a class="gof-pattern"
href="http://c2.com/cgi/wiki?SingletonPattern">Singleton</a> pattern described
by the Gang of Four usually does more harm than good. They stress that the
pattern should be used sparingly, but that message was often lost in translation
to the <span name="instance">game industry</span>.[oe]

En dépit de nobles intentions, le patron <a class="gof-pattern"
href="http://c2.com/cgi/wiki?SingletonPattern">Singleton</a> décrit par le Gang of Four fait habituellement plus de mal que de bien. Ils insistent sur le fait que le patron devrait être utilisé avec parcimonie, mais ce message était souvent perdu en route vers <span name="instance">l'industrie du jeu</span>.

[ob]Like any pattern, using Singleton where it doesn't belong is about as helpful as
treating a bullet wound with a splint. Since it's so overused, most of this
chapter will be about *avoiding* singletons, but first, let's go over the
pattern itself.[oe]

Comme tout patron, utiliser Singleton où il n'est pas adéquate est aussi utile que traiter une blessure de balle avec une attelle. Comme c'est surutilisé, la plupart de ce chapitre sera sur *éviter* les singletons, mais d'abord, regardons le patron lui-même.

<aside name="instance">

[ob]When much of the industry moved to object-oriented programming from C, one
problem they ran into was "how do I get an instance?" They had some method they
wanted to call but didn't have an instance of the object that provides that
method in hand. Singletons (in other words, making it global) were an easy way
out.
[oe]

Quand la grande majorité de l'industrie est passée de C à la programmation orientée objet, un problème qu'ils ont eu était "Comment j'obtiens une instance ?" Ils avaient une méthode qu'ils voulaient appeler mais n'avait pas une instance de l'objet qui fournissait cette méthode. Les singletons (en d'autres termes, en rendant ça global) étaient une solution facile.

</aside>

## Le Patron Singleton

[ob]*Design Patterns* summarizes Singleton like this:[oe]

*Design Patterns* résume le Singleton comme ceci :

> [ob] Ensure a class has one instance, and provide a global point of access to it.[oe]

> Assurer qu'une classe a une instance, et fournit un point d'accès global à elle.

[ob]We'll split that at "and" and consider each half separately.[oe]

Nous découperons ça au "et" et considèrerons chaque moitié séparément.

### Restraindre une classe à une instance

[ob]There are times when a class cannot perform correctly if there is more than one
instance of it. The common case is when the class interacts with an external
system that maintains its own global state.
[oe]

Il y a des  moments où une classe ne peut pas fonctionner correctement si il y a plus qu'une instance d'elle. Le cas commun est quand la classe interagit avec un système externe qui maintient son propre état global.

[ob]Consider a class that wraps an underlying file system API. Because file
operations can take a while to complete, our class performs operations
asynchronously. This means multiple operations can be running concurrently, so
they must be coordinated with each other. If we start one call to create a file
and another one to delete that same file, our wrapper needs to be aware of both
to make sure they don't interfere with each other.[oe]

Considerez une classe qui enveloppe une API de système de fichier sous-jacente. Par ce que les opérations sur les fichiers peut prendre un moment pour se terminer, notre classe effectue des opérations de manière asynchrone. Ceci veut dire que de multiple opérations peuvent être lancées parallèlement, donc elle doivent être coordonnées avec chacune des autres. Si nous commençons un appel pour créer un fichier et un autre pour supprimer ce même fichier, notre enveloppe a besoin d'être consciente des deux choses en même temps pour être sûre qu'elles n'interfèrent pas entre elles.

[ob]To do this, a call into our wrapper needs to have access to every previous
operation. If users could freely create instances of our class, one instance
would have no way of knowing about operations that other instances started.
Enter the singleton. It provides a way for a class to ensure at compile time
that there is only a single instance of the class.[oe]

Pour faire ceci, un appel dans notre wrapper a besoin d'avoir accès à toutes les opérations précédentes. Si des utilisateurs pouvaient librement créer des instances de notre classe, une instance n'aurait aucun moyen de savoir quelles opérations les autres instances ont commencé. Ici rentre en jeu Singleton. Il fournit un moyen pour une classe d'assurer à la compilation que il y a seulement une unique instance de cette classe. 

### Fournir un point d'accès global

[ob]Several different systems in the game will use our file system wrapper: logging,
content loading, game state saving, etc. If those systems can't create their own
instances of our file system wrapper, how can they get ahold of one?[oe]

Plusieurs systèmes différents dans le jeu utiliseront notre système de fichier : journalisation, chargement de contenu, sauvegarde de l'état du jeu, etc. Si ces systèmes ne peuvent pas créer leur propre instances de notre wrapper de système de fichier, comment peuvent ils mettre la main dessus ?

[ob]Singleton provides a solution to this too. In addition to creating the single
instance, it also provides a globally available method to get it. This way,
anyone anywhere can get their paws on our blessed instance. All together, the
classic implementation looks like this:[oe]

Singleton fournit une solution a ceci aussi. En plus de créer l'unique instance, ça fournit aussi une méthode globalement disponible pour la récupérer. De cette façon, quiconque depuis n'importe quel endroit peut mettre la main sur notre instance bénie. Tout ensemble, l'implémentation classique ressemble à ceci : 

^code 1

[ob]The static `instance_` member holds an instance of the class, and the private
constructor ensures that it is the *only* one. The public static `instance()`
method grants access to the instance from anywhere in the codebase. It is also
responsible for instantiating the singleton instance lazily the first time
someone asks for it.[oe]

le membre statique `instance_` garde une instance de la classe, et le constructeur privé s'assure que c'est *l'unique*. La méthode publique statique `instance()` donne accès à l'instance depuis n'importe où dans le codebase. Elle est aussi responsable de l'instantiation paresseuse de l'instance du singleton la première fois que quelqu'un la demande.

[ob]A modern take looks like this:[oe]

Une recette moderne ressemble à ceci :

^code local-static

[ob]C++11 <span name="thread">mandates</span> that the initializer for a local
static variable is only run once, even in the presence of concurrency. So,
assuming you've got a modern C++ compiler, this code is thread-safe where the
first example is not.[oe]

C++11 <span name="thread">mandate</span> que l'initialiseur pour une variable locale statique est seulement éxecuté une fois, même en présence de concurrence. Donc, en supposant que vous avez un compilateur C++ moderne, ce code est thread-safe comparé au premier exemple qu'il ne l'est pas.

<aside name="thread">

[ob]Of course, the thread-safety of your singleton class itself is an entirely
different question! This just ensures that its *initialization* is.[oe]

Bien sûr, la sureté-thread de votre classe singleton elle-même est une tout autre question ! Ceci assure juste que son initialisation l'est.

</aside>

## Pourquoi nous l'utilisons

[ob]It seems we have a winner. Our file system wrapper is available wherever we need
it without the tedium of passing it around everywhere. The class itself cleverly
ensures we won't make a mess of things by instantiating a couple of instances.
It's got some other nice features too:[oe]

Il semble que nous avons un gagnant. notre enveloppe de système de fichier est disponible partout où nous en avons besoin sans le fastidieux d'avoir à le passer partout. La classe elle-même assure intelligemment que nous ne mettrons pas le bazard en instantiant plusieurs instances. Il a quelques autres fonctionnalités aussi :

* [ob]**It doesn't create the instance if no one uses it.** Saving memory and CPU
    cycles is always good. Since the singleton is initialized only when it's
    first accessed, it won't be instantiated at all if the game never asks for
    it.[oe]

*   **Il ne crée pas d'instance si personne ne l'utilise.** Economiser de la mémoire et du CPU est toujours bon. Comme le singleton est initialisé seulement quand il est accèdé pour la première fois, il ne sera pas jamais instancié si le jeu ne s'en sert jamais.

* [ob]**It's initialized at runtime.** A common alternative to Singleton is a
    class with static member variables. I like simple solutions, so I use static
    classes instead of singletons when possible, but there's one limitation
    static members have: automatic initialization. The compiler initializes
    statics before `main()` is called. This means they can't use information
    known only once the program is up and running (for example, configuration
    loaded from a file). It also means they can't reliably depend on each other
    -- the compiler does not guarantee the order in which statics are
    initialized relative to each other.[oe]
	
	[ob]Lazy initialization solves both of those problems. The singleton will be
    initialized as late as possible, so by that time any information it needs
    should be available. As long as they don't have circular dependencies, one
    singleton can even refer to another when initializing itself.[oe]

*   **C'est initialisé au temps d'exécution.** Une alternative commune est une classe avec des variables membres statiques. J'aime les solutions simples, donc j'utilise des classes statiques au lieu des singletons quand c'est possible, mais il y a une limitation que les membres statiques ont : l'initialisation automatique. Le compilateur initialise les statiques avant que `main()` soit appelée. Ceci signifie qu'il ne peuvent pas utiliser des informations connue seulement une fois que le programme est lancé et exécuté (par exemple, une configuration chargée à partir d'un fichier). Ca veut aussi dire que qu'ils ne peuvent dépendre l'un de l'autre de manière fiable -- le compilateur ne garantie pas l'ordre dans le lequel les statiques sont initialisés des uns par rapport aux autres.

    L'initialisation paresseuse résout les deux problèmes à la fois. Le singleton sera initialisé aussi tard que possible, puis à ce moment là n'importe quelle information dont il aurait besoin devrait être disponible.Du moment qu'ils n'ont pas de dépendances circulaires, un singleton peut même en invoquer un autre quand il est en train de s'initialiser lui-même.

* [ob]**You can subclass the singleton.** This is a powerful but often overlooked
    capability. Let's say we need our file system wrapper to be cross-platform.
    To make this work, we want it to be an abstract interface for a file system
    with subclasses that implement the interface for each platform. Here is the
    base class:[oe]

*   **Vous pouvez sous-classer le singleton.** Ceci est une capacité puissante mais souvent négligé. Disons que nous avons besoin que notre enveloppe de système de fichier soit multi-plateforme. Pour faire en sorte que ça marche, nous voulons qu'elle soit une interface abstraite pour un système de fichier avec des sous-classes qui implémentent l'interface pour chaque plateforme. Voici la classe de base :

    ^code 2

    [ob]Then we define derived classes for a couple of platforms:[oe]

    Puis nous définissons des classes dérivées pour plusieurs plateformes :

    ^code derived-file-systems

    [ob]Next, we turn `FileSystem` into a singleton:[oe]

    Ensuite, nous passons `FileSystem* en un singleton :

    ^code 3

    [ob]The clever part is how the instance is created:[oe]

    La partie intelligente est comment l'instance est créée :

    ^code 4

    [ob]With a simple compiler switch, we bind our file system wrapper to the
    appropriate concrete type. Our entire codebase can access the file system
    using `FileSystem::instance()` without being coupled to any
    platform-specific code. That coupling is instead encapsulated within the
    implementation file for the `FileSystem` class itself.[oe]

    Avec un simple switch de compilateur, nous lions notre enveloppe de système de fichier au type concret approprié. Notre codebase entier  peut acceder au système de fichier en utilisant `FileSystem::instance()` sans être couplé à aucun code  spécifique à la plateforme. Ce couplage est encapsulé à la place à l'intérieur du fichier d'implémentation pour la classe `FileSystem` elle-même.

[ob]This takes us about as far as most of us go when it comes to solving a problem
like this. We've got a file system wrapper. It works reliably. It's available
globally so every place that needs it can get to it. It's time to check in the
code and celebrate with a tasty beverage.[oe]

Ceci nous amène aussi loin que la plupart d'entre nous va quand on doit résoudre un problème comme celui-ci. Nous avons une enveloppe de système de fichier. Ca fonctionne de façon fiable. C'est accessible globalement pour qu'en chaque endroit où c'est nécessaire on puisse l'obtenir. Il est temps de vérifier dans le code et de le célébrer avec une boisson savoureuse.

## Pourquoi nous regrettons de l'utiliser

[ob]In the short term, the Singleton pattern is relatively benign. Like many design
choices, we pay the cost in the long term. Once we've cast a few unnecessary
singletons into cold hard code, here's the trouble we've bought ourselves:[oe]

Pour faire court, le patron Singleton est relativement bénin. Comme beaucoup de choix conceptuels, nous payons le prix au long terme. Une fois que nous avons fondu quelques singletons inutiles dans du code dur ancien, voici le trouble que nous nous sommes achetés :

### C'est une variable globale

[ob]When games were still written by a couple of guys in a garage, pushing the
hardware was more important than ivory-tower software engineering principles.
Old-school C and assembly coders used globals and statics without any trouble
and shipped good games. As games got bigger and more complex, architecture and
maintainability started to become the bottleneck. We struggled to ship games not
because of hardware limitations, but because of *productivity* limitations.[oe]

Quand les jeux étaient créés par un poignée de gars dans un garage, pousser le matériel était plus important que les principes d'ingénieurie logicielle de tour d'ivoire. Les codeurs de C old-school et d'assembleur utilisaient des globales et des statiques sans gène et vendaient de bons jeux. Comme les jeux sont devenus plus gros et plus complexe, l'architecture et la maintenabilité ont commencé à devenir le goulet d'étranglement. Nous luttions pour sortir des jeux non à cause des limitations du matériel, mais à cause des limitations de *productivité*.

[ob]So we moved to languages like C++ and started applying some of the hard-earned
wisdom of our software engineer forebears. One lesson we learned is that global
variables are bad for a variety of reasons:[oe]

Donc nous avons migré vers des langages comme C++ et commencé en appliquant quelques uns des sagesses durement acquises de nos ancêtres ingénieurs logiciel. Une leçon que nous avons apprise est que les variables globales sont mauvaises pour plusieurs raisons :

* [ob]**They make it harder to reason about code.** Say we're tracking down a bug
    in a function someone else wrote. If that function doesn't touch any global
    state, we can wrap our heads around it just by understanding the body of the
    <span name="pure">function</span> and the arguments being passed to it.[oe]

*   **Elles font qu'il est plus difficile de raisonner sur le code.** Disons que nous sommes en train de chercher un bug dans une fonction que quelqu'un a écrit. Si cette fonction ne touche aucune globale, nous pouvons essayer de comprendre juste en comprenant le corps de la <span name="pure">fonction</span> et les arguments qui lui sont passés.

    <aside name="pure">

    [ob]Computer scientists call functions that don't access or modify global state
    "pure" functions. Pure functions are easier to reason about, easier for the
    compiler to optimize, and let you do neat things like memoization where you
    cache and reuse the results from previous calls to the function.[oe]

    Les informaticiens appellent fonctions "pures" les fonctions celles n'accèdent pas ou ne modifient pas d'état globale. Les fonction pures permettent un raisonnement plus facile, sont plus facilement optimisées par le compilateur, et vous permez de faire des choses propres comme la mémoization dans laquelle vous cachez et réutilisez les résultats d'appels précédents à la fonction.

    [ob]While there are challenges to using purity exclusively, the benefits are
    enticing enough that computer scientists have created languages like Haskell
    that *only* allow pure functions.[oe]

	Tandis qu'il y a des challenges à utiliser exclusivement la pureté, les bénéfices sont si alléchants que les informaticiens ont créé des langages comme Haskell qui autorisent *uniquement* des fonctions pures.

    </aside>

    [ob]Now, imagine right in the middle of that function is a call to
    `SomeClass::getSomeGlobalData()`. To figure out what's going on, we have
    to hunt through the entire codebase to see what touches that global data.
    You don't really hate global state until you've had to `grep` a million
    lines of code at three in the morning trying to find the one errant call
    that's setting a static variable to the wrong value.[oe]

    Maintenant, imaginez vous que juste au milieu de cette fonction il y a un appel à `SomeClass::getSomeGlobalData()`. Pour découvrir ce qu'il se passe, nous avons à chasser dans le codebase entier pour voir qui touche cette donnée globale. Vous ne détestez pas vraiment l'état global jusqu'à ce que vous ayez à `grep` un million de lignes de code à trois heures du mat' en essayant de trouver l'appel errant qui configure une variable statique à la mauvaise valeur.

* [ob]**They encourage coupling.** The new coder on your team isn't familiar with
    your game's beautifully maintainable loosely coupled architecture, but he's
    just been given his first task: make boulders play sounds when they crash
    onto the ground. You and I know we don't want the physics code to be coupled
    to *audio* of all things, but he's just trying to get his task done.
    Unfortunately for us, the instance of our `AudioPlayer` is globally visible.
    So, one little `#include` later, and our new guy has compromised a carefully
    constructed architecture.[oe]
	
* **Elles encouragent le couplage.** Le nouveau codeur de votre équipe n'est pas familier avec votre architecture faiblement couplée et magnifiquement maintenable, mais il lui a été donné sa première tâche : faire que le rochers jouent des sons quand ils percutent le sol. Vous et moi ne voulons pas que le code physique soit couplé à l'*audio* produite par tout le monde, mais il essaie juste de faire sa tâche. Malheureusement pour nous, l'instance de notre `AudioPlayer` est globalement visible. Donc, un petit `#include` plus tard, et notre nouveau gars a compromis une architecture soigneusement construite.

    [ob]Without a global instance of the audio player, even if he *did* `#include`
    the header, he still wouldn't be able to do anything with it. That
    difficulty sends a clear message to him that those two modules should not
    know about each other and that he needs to find another way to solve his
    problem. *By controlling access to instances, you control coupling.*[oe]

	Sans instance globale du player audio, même si il avait `#include` l'en-tête, il ne devrait pas être capable de faire quoi que soit avec. Cette difficulté lui envoie un message clair, ces deux modules ne devraient pas se connaitre et il a besoin de trouver une autre façon de résoudre son problème. *En controllant l'accès aux instances, vous controllez le couplage.

* [ob]**They aren't concurrency-friendly.** The days of games running on a simple
    single-core CPU are pretty much over. Code today must at the very least
    *work* in a multi-threaded way even if it doesn't take full advantage of
    concurrency. When we make something global, we've created a chunk of memory
    that every thread can see and poke at, whether or not they know what other
    threads are doing to it. That path leads to deadlocks, race conditions, and
    other hell-to-fix thread-synchronization bugs.[oe]

* **Elles ne sont favorables à la concurrence.** Le temps des jeux tournant sur un simple CPU mono-core est plus ou moins révolu. Le code d'aujourd'hui doit au minimum *fonctionner* d'un manière multi-threadée même si il ne prend pas plein avantage de la concurrence. Quand nous faisons des globales, nous créons un morceau de mémoire que chaque thread peut voir et toucher, qu'ils savent ou non ce que d'autres threads sont en train de faire sur lui. Cette manière mène à des interblocages, des situations de compétition, et autres bugs de synchronisation de thread de la mort qui tue. 

[ob]Issues like these are enough to scare us away from declaring a global variable,
and thus the Singleton pattern too, but that still doesn't tell us how we
*should* design the game. How do you architect a game without global state?[oe]

Des problèmes comme ceux-ci sont suffisant pour nous dissuader de déclarer une variable globale, et donc du patron Singleton également, mais ça ne nous dit pas encore comment nous devrions concevoir le jeu. Comment architecturerez vous un jeu sans état global ?

[ob]There are some extensive answers to that question (most of this book in many
ways *is* an answer to just that), but they aren't apparent or easy to come by.
In the meantime, we have to get games out the door. The Singleton pattern looks
like a panacea. It's in a book on object-oriented design patterns, so it *must*
be architecturally sound, right? And it lets us design software the way we have
been doing for years.[oe]

Il y a quelque réponses extensives à cette question (la majeure partie de ce livre de maintes façons *est* juste une réponse à ceci), mais elles sont pas apparentes ou facile à trouver. En même temps, nous devons sortir des jeux. Le patron Singleton semble la panacée. C'est dans un livre sur les patrons de conception orienté-objet, donc ça *doit* être architecturalement solide, n'est ce pas? Et ça nous permet de concevoir du logiciel comme nous l'avons fait pendant des années.

[ob]Unfortunately, it's more placebo than cure. If you scan the list of problems
that globals cause, you'll notice that the Singleton pattern doesn't solve any
of them. That's because a singleton *is* global state -- it's just encapsulated in a
class.[oe]

Malheureusement, c'est plus un placebo qu'un cure. Si vous lisez la liste de problèmes que posent les globales, vous noterez que le patron Singleton n'en résout aucun. C'est par ce qu'un singleton *est* un état global -- c'est juste enveloppé dans une classe.

### Ca résout deux problèmes quand vous n'en avez qu'un

[ob]The word "and" in the Gang of Four's description of Singleton is a bit strange.
Is this pattern a solution to one problem or two? What if we have only one of
those? Ensuring a single instance is useful, but who says we want to let
*everyone* poke at it? Likewise, global access is convenient, but that's true
even for a class that allows multiple instances.[oe]

Le mot "et" dans la description du Singleton par le Gang of Four est un petit peu étrange. Ce patron est-il une solution à un ou deux problèmes ? Que dire si nous avons seulement un des deux ? S'assurer d'une unique instance est utile, mais qui dit que nous voulons permettre à *n'importe qui* d'y toucher. Egalement, l'accès global est pratique, mais ceci est vrai même pour pour une classe qui autorise de multiples instances.

[ob]The latter of those two problems, convenient access, is almost always why we
turn to the Singleton pattern. Consider a logging class. Most modules in the
game can benefit from being able to log diagnostic information. However, passing
an instance of our `Log` class to every single function clutters the method
signature and distracts from the intent of the code.[oe]

le dernier de ces deux problèmes, l'accès pratique, est presque toujours pourquoi nous utilisons le patron Singleton. Considerez une classe de journalisation. La plupart des modules dans le jeu peuvent en bénéficier en étant capable d'enregistrer des informations de diagnostique. Cependant, passer une instance de notre classe `Log` à chaque fonction embarrasse la signature de la méthode et détourne de l'attention le but du code.

[ob]The obvious fix is to make our `Log` class a singleton. Every function can then
go straight to the class itself to get an instance. But when we do that, we
inadvertently acquire a strange little restriction. All of a sudden, we can no
longer create more than one logger.[oe]

La correction évidente est de faire de notre classe de `Log` un singleton. Chaque fonction peut alors aller directement à la classe elle-même pour obtenir une instance. Mais quand nous faisons ça, nous acquérons par inadvertance une étrange et petite restriction. Tout d'un coup, nous ne pouvons plus créer plus d'un journal.

[ob]At first, this isn't a problem. We're writing only a single log file, so we only
need one instance anyway. Then, deep in the development cycle, we run into
trouble. Everyone on the team has been using the logger for their own
diagnostics, and the log file has become a massive dumping ground. Programmers
have to wade through pages of text just to find the one entry they care about.[oe]

Au premier abord, ça n'est pas un problème. Nous écrivons seulement dans un unique fichier de journal, donc nous n'avons besoin que d'une instance. Puis, plus loin dans le cycle de développement, nous nous heurtons à des difficultés. Chacun dans l'équipe ont utilisés le journal pour leur propres diagnostiques, et le fichier de journal est devenu une immense décharge publique. Les programmeurs ont à avancer péniblement à travers des pages de texte juste pour trouver l'enregistrement dont il se préoccupe.

[ob]We'd like to fix this by partitioning the logging into multiple files. To do
this, we'll have separate loggers for different game <span
name="worse">domains</span>: online, UI, audio, gameplay. But we can't. Not only
does our `Log` class no longer allow us to create multiple instances, that
design limitation is entrenched in every single call site that uses it:[oe]

Nous aimerions corriger ceci en partitionnant la journalisation en de multiple fichiers. Pour faire ceci, nous aurons des journaux pour les différents <span name="worse">domaines</span> de jeu : online, UI, audio, gameplay. Mais nous ne pouvons pas. Pas seulement par ce que notre classe `Log` ne nous autorise plus à créer de multiples instances, cette limitation de conception est enracinée à chaque endroit où on l'utilise :

    Log::instance().write("Some event.");

[ob]In order to make our `Log` class support multiple instantiation (like it
originally did), we'll have to fix both the class itself and every line of code
that mentions it. Our convenient access isn't so convenient anymore.[oe]

En vue de faire supporter à notre classe `Log` de multiples instanciations (comme elle le faisait originellement), nous aurons à corriger à la fois la classe elle-même et chaque ligne de code qui la mentionne. Notre classe pratique ne l'est plus vraiment.

<aside name="worse">

[ob]It could be even worse than this. Imagine your `Log` class is in a library being
shared across several *games*. Now, to change the design, you'll have to
coordinate the change across several groups of people, most of whom have neither
the time nor the motivation to fix it.[oe]

Ca pourrait être même pire que ça. Imaginez votre classe `Log` est dans une bibliothèque partagée par plusieurs *jeux*. Maintenant, pour changer la conception, vous aurez à coordonner le changement sur plusieurs groupes de personnes, la plupart d'entre eux n'ayant ni le temps ni la motivation de corriger ça.

</aside>

### L'initialisation paresseuse prend le contrôle

[ob]In the desktop PC world of virtual memory and soft performance requirements,
lazy initialization is a smart trick. Games are a different animal. Initializing
a system can take time: allocating memory, loading resources, etc. If
initializing the audio system takes a few hundred milliseconds, we need to
control when that's going to happen. If we let it lazy-initialize itself the
first time a sound plays, that could be in the middle of an action-packed part
of the game, causing visibly dropped frames and stuttering gameplay.[oe]

Dans le monde du PC de la mémoire virtuelle et des exigences basses de performance, l'initialisation paresseuse est une astuce intelligente. Le jeu est un animal différent. Initialiser un système peut prendre du temps : allouer de la mémoire, charger des ressources, etc. Si initialiser le système audio prend quelques centaines de millisecondes, nous avons besoin de contrôler quand ça va arriver. Si nous lui permettons de s'initialiser paresseusement lui-même à la première fois que l'on joue un son, ça pourrait être au milieu d'une action du jeu, causant un baisse du nombre d'images par seconde et un gameplay saccadé.

[ob]Likewise, games generally need to closely control how memory is laid out in the
heap to avoid <span name="fragment">fragmentation</span>. If our audio
system allocates a chunk of heap when it initializes, we want to know *when*
that initialization is going to happen, so that we can control *where* in the
heap that memory will live.[oe]

Egalement, les jeux ont généralement besoin de contrôler de très près comment la mémoire est agencée dans le tas pour éviter la <span name="fragment">fragmentation</span>. Si notre système audio alloue un morceau de tas quand il initialise, nous voulons savoir *quand* cette initialisation va arriver, pour que nous puissions contrôler *où* cette mémoire sera dans le tas.

<aside name="fragment">

[ob]See <a class="pattern" href="object-pool.html">Object Pool</a> for a detailed
explanation of memory fragmentation.[oe]

Voir <a class="pattern" href="object-pool.html">Piscine d'objet</a> pour une explication détaillée de la fragmentation de la mémoire.

</aside>

[ob]Because of these two problems, most games I've seen don't rely on lazy
initialization. Instead, they implement the Singleton pattern like this:[oe]

A cause de ces deux problèmes, la plupart de jeux que j'ai vu ne repose pas sur l'initialisation paresseuse. A la place, ils implémentent le patron Singleton comme ceci :

^code 5

[ob]That solves the lazy initialization problem, but at the expense of discarding
several singleton features that *do* make it better than a raw global variable.
With a static instance, we can no longer use polymorphism, and the class must be
constructible at static initialization time. Nor can we free the memory that the
instance is using when not needed.[oe]

Ca résoud le problème d'initialisation paresseuse, mais au prix de se séparer de plusieurs fonctionnalités du singleton qui le *rende* meilleur qu'une variable globale brute. Avec une instance statique, nous ne pouvons plus utiliser le polymorphisme, et la classe doit être constructible au temps de l'initialisation statique. Nous ne pouvons pas non plus libérer la mémoire que l'instance utilise quand elle n'est pas nécessaire.

[ob]Instead of creating a singleton, what we really have here is a simple static
class. That isn't necessarily a bad thing, but if a static class is all you
need, <span name="static">why not</span> get rid of the `instance()` method
entirely and use static functions instead? Calling `Foo::bar()` is simpler than
`Foo::instance().bar()`, and also makes it clear that you really are dealing
with static memory.[oe]

Au lieu de créer un singleton, ce que nous avons vraiment ici est une simple classe statique. Ca n'est pas nécessairement une mauvaise chose, mais si une classe statique est tout ce que vous avez, <span name="static">pourquoi ne pas</span> se débarrasser de la méthode `instance()` entièrement et utiliser des fonctions statiques à la place ? Appeler `Foo::bar()` est plus simple que `Foo::instance().bar()`, et aussi rend plus clair que vous travaillez avec de la mémoire statique.

<aside name="static">

[ob]The usual argument for choosing singletons over static classes is that if you
decide to change the static class into a non-static one later, you'll need to
fix every call site. In theory, you don't have to do that with singletons
because you could be passing the instance around and calling it like a normal
instance method.[oe]

L'argument habituel pour le choix des singletons plutôt que des classes statiques est que si vous décidez de changer la classe statique en une non-statique plus tard, vous aurez besoin de corriger chaque appel à celle-ci. En théorie, vous n'avez pas à faire cela avec des singletons par ce que vous pourriez passer l'instance quelque part et l'appeler comme une méthode d'instance normale.

[ob]In practice, I've never seen it work that way. Everyone just does
`Foo::instance().bar()` in one line. If we changed Foo to not be a
singleton, we'd still have to touch every call site. Given that, I'd rather have
a simpler class and a simpler syntax to call into it.[oe]

En pratique, je n'ai jamais vu ça fonctionner de cette façon. Tout le monde fait juste `Foo::instance().bar()` en une ligne. Si nous n'avions pas changé Foo pour ne pas être un singleton, nous aurions à modifier chaque appel. Etant donné cela, j'aurais une classe plus simple et une syntaxe plus simple pour appeler.

</aside>

## Ce que nous faisons à la place

[ob]If I've accomplished my goal so far, you'll think twice before you pull
Singleton out of your toolbox the next time you have a problem. But you still
have a problem that needs solving. What tool *should* you pull out? Depending on
what you're trying to do, I have a few options for you to consider, but first...[oe]

Si j'ai accompli mon but jusqu'ici, vous y refléchirez deux fois avant de sortir le Singleton de votre boite à outils la prochaine que vous avez un problème. Mais vous avez encore un problème qui a besoin d'être résolu. Quel outil *devriez* vous sortir ? Dépendant de ce que vous êtes en train d'essayer de faire, j'ai quelques options pour vous à considerer, mais d'abord...

### Voyons si vous avez vraiment besoin de classe

[ob]Many of the singleton classes I see in games are "managers" -- those nebulous
classes that exist just to babysit other objects. I've seen codebases where it
seems like *every* class has a manager: Monster, MonsterManager, Particle,
ParticleManager, Sound, SoundManager, ManagerManager. Sometimes, for variety,
they'll throw a "System" or "Engine" in there, but it's still the same idea.[oe]

Maintes classes singletons que je vois dans les jeux sont des "managers" -- ces classes nébuleuses qui existent juste pour baby-sitter d'autre objets. J'ai vu des codebases où il semblait que *chaque* classe avait un manager : Monster, MonsterManager, Particle, ParticleManager, Sound, SoundManager, ManagerManager. Parfois, pour varier, ils sèment un "System" ou un "Engine" par-ci par-là, mais c'est encore la même idée.

[ob]While caretaker classes are sometimes useful, often they just reflect
unfamiliarity with OOP. Consider these two contrived classes:[oe]

Tandis que des concierges de classes sont parfois utiles, souvent ils reflètent juste le manque de familiarité avec la POO. Considerez ces deux classes artificielles :

^code 8

[ob]Maybe this example is a bit dumb, but I've seen plenty of code that reveals a
design just like this after you scrape away the crusty details. If you look at
this code, it's natural to think that `BulletManager` should be a singleton. After
all, anything that has a `Bullet` will need the manager too, and how many
instances of `BulletManager` do you need?[oe]

Peut être que cet exemple est un peu bête, mais j'ai vu plein de code qui révélait une conception comme celle-ci après avoir racler les détails croustillants. Si vous regardez ce code, il est naturel de penser que `BulletManager` devrait être un singleton. Après tout, tout ce qui a une `Bullet` aura besoin du manager aussi, et de combien d'instances de `BulletManager` avez vous besoin ?

[ob]The answer here is *zero*, actually. Here's how we solve the "singleton" problem
for our manager class:[oe]

La réponse est *zéro*, en fait. Voici comment résoudre le problème du "singleton" pour classe manager :

^code 9

[ob]There we go. No manager, no problem. Poorly designed singletons are often
"helpers" that add functionality to another class. If you can, just move all of
that behavior into the class it helps. After all, OOP is about letting objects
take care of themselves.[oe]

Et voilà. Pas de manager, pas de problème. Des singletons mal conçus sont souvent des "assitants" qui ajoutent de la fonctionnalité à une autre classe. Si vous pouvez, déplacez juste ce comportement dans la classe assistée. Après tout, la POO c'est permettre aux objets de prendre soin d'eux-même.

[ob]Outside of managers, though, there are other problems where we'd reach to
Singleton for a solution. For each of those problems, there are some alternative
solutions to consider.[oe]

En dehors des managers, malgré tout, il y a d'autres problèmes pour lesquels nous devrions aboutir à Singleton comme solution. Pour chacun de ces problèmes, il y a des solutions alternatives à considérer.

### Limiter une classe à une unique instance

[ob]This is one half of what the Singleton pattern gives you. As in our file system
example, it can be critical to ensure there's only a single instance of a class.
However, that doesn't necessarily mean we also want to provide *public*,
*global* access to that instance. We may want to restrict access to certain
areas of the code or even make it <span name="wrapper">private</span> to a
single class. In those cases, providing a public global point of access weakens
the architecture.[oe]

Ceci est la moitié de ce que vous offre le patron Singleton. Comme dans notre exemple de système de fichier, il peut être critique de s'assurer qu'il y a seulement une instance d'une classe. Cependant, ça ne veut pas nécessairement dire que nous voulons aussi fournir un accès *publique* et *global* à l'instance. Nous aimerions restreindre l'accès à certains endroits du code ou même la rendre <span name="wrapper">privée</span> à une unique classe. Dans ces cas, fournir un point d'accès publique et global affaiblit l'architecture.

<aside name="wrapper">

[ob]For example, we may be wrapping our file system wrapper inside *another* layer
of abstraction.[oe]

Par exemple, nous pourrions envelopper notre enveloppe de système de fichier à l'intérieur d'une *autre* couche d'abstraction.

</aside>

[ob]We want a way to ensure single instantiation *without* providing global access.
There are a couple of ways to accomplish this. Here's one:[oe]

Nous voulons une manière de d'assurer une unique instanciation *sans*  fournir un accès global. Il y a différentes façons de réaliser ceci. En voici une :

<span name="assert"></span>

^code 6

[ob]This class allows anyone to construct it, but it will assert and fail if you try to
construct more than one instance. As long as the right code creates the instance
first, then we've ensured no other code can either get at that instance or
create their own. The class ensures the single instantiation requirement it
cares about, but it doesn't dictate how the class should be used.[oe]

Cette classe autorise quiconque à la construire, mais elle lancera l'assert et échouera si vous essayez de construire plus qu'une instance. Du moment que le bon code crée l'instance en premier, alors nous sommes assurés qu'aucun autre code ne peut obtenir cette instance ou créer la leur. La classe assure que l'exigence de l'unique instanciation dont il se soucie, mais elle ne dicte pas comment la classe devrait être utilisée.

<aside name="assert">

[ob]An *assertion* function is a way of embedding a contract into your code.
When `assert()` is called, it evaluates the expression passed to it. If it
evaluates to `true`, then it does nothing and lets the game continue. If it
evaluates to `false`, it immediately halts the game at that point. In a
debug build, it will usually bring up the debugger or at least print out the
file and line number where the assertion failed.[oe]

Une fonction d' *assertion* est une manière d'embarquer un contrat dans votre code. Quand `assert`  est appelée, elle évalue l'expression qui lui est passée. Si elle l'évalue à `true`, alors elle ne fait rien et permet au jeu de continuer. Si elle l'évalue à `false`  elle arrête le jeu à cet endroit. Dans une construction de debug, ça lancera le debugger ou au moins affichera le fichier et le numéro de ligne où l'assertion a échouée.

[ob]An `assert()` means, "I assert that this should always be true. If it's not,
that's a bug and I want to stop *now* so you can fix it." This lets you
define contracts between regions of code. If a function asserts that one of
its arguments is not `NULL`, that says, "The contract between me and the
caller is that I will not be passed `NULL`."[oe]

Un `assert()` veut dire, "J'affirme que ceci devrait toujours être vrai. Si ça ne l'est pas, c'est un bug et je veux stopper *maintenant* pour que vous puissiez le corriger." Ceci vous permet de définir des contrats entre des régions de code. Si une fonction assure qu'un des ces arguments n'est pas NULL, ça dit, "Le contrat entre moi et l'appelant est que je ne serai pas passée avec NULL".

[ob]Assertions help us track down bugs as soon as the game does something
unexpected, not later when that error finally manifests as something
visibly wrong to the user. They are fences in your codebase, corralling bugs
so that they can't escape from the code that created them.[oe]

Les assertions nous aide à traquer les bugs dès que le jeu fait quelque chose d'inattendu, pas plus tard quand cette erreur manifeste finalement quelque chose de visiblement faux à l'utilisateur. Elles sont des clôtures dans votre codebase, encerclant les bugs pour qu'ils ne puissent s'échapper du code qui les a créé.

</aside>

[ob]The downside with this implementation is that the check to prevent multiple
instantiation is only done at *runtime*. The Singleton pattern, in contrast,
guarantees a single instance at compile time by the very nature of the class's
structure.[oe]

L'inconvénient avec cette implémentation est que la vérification pour prévenir de multiple instanciation est seulement garantie au *temps d'exécution*. Le patron Singleton, en contraste, garantie une unique instance au temps de compilation par la nature de la structure de la classe.

### Fournir un accès pratique à une instance

[ob]Convenient access is the main reason we reach for singletons. They make it easy
to get our hands on an object we need to use in a lot of different places. That
ease comes at a cost, though -- it becomes equally easy to get our hands on the
object in places where we *don't* want it being used.[oe]

Un accès pratique est la principale raison pour laquelle nous aboutissons aux singletons. Ils nous permettent de facilement mettre la main sur un objet que nous avons besoin d'utiliser en de nombreux endroits. Cette facilité vient avec un coût, cependant -- il devient également facile de mettre la main sur un objet en des endroits où nous *ne* voulons pas qu'il soit utilisé.

[ob]The general rule is that we want variables to be as narrowly scoped as possible
while still getting the job done. The smaller the scope an object has, the fewer
places we need to keep in our head while we're working with it. Before we take
the shotgun approach of a singleton object with *global* scope, let's consider
other ways our codebase can get access to an object:[oe]

La règle générale est que nous voulons des variables d'une portée aussi petite que possible tout en ayant le travail fait. Plus petite est la portée d'un objet, moins nous avons d'endroits à mémoriser dans notre tête quand on travaille avec. Avant nous prenions l'approche shotgun d'un objet singleton avec de la portée *globale*, considérons d'autres manières par lesquelles notre codebase peut accèder à un objet :

* [ob]**Pass it in.** The <span name="di">simplest</span> solution, and often the
    best, is to simply pass the object you need as an argument to the functions
    that need it. It's worth considering before we discard it as too cumbersome.[oe]

 * **Le passer en entrée.** La solution <span name="di">la plus simple</span>, et souvent la meilleure, est de passer simplement l'objet dont vous avez besoin comme un argument à la fonction qui en a besoin. Il est utile de considérer ceci avant de le juger comme trop lourd.
	
    <aside name="di">

    [ob]Some use the term "dependency injection" to refer to this. Instead of code
    reaching *out* and finding its dependencies by calling into something
    global, the dependencies are pushed *in* to the code that needs it through
    parameters. Others reserve "dependency injection" for more complex ways of
    providing dependencies to code.[oe]

    Certains utilisent le terme d' "injection de dépendance" pour se référer à ça. Au lieu du code essayant d'atteindre en dehors et trouver ces dépendances en appelant dans quelque chose de globale, les dépendances sont poussées *dans* le code qui en a besoin via ces paramètres. D'autres réserve l'"injection de dépendance" pour des façons plus complexes de fournir des dépendances au code.

    </aside>

    [ob]Consider a function for rendering objects. In order to render, it needs
    access to an object that represents the graphics device and maintains the
    render state. It's very common to simply pass that in to all of the rendering
    functions, usually as a parameter named something like `context`.[oe]

    Considerez une fonction pour rendre des objets. En vue de rendre, elle a besoin d'accèder à un objet qui représente le dispositif graphique et de maintenir l'état du rendu. C'est très commun de passer ça simplement à tous les fonctions de rendu, habituellement comme un paramètre nommé, quelque chose comme `context`.

    [ob]On the other hand, some objects don't belong in the signature of a method.
    For example, a function that handles AI may need to also write to a <span
    name="aop">log file</span>, but logging isn't its core concern. It would be
    strange to see `Log` show up in its argument list, so for cases like that
    we'll want to consider other options.[oe]

    D'un autre côté, certains objets n'a rien à faire dans la signature d'une méthode. Par exemple, une fonction qui gère l'IA pourrait aussi avoir besoin d'écrire dans un <span
    name="aop">fichier de journal</span>, mais la journalisation n'est pas dans sa liste d'argument, donc pour des cas comme celui là nous voudrons considérer d'autres options.

    <aside name="aop">

    [ob]The term for things like logging that appear scattered throughout a codebase
    is "cross-cutting concern". Handling cross-cutting concerns gracefully is
    a continuing architectural challenge, especially in statically typed
    languages.[oe]

	Le terme pour des choses comme la journalisation qui apparaisse découpées à travers un codebase est "préoccupation transversale". Gérer les préoccupations transversales gracieusement est challenge architectural en continu, spécialement dans les langages statiquement typés.

    [ob][Aspect-oriented
    programming](http://en.wikipedia.org/wiki/Aspect-oriented_programming) was
    designed to address these concerns.[oe]

	[La programmation orientée-aspect](http://en.wikipedia.org/wiki/Aspect-oriented_programming) a été conçue pour répondre à ces préoccupations.

    </aside>

* [ob]**Get it from the base class.** Many game architectures have shallow but
    wide inheritance hierarchies, often only one level deep. For example, you
    may have a base `GameObject` class with derived classes for each enemy or
    object in the game. With architectures like this, a large portion of the
    game code will live in these "leaf" derived classes. This means that all
    these classes already have access to the same thing: their `GameObject` base
    class. We can use that to our advantage:[oe]

 *  **L'obtenir de la classe de base.** Beaucoup d'architectures de jeu ont de peu profondes mais de 	  larges hiérarchies, souvent seulement un niveau de profondeur. Par exemple, vous pourriez avoir une classe de base `GameObject` avec des classes dérivées pour chaque ennemis ou objet dans le jeu. Avec des architectures comme celle-ci, une grande portion de code du jeu sera dans ces "feuilles" des classes dérivées. Ceci veut dire que toutes ces classes ont déjà accès à la même chose : leur classe de base `GameObject`. Nous pouvons utiliser ça à notre avantage :

    <span name="gameobject"></span>

    ^code 10

    [ob]This ensures nothing outside of `GameObject` has access to its `Log` object,
    but every derived entity does using `getLog()`. This pattern of letting
    derived objects implement themselves in terms of protected methods provided
    to them is covered in the <a class="pattern"
    href="subclass-sandbox.html">Subclass Sandbox</a> chapter.[oe]

    Ceci assure que rien en dehors de `GameObject` n'a accès à son objet `Log`, mais toute entité dérivée utilise `getLog()`. Ce patron qui permet à des objets dérivés d'implémenter eux-même en terme de méthodes protégées fourni à eux est couverte dans le chapitre <a class="pattern"
    href="subclass-sandbox.html">Bac à sable de sous-classe</a>.

    <aside name="gameobject">

    [ob]This raises the question, "how does `GameObject` get the `Log` instance?" A
    simple solution is to have the base class simply create and own a static
    instance.[oe]
	
	Ceci soulève la question, "comment `GameObject` obtient l'instance de `Log` ? Une solution simple est d'avoir la classe de base qui créé et possède une instance statique.

    [ob]If you don't want the base class to take such an active role, you can
    provide an initialization function to pass it in or use the <a
    class="pattern" href="service-locator.html">Service Locator</a> pattern to
    find it.[oe]

	Si vous ne voulez pas que la classe de base prenne un tel rôle, vous pouvez fournir une fonction d' initialisation pour la passer en paramètre ou utiliser le patron <a
    class="pattern" href="service-locator.html">Localisateur de Service</a> pour la trouver.

    </aside>

* [ob]**Get it from something already global.** The goal of removing *all* global
    state is admirable, but rarely practical. Most codebases will still have a
    couple of globally available objects, such as a single `Game` or `World`
    object representing the entire game state.[oe]

 *  **L'obtenir à partir de quelque chose de globale.** Le but d'effacer *tout* état global est admirable, mais rarement pratique. La plupart des codebases auront encore quelque objets globaux disponibles, tel qu'un objet unique `Game` ou `World` représentant l'état du jeu complet.

    [ob]We can reduce the number of global classes by piggybacking on existing ones
    like that. Instead of making singletons out of `Log`, `FileSystem`, and
    `AudioPlayer`, do this:[oe]

	Nous pouvons réduire le nombre de classes globales en les insérant dans des classes existantes comme ces dernières. Au lieu de faire des singletons de `Log`, `FileSystem`, et `AudioPlayer`, faites ceci :

    ^code 11
  
    [ob]With this, only `Game` is globally available. Functions can get to the
    other systems <span name="demeter">through</span> it:[oe]
 
	Avec ceci, seulement `Game` est disponible globalement. Des fonctions peuvent récupérer à d'autres systèmes à <span name="demeter">travers</span> lui : 

    ^code 12

    <aside name="demeter">

    [ob]Purists will claim this violates the Law of Demeter. I claim that's still
    better than a giant pile of singletons.[oe]

    Des puristes déclarerons que ceci viole la Loi de Demeter. Je déclare que c'est mieux qu'une pile géante de singletons.

    </aside>

    [ob]If, later, the architecture is changed to support multiple `Game` instances
    (perhaps for streaming or testing purposes), `Log`, `FileSystem`, and
    `AudioPlayer` are all unaffected -- they won't even know the difference. The
    downside with this, of course, is that more code ends up coupled to `Game`
    itself. If a class just needs to play sound, our example still requires it
    to know about the world in order to get to the audio player.[oe]

    Si, plus tard, l'architecture est changée pour supporter de multiples instances de `Game` (peut être à but de streaming ou de test), `Log`, `FileSystem`, et `AudioPlayer` sont tous non affectés -- Ils ne verront même pas la différence. L'inconvénient avec ceci, bien sûr, est que plus de code finit couplé à `Game` elle-même. Si une classe a juste besoin de jouer des sons, notre exemple l'exige encore pour connaitre le monde en vue de récupérer le player audio.

    [ob]We solve this with a hybrid solution. Code that already knows about `Game`
    can simply access `AudioPlayer` directly from it. For code that doesn't, we
    provide access to `AudioPlayer` using one of the other options described
    here.[oe]

	Nous résolvons ceci avec un solution hybride. Du code qui connait déjà `Game` peut simplement accéder à `AudioPlayer` directement à partir de lui. Pour du code qui ne connait le pas, nous fournissons un accès à `AudioPlayer` en utilisant une des autres options décrites ici.

* [ob]**Get it from a Service Locator.** So far, we're assuming the global class
    is some regular concrete class like `Game`. Another option is to define a
    class whose sole reason for being is to give global access to objects. This
    common pattern is called a <a class="pattern"
    href="service-locator.html">Service Locator</a> and gets its own chapter.[oe]

 *  **L'obtenir à partir d'un Localisateur de Service.** Jusqu'ici, nous supposions que la classe globale est une classe concrète ordinaire comme `Game`. Une autre option est de définir une classe à qui la seule raison d'exister est de donner un accès global à des objets. Ce patron commun est appelé un <a class="pattern" href="service-locator.html">Localisateur de service</a> et a son propre chapitre.

## Ce qu'il reste pour le Singleton

[ob]The question remains, where *should* we use the real Singleton pattern?
Honestly, I've never used the full Gang of Four implementation in a game. To
ensure single instantiation, I usually simply use a static class. If that
doesn't work, I'll use a static flag to check at runtime that only one instance
of the class is constructed.[oe]

La question reste, où *devrions* nous utiliser le vrai patron Singleton ? Honnêtement, je n'ai jamais utilisé l'implémentation complète du Gang of Four dans un jeu. Pour s'assurer une unique instanciation, j'utilise habituellement simplement une classe statique. Si ça ne fonctionne pas, j'utiliserai un flag statique pour vérifier au temps d'exécution que seulement une instance de la classe est construite.

[ob]There are a couple of other chapters in this book that can also help here. The
<a class="pattern" href="subclass-sandbox.html">Subclass Sandbox</a> pattern
gives instances of a class access to some shared state without making it
globally available. The <a class="pattern" href="service-locator.html">Service
Locator</a> pattern *does* make an object globally available, but it gives you more
flexibility with how that object is configured.[oe]

Il y a quelques autres chapitres dans ce livre qui peuvent aussi aider ici. Le patron <a class="pattern" href="subclass-sandbox.html">Bac à sable de sous-classe</a> donne des accès à des instances d'une classe à quelque état partagé sans le rendre globalement disponible. Le patron <a class="pattern" href="service-locator.html">Localisateur de service</a> *rend* un objet globalement disponible, mais il vous donne plus de flexibilité sur comment cet objet est configuré.
