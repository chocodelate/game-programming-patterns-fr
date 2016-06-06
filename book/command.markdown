^title Command
^section Design Patterns Revisited

[ob]Command is one of my favorite patterns. Most large programs I write, games or
otherwise, end up using it somewhere. When I've used it in the right place, it's
neatly untangled some really gnarly code. For such a swell pattern, the Gang of
Four has a predictably abstruse description:[oe]

Commande est un des mes patrons favoris. La plupart des gros programmes que j'écris, jeux ou autres, finissent par l'utiliser quelque part. Quand je l'ai utilisé au bon endroit, j'obtiens un code complexe soigneusement démêlé. Pour un si chouette patron, le Gang of Four à une description absconse prévisible : 

> [ob]Encapsulate a request as an object, thereby letting users parameterize clients with different requests, queue or log requests, and support undoable operations.[oe]

> Encapsuler une requête comme un objet, permettre aux utilisateurs de paramétrer des clients par ce moyen avec différentes requêtes, files ou requêtes de journalisation, et supporter les opérations annulables.

[ob]I think we can all agree that that's a terrible sentence. First of all, it
mangles whatever metaphor it's trying to establish. Outside of the weird world
of software where words can mean anything, a "client" is a *person* -- someone
you do business with. Last I checked, human beings can't be "parameterized".[oe]

Je crois qu'on peut s'accorder sur le fait que c'est une phrase horrible. En tout premier lieu, ça broie n'importe quelle métaphore que ca  essaie d'établir. Hors de l'étrange monde du logiciel où les mots peuvent vouloir dire quelque chose, un "client" est une *personne* -- quelqu'un avec qui tu fais du business. La dernière fois que j'ai vérifié, l'être humain ne peut pas être "paramétré".

[ob]Then, the rest of that sentence is just a list of stuff you could maybe possibly
use the pattern for. Not very illuminating unless your use case happens to be in
that list. *My* pithy tagline for the Command pattern is:[oe]

Donc, le reste de cette phrase est juste une liste de chose que vous pourriez possiblement utiliser avec ce patron. Pas vraiment éclairant à moins que votre cas d'utilisation soit dans cette liste. *Mon* slogan concis pour le patron Commande est :

**[ob]A command is a *<span name="latin">reified</span> method call*.[oe]**

**Une commande est un *appel de méthode <span name="latin">réifié</span>*.**

<aside name="latin">

[ob]"Reify" comes from the Latin "res", for "thing", with the English suffix
"&ndash;fy". So it basically means "thingify", which, honestly, would be a more
fun word to use.[oe]

Réifier vient du latin "res", pour "chose", avec le suffixe français "&ndash;fier". Ca signifie basiquement "chosifier", lequel, sincèrement, devrait être un mot plus amusant à employer.

</aside>

[ob]Of course, "pithy" often means "impenetrably terse", so this may not be much of
an improvement. Let me unpack that a bit. "Reify", in case you've never heard
it, means "make real". Another term for reifying is making something "first-class".[oe]

Bien sûr, "concis" signifie souvent "impénétrablement laconique", alors ceci pourrait ne pas être beaucoup mieux. Permettez moi de déconstruire ça un petit peu. "Réifier", dans le cas où vous ne l'auriez jamais entendu, signifie "rendre réel". Un autre terme pour la réification est de rendre quelque chose de "première classe".

<aside name="reflection">

[ob]*Reflection systems* in some languages let you work with the types in your
program imperatively at runtime. You can get an object that represents the class
of some other object, and you can play with that to see what the type can do. In
other words, reflection is a *reified type system*.[oe]

Les *systèmes de réflexion* dans certains langages vous permettent de travailler avec les types dans votre programme dans un style impératif à l'exécution. Vous pouvez récupérer un objet qui représente une classe d'un autre objet, et vous pouvez jouer avec ça pour voir ce que le type peut faire. En d'autres termes, la réflexion est un *système de type réifié*.

</aside>

[ob]Both terms mean taking some <span name="reflection">*concept*</span> and turning
it into a piece of *data* -- an object -- that you can stick in a variable, pass
to a function, etc. So by saying the Command pattern is a "reified method call",
what I mean is that it's a method call wrapped in an object.[oe]

Les deux termes signifient à la fois prendre un <span name="reflection">*concept*</span> et transformer ça en un morceau de *données* -- un objet -- que vous pouvez mettre dans une variable, passer à une fonction, etc. Donc en disant que le patron Commande est un "appel de méthode réifié", ce que je veux dire est que c'est un appel de méthode enveloppé dans un objet.

[ob]That sounds a lot like a "callback", "first-class function", "function pointer",
"closure", or "partially applied function" depending on which language you're
coming from, and indeed those are all in the same ballpark. The Gang of Four
later says:[oe]

Ca ressemble beaucoup à une "fonction de rappel", une "fonction de première classe", un "pointeur de fonction", une "fermeture", une "fonction partiellement appliquée" selon le langage que vous utilisez, et ils sont en effet tous de la même catégorie. Le Gang of Four plus loin dit :

> [ob]Commands are an object-oriented replacement for callbacks.[oe]

> Les commandes sont un équivalent des fonctions de rappel en orienté-objet.

[ob]That would be a better slugline for the pattern than the one they chose.[oe]

Ca aurait été un meilleur slogan par rapport à celui qu'ils ont choisi.

[ob]But all of this is abstract and nebulous. I like to start chapters with
something concrete, and I blew that. To make up for it, from here on out it's
all examples where commands are a brilliant fit.[oe]

Mais tout ceci est abstrait et complexe. J'aime démarrer les chapitres avec quelque chose de concret, et j'ai gâché ça. Pour rattraper ça, à partir de maintenant nous verrons tous les exemples où les commandes sont une excellente approche.

[ob]## Configuring Input[oe]

## Configurer l'entrée

[ob]Somewhere in every game is a chunk of code that reads in raw user input --
button presses, keyboard events, mouse clicks, whatever. It takes each input and
translates it to a meaningful action in the game:[oe]

Quelque part dans chaque jeu il  y a un morceau de code qui lit l'entrée brute d'un utilisateur -- pressions de boutons, évènement de clavier, cliques de souris, et bien d'autres. Il prend chaque entrée et traduit ça en une action significative dans le jeu :

<img src="images/command-buttons-one.png" alt="Un controlleur, avec A mappé à swapWeapon(), B mappé à lurch(), X mappé à jump(), et Y mappé à fireGun()." />

[ob]A dead simple implementation looks like:[oe]

Une implémentation toute simple ressemble à cela :

<span name="lurch"></span>

^code handle-input

<aside name="lurch">

[ob]Pro tip: Don't press B very often.[oe]

Conseil de pro : n'appuyez pas sur B trop souvent.

</aside>

[ob]This function typically gets called once per frame by the <a class="pattern"
href="game-loop.html">Game Loop</a>, and I'm sure you can figure out what it
does. This code works if we're willing to hard-wire user inputs to game actions,
but many games let the user *configure* how their buttons are mapped.[oe]

Cette fonction est typiquement appelée une fois par image par la <a class="pattern" href="game-loop.html"> Boucle de Jeu</a>, je suis sûr que vous pouvez découvrir ce qu'elle fait. Ce code fonctionne si nous sommes prêts à raccorder en dur les entrées utilisateur avec les actions du jeu, mais la plupart des jeux permette aux utilisateurs de *configurer* comment les boutons sont mappés.

[ob]To support that, we need to turn those direct calls to `jump()` and `fireGun()`
into something that we can swap out. "Swapping out" sounds a lot like assigning
a variable, so we need an *object* that we can use to represent a game action.
Enter: the Command pattern.[oe]

Pour le supporter, nous avons besoin de transformer ces appels directs à `jump()` et `fireGun()` en quelque chose nous pouvons échanger. "Echanger" est un peu comme assigner une variable, alors nous avons besoin d'un objet que nous pouvons l'utiliser pour représenter une action du jeu. Débutons : le patron Commande.


[ob]We define a base class that represents a triggerable game command:[oe]


Nous définissons une classe de base qui représente une commande de jeu déclenchable : 

<span name="one-method"></span>

^code command

<aside name="one-method">

[ob]When you have an interface with a single method that doesn't return anything,
there's a good chance it's the Command pattern.[oe]

Quand vous avez une interface avec une simple méthode qui ne retourne rien, il y a de bonnes chances que ce soit le patron Commande.

</aside>

[ob]Then we create subclasses for each of the different game actions:[oe]

Puis nous créons des sous-classes pour chacune des différentes actions :

^code command-classes

[ob]In our input handler, we store a pointer to a command for each button:[oe]

Dans notre gestionnaire d'entrée, nous stockons un pointeur vers une commande pour chaque bouton : 

^code input-handler-class

[ob]Now the input handling just delegates to those:[oe]

Maintenant la gestion d'entrée délègue ceux-ci : 

<span name="null"></span>

^code handle-input-commands

<aside name="null">

[ob]Notice how we don't check for `NULL` here? This assumes each button will have
*some* command wired up to it.
[oe]

Avez vous noté que nous ne vérifions pas le cas `NULL` ici ? Ca suppose que chaque bouton aura *une* commande raccordée à lui.

[ob]If we want to support buttons that do nothing without having to explicitly check
for `NULL`, we can define a command class whose `execute()` method does nothing.
Then, instead of setting a button handler to `NULL`, we point it to that object.
This is a pattern called [Null
Object](http://en.wikipedia.org/wiki/Null_Object_pattern).[oe]

Si nous voulons supporter les boutons qui ne font rien sans avoir à explicitement vérifier un NULL, nous pouvons définir une classe de commande pour laquelle la méthode `execute()` ne fait rien. Puis, au lieu de configurer un gestionnaire de bouton à `NULL`, Nous le faisons pointer vers cet objet.Ceci est un patron appelé [L'Objet Null](http://en.wikipedia.org/wiki/Null_Object_pattern).

</aside>

[ob]Where each input used to directly call a function, now there's a layer of
indirection:[oe]

Là où chaque entrée avait l'habitude d'appeler directement une fonction, maintenant il y a une couche d'indirection : 

<img src="images/command-buttons-two.png" alt="A controller, with each button mapped to a corresponding 'button_' variable which in turn is mapped to a function." />

[ob]This is the Command pattern in a nutshell. If you can see the merit of it
already, consider the rest of this chapter a bonus.[oe]

Ceci est le patron Commande en bref. Si vous pouvez déjà voir son mérite, considèrez le reste de ce chapitre comme un bonus.

[ob]## Directions for Actors[oe]

## Des mises en scène pour des acteurs

[ob]The command classes we just defined work for the previous example, but they're
pretty limited. The problem is that they assume there are these top-level
`jump()`, `fireGun()`, etc. functions that implicitly know how to find the
player's avatar and make him dance like the puppet he is.[oe]

Les classes de commande que nous venons de définir fonctionnent pour l'exemple précédent, mais elles sont assez limitées. Le problème est qu'elles supposent que ces fonction de haut niveau `jump()`, `fireGun()`, etc. savent implicitement comment trouver l'avatar du joueur et le faire danser telle la marionnette qu'il est.

[ob]That assumed coupling limits the usefulness of those commands. The *only* thing
the `JumpCommand` can make jump is the player. Let's loosen that restriction.
Instead of calling functions that find the commanded object themselves, we'll
*pass in* the object that we want to order around:[oe]

Ca supposait que le couplage limite l'utilité de ces commandes. La *seule* chose que `JumpCommand` peut faire sauter est le joueur. Essayons d'enlever cette restriction. Au lieu d'appeler des fonctions qui trouvent l'objet commandé elles-mêmes, nous *passerons* l'objet que nous voulons commander :

^code actor-command

[ob]Here, `GameActor` is our "game object" class that represents a character in the
game world. We pass it in to `execute()` so that the derived command can invoke
methods on an actor of our choice, like so:[oe]

Ici, `GameActor` est notre classe de "game object" qui représente un personnage dans le monde du jeu. Nous le passons à  `execute()` pour que la commande dérivée puisse invoquer des méthodes sur un acteur de notre choix, comme ceci :

^code jump-actor

[ob]Now, we can use this one class to make any character in the game hop around.
We're just missing a piece between the input handler and the command that takes
the command and invokes it on the right object. First, we change `handleInput()`
so that it *returns* commands:[oe]

Maintenant, nous pouvons utiliser cette classe pour n'importe quel personnage dans le jeu. Nous avons juste oublié un morceau entre le gestionnaire d'entrée et la requête qui prend la commande et l'invoque sur le bon objet. D'abord, nous modifions  `handleInput()` pour que ca  *retourne* les commandes :

^code handle-input-return

[ob]It can't execute the command immediately since it doesn't know what actor to
pass in. Here's where we take advantage of the fact that the command is a
reified call -- we can *delay* when the call is executed.[oe]

Il ne peut pas exécuter la commande immédiatement car il ne sait pas quel acteur passer. C'est là où nous prenons avantage du fait que la commande est un appel réifié -- nous pouvons *différer* quand l'appel est exécuté.

[ob]Then, we need some code that takes that command and runs it on the actor
representing the player. Something like:[oe]

Puis, nous avons besoin d'un peu de code qui prend la commande et la lance sur l'acteur représentant le joueur. Quelque chose comme :

^code call-actor-command

[ob]Assuming `actor` is a reference to the player's character, this correctly drives
him based on the user's input, so we're back to the same behavior we had in the
first example. But adding a layer of indirection between the command and the
actor that performs it has given us a neat little ability: *we can let the
player control any actor in the game now by changing the actor we execute
the commands on.*[oe]

En supposant que `actor` est une référence au personnage du joueur, ceci le conduit selon les entrées utilisateur, donc nous revenons au même comportement nous avions au premier exemple. Mais en ajoutant une couche d'indirection entre la commande et l'acteur qui l'effectue cela nous a donné une petite capacité ingénieuse : *nous pouvons maintenant permettre le contrôle du joueur de n'importe quel acteur dans le jeu en changeant l'acteur sur lequel on exécute les commandes.*

[ob]In practice, that's not a common feature, but there is a similar use case that
*does* pop up frequently. So far, we've only considered the player-driven
character, but what about all of the other actors in the world? Those are driven
by the game's AI. We can use this same command pattern as the interface between
the AI engine and the actors; the AI code simply emits `Command` objects.[oe]

En pratique, ceci n'est pas une fonctionnalité commune, mais il y a un cas d'utilisation qui la *fait* intervenir fréquemment. Jusqu'ici, nous avons seulement considéré le personnage piloté par un joueur, mais qu'en est il des autres acteurs dans le monde ? Ils sont pilotés par l'IA du jeu. Nous pouvons utiliser ce même patron commande comme l'interface entre le moteur IA et les acteurs; le code de l'IA émet simplement des objets `Command`.

[ob]The decoupling here between the AI that selects commands and the actor code
that performs them gives us a lot of flexibility. We can use different AI
modules for different actors. Or we can mix and match AI for different kinds of
behavior. Want a more aggressive opponent? Just plug-in a more aggressive AI to
generate commands for it. In fact, we can even bolt AI onto the *player's*
character, which can be useful for things like demo mode where the game needs to
run on auto-pilot.[oe]

Le découplage ici entre l'IA qui sélectionne les commandes et le code de l'acteur qui les effectue nous donne beaucoup de flexibilité. Nous pouvons utiliser différents modules d'IA pour différents acteurs. Ou nous pouvons mélanger et assortir l'IA pour différents genres de comportement. Vous voulez un adversaire plus agressif ? Attachez juste une IA plus agressive pour en générer les commandes. En fait, nous pouvons même mettre l'IA sur le personnage du *joueur*, ce qui peut être utile pour un mode démo où le jeux a besoin de tourner automatiquement.

[ob]<span name="queue">By</span> making the commands that control an actor
first-class objects, we've removed the tight coupling of a direct method call.
Instead, think of it as a queue or stream of commands:[oe]

<span name="queue">En</span> faisant que les commandes qui contrôlent un acteur des objets de première classe, nous avons supprimé le fort couplage d'un appel de méthode direct. Au lieu de ça, penser ça comme une file ou un flux de commandes :

<aside name="queue">

[ob]For lots more on what queueing can do for you, see <a href="event-queue.html"
class="pattern">Event Queue</a>.[oe]

Pour en savoir plus sur ce que peuvent faire les files d'attentes pour vous, allez voir <a href="event-queue.html" class="pattern">File d'évènement</a>.

</aside>

<span name="stream"></span>

<img src="images/command-stream.png" alt="A pipe connecting AI to Actor." />

<aside name="stream">

[ob]Why did I feel the need to draw a picture of a "stream" for you? And why does it
look like a tube?[oe]

Pourquoi ai-je senti le besoin de vous dessiner un "flux " ? Et pourquoi cela ressemble à un tube ?

</aside>

[ob]Some code (the input handler or AI) <span name="network">produces</span>
commands and places them in the stream. Other code (the dispatcher or actor
itself) consumes commands and invokes them. By sticking that queue in the
middle, we've decoupled the producer on one end from the consumer on the other.[oe]

Une partie du code (le gestionnaire d'entrée ou l'IA) <span name="network">produit</span> des commandes et les place dans le flux. Une autre partie du code (l'appelant ou l'acteur lui-même) consomme les commandes et les invoquent. En mettant cette file au milieu, nous avons découplé le producteur du consommateur.

<aside name="network">

[ob]If we take those commands and make them *serializable*, we can send the stream
of them over the network. We can take the player's input, push it over the
network to another machine, and then replay it. That's one important piece of
making a networked multi-player game.[oe]

Si nous prenons ces commandes et les rendons *sérialisables*, nous pouvons envoyer leur flux sur le réseau. Nous pouvons prendre les entrées du joueur, envoyer ça sur le réseau vers une autre machine, et puis le rejouer. C'est une partie importante quand on fait un jeu en réseau multijoueur.

</aside>

[ob]## Undo and Redo[oe]

## Annuler et Refaire

[ob]The final example is the most well-known use of this pattern. If a command object
can *do* things, it's a small step for it to be able to *undo* them. Undo is
used in some strategy games where you can roll back moves that you didn't like.
It's *de rigueur* in tools that people use to *create* games. The <span
name="hate">surest way</span> to make your game designers hate you is giving
them a level editor that can't undo their fat-fingered mistakes.[oe]

L'exemple final est l'utilisation la plus connue du patron. Si un objet commande peut *faire* des choses, nous n'avons pas grand chose à faire  pour être capable de les *annuler*. Défaire est utilisé dans certains jeux de stratégie dans lesquels vous pouvez annuler des coups que vous ne vouliez pas faire. c'est *de rigueur* dans les outils que les gens utilisent pour *créer* des jeux. La <span
name="hate">manière la plus sûre</span> de vous faire détester par votre concepteur de jeu est de lui donner un éditeur de niveau qui ne peut pas annuler ses erreurs causées par ses gros doigts.


<aside name="hate">

[ob]I may be speaking from experience here.[oe]

C'est l'expérience qui parle ici.

</aside>

[ob]Without the Command pattern, implementing undo is surprisingly hard. With it,
it's a piece of cake. Let's say we're making a single-player, turn-based game and
we want to let users undo moves so they can focus more on strategy and less on
guesswork.[oe]

Sans le patron Commande, implémenter l'annulation est étonnamment difficile. Avec, c'est du gâteau ! Disons que nous faisons un jeu solo au tour par tour, et que nous voulons permettre aux utilisateurs d'annuler des coups pour qu'ils se focalisent plus sur la stratégie que sur des suppositions.

[ob]We're conveniently already using commands to abstract input handling, so every
move the player makes is already encapsulated in them. For example, moving a
unit may look like:[oe]

Nous utilisons déjà des commandes pour abstraire la gestion d'entrée, donc chaque coups que le joueur fait est déjà encapsulé dans celles-ci. Par exemple, déplacer une unité pourrait ressembler à cela :

^code move-unit

[ob]Note this is a little different from our previous commands. In the last example,
we wanted to *abstract* the command from the actor that it modified. In this
case, we specifically want to *bind* it to the unit being moved. An instance of
this command isn't a general "move something" operation that you could use in a
bunch of contexts; it's a specific concrete move in the game's sequence of
turns.[oe]

Notez que ceci est un petit peu différent de nos précédentes commandes. Dans le dernier exemple, nous voulions *abstraire* les commandes grâce à l'acteur qu'elles modifiaient. Dans ce cas, nous voulons spécifiquement la *relier* à l'unité déplacée. Une instance de cette commande n'est pas une opération générale qui "déplace quelque chose" que vous pourrez utiliser dans un tas de contextes; c'est un coup particulier concret dans la séquence des tours du jeu. 

[ob]This highlights a variation in how the Command pattern gets implemented. In some
cases, like our first couple of examples, a command is a reusable object that
represents a *thing that can be done*. Our earlier input handler held on to a
single command object and called its `execute()` method anytime the right button
was pressed.[oe]

Ceci souligne une variation sur comment le patron Commande est implémenté. Dans quelques cas, comme nos premiers exemples, une commande est un objet réutilisable qui représente *une chose qui peut être faite*. Notre gestionnaire d'entrée précédent s'en tenait à un simple objet de commande et appelait sa méthode `execute()` à chaque fois le bon bouton était pressé.

[ob]Here, the commands are more specific. They represent a thing that can be done at
a specific point in time. This means that the input handling code will be <span
name="free">*creating*</span> an instance of this every time the player chooses
a move. Something like:
[oe]

Ici, les commandes sont un peu plus spécifiques. Elles représentent une chose qui peut être faite à un moment spécifique dans le temps. Ceci veut dire que le code du gestionnaire d'entrée <span
name="free">*créera*</span> une instance de ceci à chaque fois que le joueur choisit un coup. Quelque chose comme :

^code get-move

<aside name="free">

[ob]Of course, in a non-garbage-collected language like C++, this means the code
executing commands will also be responsible for freeing their memory.[oe]

Bien sûr, dans un langage comme C++ ne possédant pas de ramasse-miette, ceci veut dire que le code exécutant les commandes sera aussi responsable de la libération de leur mémoire.

</aside>

[ob]The fact that commands are one-use-only will come to our advantage in a second.
To make commands undoable, we define another operation each command class needs
to implement:[oe]

Le fait que les commandes sont à usage unique tournera à notre avantage dans une seconde. Pour rendre les commandes annulables, nous définissons une autre opération pour chaque commande que la classe a besoin d'implémenter :

^code undo-command

[ob]An `undo()` method reverses the game state changed by the corresponding
`execute()` method. Here's our previous move command with undo support:[oe]

Une méthode `undo()` inverse l'état du jeu changé par la méthode `execute()` correspondante. Voici notre commande de coup précédent avec la prise en charge de l'annulation :

^code undo-move-unit

[ob]Note that we added some <span name="memento">more state</span> to the class.
When a unit moves, it forgets where it used to be. If we want to be able to undo
that move, we have to remember the unit's previous position ourselves, which is
what `xBefore_` and `yBefore_` do.[oe]

Notez que nous avons ajouté quelques états en plus à la classe. Quand une unité se déplace, elle oublie où elle était avant. Si nous voulons être capable d'annuler son déplacement, nous avons à enregistrer la position précédente de l'unité nous-même, ce qui est fait par `xBefore_` et `yBefore_`.

<aside name="memento">

[ob]This seems like a place for the <a
href="http://en.wikipedia.org/wiki/Memento_pattern"
class="gof-pattern">Memento</a> pattern, but I haven't found it to work well.
Since commands tend to modify only a small part of an object's state,
snapshotting the rest of its data is a waste of memory. It's cheaper to
manually store only the bits you change.[oe]

Ceci semble comme une place pour le patron <a
href="http://en.wikipedia.org/wiki/Memento_pattern"
class="gof-pattern">Memento</a>, mais je n'ai pas trouvé que ça fonctionnait bien. Depuis que les commandes tendent à modifier seulement une petite partie de l'état d'un objet, Enregistrer le reste de ses données est un gaspillage de mémoire. Ca coûte moins cher de stocker manuellement seulement les bits que vous changez. 

*<a href="http://en.wikipedia.org/wiki/Persistent_data_structure">Les Structures de Données Persistantes</a>* sont une autre option. Avec celles-ci, chaque modification d'un objet en retourne un nouveau, en laissant l'original inchangé. Par une implémentation intelligente, ces nouveaux objets partage des données avec les précédents, donc c'est beaucoup moins coûteux que cloner l'objet entier.


En utilisant une structure de données persistante, chaque commande stocke une référence vers l'objet avant que la commande soit effectuée, et annuler signifie juste basculer vers l'ancien objet.

</aside>

[ob]To let the player undo a move, we keep around the last command they executed.
When they bang on Control-Z, we call that command's `undo()` method. (If they've
already undone, then it becomes "redo" and we execute the command again.)[oe]

En permettant au joueur d'annuler un coup, nous gardons la dernière commande qu'il a exécuté. Quand il tape Control-Z, Nous appelons la méthode `undo()` de la commande. (Si il a déjà annulé, alors ca  devient "refaire" et nous exécutons la commande encore.)

[ob]Supporting multiple levels of undo isn't much harder. Instead of remembering the
last command, we keep a list of commands and a reference to the "current" one.
When the player executes a command, we append it to the list and point "current"
at it.[oe]

Supporter de multiples niveaux d'annulation n'est pas beaucoup plus difficile. Au lieu de se rappeler de la dernière commande, nous gardons une liste de commandes et une référence à l'*actuelle*.

<img src="images/command-undo.png" alt="Une pile de commandes du plus ancien au plus récent. Une flèche 'current' pointe vers une commande, une flèche 'undo' pointe vers le précédent, et 'redo' pointe vers le suivant." />

[ob]When the player chooses "Undo", we undo the current command and move the current
pointer back. When they choose <span name="replay">"Redo"</span>, we advance the
pointer
and then execute that command. If they choose a new command after undoing some,
everything in the list after the current command is discarded.[oe]
	
Quand le joueur choisit "Annuler", nous annulons la commande actuelle et nous reculons le pointeur actuel. Quand Il choisit <span name="replay">"Refaire"</span> , nous avançons le pointeur et puis exécutons cette commande. Si il choisit une nouvelle commande après en avoir annulé, tout ce qui est dans la liste après la commande actuelle est retiré.

[ob]The first time I implemented this in a level editor, I felt like a genius. I was
astonished at how straightforward it was and how well it worked. It takes
discipline to make sure every data modification goes through a command, but once
you do that, the rest is easy.[oe]

La première fois que j'ai implémenté ça dans un éditeur de niveau, je me sentais comme un génie. J'étais étonné combien c'était simple et à quel point ca  marchait bien. Ca demande de la discipline pour être sûr que chaque modification de données le fait via une commande, mais une fois que vous l'avez fait, le reste est simple.

<aside name="replay">

Refaire n'est peut être pas très commun dans les jeux, par contre le *rejouer* l'est. Une implémentation naïve enregistrerait l'état du jeu entier à chaque image pour que ça soit rejoué, mais cela utiliserait trop de mémoire.

[ob]Instead, many games record the set of commands every entity performed each
frame. To replay the game, the engine just runs the normal game simulation,
executing the pre-recorded commands.[oe]

A la place, beaucoup de jeux enregistrent l'ensemble des commandes de chaque entité effectuées à chaque image. Pour rejouer le jeu, le moteur a juste à lancer la simulation normal du jeu, en exécutant le commandes pré-enregistrées.

</aside>

[ob]## Classy and Dysfunctional?[oe]

## Elégant et Dysfonctionnel ?

[ob]Earlier, I said commands are similar to first-class functions or closures, but
every example I showed here used class definitions. If you're familiar with
functional programming, you're probably wondering where the functions are.[oe]

Plus tôt, j'ai dit que les commandes sont similaires aux fonctions de premières classes ou  aux fermetures, mais chaque exemple que j'ai montré ici utilisait des définitions de classe. Si vous êtes familier avec la programmation fonctionnelle, Vous vous demandez où sont les fonctions.

[ob]I wrote the examples this way because C++ has pretty limited support for
first-class functions. Function pointers are stateless, functors are weird and
still
require defining a class, and the lambdas in C++11 are tricky to work with
because of manual memory management.[oe]

J'ai écrit les exemples de cette manière car C++ a un support assez limité des fonctions de premières classe. Les pointeurs de fonction n'ont pas d'état, les foncteurs sont étranges et nécessite en plus de définir une classe, et les lambdas en C++11 sont difficile à utiliser à cause de la gestion manuelle de la mémoire.

[ob]That's *not* to say you shouldn't use functions for the Command pattern in other
languages. If you have the luxury of a language with real closures, by all means,
use them! In <span name="some">some</span> ways, the Command pattern is a way of
emulating closures in languages that don't have them.[oe]

Ceci ne veut *pas* dire que vous ne devriez pas utiliser des fonctions pour le patron Commande dans d'autres langages. Si vous avez le luxe d'avoir un langage qui offre de vraies fermetures, par tous les moyens, utilisez les ! En quelque <span name="some">sorte</span>, le patron Commande est une manière d'émuler les fermetures dans des langages qui n'en ont pas.

<aside name="some">

[ob]I say *some* ways here because building actual classes or structures for
commands is still useful even in languages that have closures. If your command
has multiple operations (like undoable commands), mapping that to a single
function is awkward.[oe]

Je dis bien *quelque* sorte par ce que construire de vraies classes ou structures pour des commandes est tout de même utile même dans des langages qui ont des fermetures. Si votre commande a de multiples opérations (comme des commandes annulables), faire correspondre ça à une simple fonction est maladroit.

[ob]Defining an actual class with fields also helps readers easily tell what data
the command contains. Closures are a wonderfully terse way of automatically
wrapping up some state, but they can be so automatic that it's hard to see what
state they're actually holding.[oe]

Définir une vraie classe avec des champs aide les lecteurs à dire facilement quelles données la commande contient. Les fermetures sont un moyen génialement concis d'envelopper un état, mais elle peuvent être si automatique qu'il peut être difficile de voir quel état elles contiennent.

</aside>

[ob]For example, if we were building a game in JavaScript, we could create a move
unit command just like this:[oe]

Par exemple, si nous construisions un jeu en JavaScript, nous pourrions créer une commande pour déplacer une unité juste comme ceci :

    :::javascript
    function makeMoveUnitCommand(unit, x, y) {
	  // Cette fonction est l'objet commande :
      return function() {
        unit.moveTo(x, y);
      }
    }

[ob]We could add support for undo as well using a pair of closures:[oe]

Nous pourrions ajouter l'annulation aussi bien en utilisant une paire de fermetures : 

    :::javascript
    function makeMoveUnitCommand(unit, x, y) {
      var xBefore, yBefore;
      return {
        execute: function() {
          xBefore = unit.x();
          yBefore = unit.y();
          unit.moveTo(x, y);
        },
        undo: function() {
          unit.moveTo(xBefore, yBefore);
        }
      };
    }

	
[ob]If you're comfortable with a functional style, this way of doing things is
natural. If you aren't, I hope this chapter helped you along the way a bit. For
me, the usefulness of the Command pattern really shows how effective the
functional paradigm is for many problems.[oe]

Si vous êtes à l'aise avec un style fonctionnel, cette façon de faire les choses est naturelle. Si vous ne l'êtes pas, j'espère que ce chapitre vous aura aidé un peu. Pour moi, l'utilité du patron Commande montre vraiment combien le paradigme fonctionnel est efficace pour beaucoup de problèmes.


[ob]## See Also[oe]

## Voir aussi


* [ob]You may end up with a lot of different command classes. In order to make it
    easier to implement those, it's often helpful to define a concrete base
    class with a bunch of convenient high-level methods that the derived
    commands can compose to define their behavior. That turns the command's main
    `execute()` method into a <a href="subclass-sandbox.html"
    class="pattern">Subclass Sandbox</a>.[oe]

 *  Vous pourriez finir avec beaucoup de classe de commandes. En vue de rendre cela plus simple à les    implémenter, il est souvent utile de définir une classe de base concrète avec des tas de méthodes avantageuses que les commandes dérivées peuvent composer pour définir leur comportement. Cela transforme la méthode principale de la commande, `execute()`, en le patron <a href="subclass-sandbox.html"
    class="pattern">Bac à sable de sous-classe</a>.

 * [ob]In our examples, we explicitly chose which actor would handle a command. In
    some cases, especially where your object model is hierarchical, it may not
    be so cut-and-dried. An object may respond to a command, or it may decide to
    pawn it off on some subordinate object. If you do that, you've got yourself
    a <a class="gof-pattern" href="
    http://en.wikipedia.org/wiki/Chain-of-responsibility_pattern">Chain of Responsibility</a>.[oe]

 *  Dans nos exemples, nous avons choisi explicitement quel acteur devrait gérer une commande. Dans quelques cas, spécialement où votre modèle objet est hiérarchique, ca  pourrait ne pas être si simple. Un objet pourrait répondre à une commande, ou il pourrait décider de déléguer cela à un objet subordonné. Si vous faites cela, vous avez le patron <a class="gof-pattern" href="
    http://en.wikipedia.org/wiki/Chain-of-responsibility_pattern">Chaîne de Responsabilité</a>.

 * [ob]Some commands are stateless chunks of pure behavior like the `JumpCommand`
    in the first example. In cases like that, having <span
    name="singleton">more</span> than one instance of that class wastes memory
    since all instances are equivalent. The <a class="gof-pattern"
    href="flyweight.html">Flyweight</a> pattern addresses that.
[oe]

 *  Quelques commandes sont des morceaux sans état de comportement pur comme `JumpCommand` dans le premier exemple. Dans des cas comme celui-ci, avoir <span
    name="singleton">plus</span> d'une instance de cette classe gaspille de la mémoire car toutes les instances sont équivalentes. Le patron <a class="gof-pattern"
    href="flyweight.html">Poids léger</a> traite cela. 

    <aside name="singleton">
    [ob]You could make it a <a href="singleton.html class="gof-pattern">Singleton</a> too, but friends don't let friend create singletons.[oe]
	
	Vous pourriez en faire un <a href="singleton.html" class="gof-pattern">Singleton</a> aussi, mais les amis ne permettent pas aux amis de créer des singletons.

    </aside>