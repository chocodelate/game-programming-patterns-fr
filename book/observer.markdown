^title Observer
^section Design Patterns Revisited

[ob]You can't throw a rock at a computer without hitting an application built using
the <span name="devised">[Model-View-Controller][MVC]</span> architecture, and
underlying that is the Observer pattern. Observer is so pervasive that Java put
it in its core library ([`java.util.Observer`][java]) and C# baked it right into
the *language* (the [`event`][event] keyword).[oe]

Vous ne pouvez pas jeter une pierre à un ordinateur sans percuter une application construite en utilisant l'architecture <span name="devised">[Modèle-Vue-Controlleur][MVC]</span>, et que ce soit basé sur le patron Observateur. Observateur est si largement répandu que Java l'a mis dans sa bibliothèque core ([`java.util.Observer`][java]) et C# l'a intégré dans le *langage* (le mot-clé [`event`][event]).

[MVC]: http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[java]: http://docs.oracle.com/javase/7/docs/api/java/util/Observer.html
[event]: http://msdn.microsoft.com/en-us/library/8627sbea.aspx

<aside name="devised">

[ob]Like so many things in software, MVC was invented by Smalltalkers in the
seventies. Lispers probably claim they came up with it in the sixties but didn't
bother writing it down.[oe]

Comme tant de choses dans le logiciel, MVC a été inventé par les Smalltalkeurs dans les années 70. Des lispeurs revendiquent son invention dans les années 60 mais ils ne sont pas donnés la peine de le coucher sur le papier.

</aside>

[ob]Observer is one of the most widely used and widely known of the original Gang of
Four patterns, but the game development world can be strangely cloistered at
times, so maybe this is all news to you. In case you haven't left the abbey in a
while, let me walk you through a motivating example.[oe]

Observateur est un des patrons du Gang of Four les plus largement utilisés et largement connus, mais le monde du développement de jeu  peut être étrangement fermé parfois, alors peut-être que ceci est tout nouveau pour vous. Dans le cas où vous n'avez pas quitté le couvent depuis un moment, permettez moi de vous le faire découvrir au travers d'un exemple motivant.

## Succès déverrouillé

[ob]Say we're adding an <span name="weasel">achievements</span> system to our game.
It will feature dozens of different badges players can earn for completing
specific milestones like "Kill 100 Monkey Demons", "Fall off a Bridge", or
"Complete a Level Wielding Only a Dead Weasel".[oe]

Disons que nous sommes en train d'ajouter un système de <span name="weasel">succès</span> à notre jeu. Il comprendra une douzaine de badges différents que les joueurs peuvent remporter pour compléter un jalon spécifique comme  "Tuer 100 singes démons", "Faire tomber un pont", ou "Compléter un niveau en ne tuant qu'une seule belette".

<aside name="weasel">

<img src="images/observer-weasel-wielder.png" width="240" alt="Achievement: Weasel Wielder" />

[ob]I swear I had no double meaning in mind when I drew this.[oe]

Je jure que je n'ai eu aucun double sens à l'esprit quand j'ai dessiné ceci.

</aside>


[ob]This is tricky to implement cleanly since we have such a wide range of
achievements that are unlocked by all sorts of different behaviors. If we aren't
careful, tendrils of our achievement system will twine their way through every
dark corner of our codebase. Sure, "Fall off a Bridge" is somehow tied to the
<span name="physics">physics engine</span>, but do we really want to see a call
to `unlockFallOffBridge()` right in the middle of the linear algebra in our
collision resolution algorithm?[oe]



Ceci est difficile à implémenter proprement car nous avons un très grand nombre de succès déverrouillés par toutes sortes de comportements différents. Si nous ne sommes pas prudent, des mèches de notre système de succès s'enrouleront à leur manière à travers les coins sombres du codebase. Bien sûr, "Faire tomber un pont" est quelque chose qui est attaché au <span name="physics">moteur physique</span>, mais voulons nous vraiment voir un appel à `unlockFallOffBridge()` en plein milieu de l'algèbre linéaire dans notre algorithme de résolution de collision ?


<aside name="physics">

[ob]This is a rhetorical question. No self-respecting physics programmer would ever
let us sully their beautiful mathematics with something as pedestrian as
*gameplay*.[oe]

Ceci est une question rhétorique. Aucun programmeur physique qui se respecte ne voudrait nous permettre de souiller leurs belles mathématiques avec quelque chose d'aussi terre à terre que le *gameplay*.

</aside>

[ob]What we'd like, as always, is to have all the code concerned with one facet of
the game nicely lumped in one place. The challenge is that achievements are
triggered by a bunch of different aspects of gameplay. How can that work without
coupling the achievement code to all of them?[oe]

Ce que nous aimerions, comme toujours, serait d'avoir tout le code concerné par une facette du jeu 
regroupé dans un endroit. Le challenge est que les succès sont déclenchés par des tas aspects différents du gameplay. Comment ceci peut fonctionner sans les coupler au code des succès ?

[ob]That's what the observer pattern is for. It lets one piece of code announce that
something interesting happened *without actually caring who receives the
notification*.[oe]

C'est ce que résout le patron observateur. Il permet à un morceau de code d'annoncer que quelque chose d'intéressant s'est produit *sans vraiment se soucier de qui reçoit la notification*.

[ob]For example, we've got some physics code that handles gravity and tracks which
bodies are relaxing on nice flat surfaces and which are plummeting toward sure
demise. To implement the "Fall off a Bridge" badge, we could just jam the
achievement code right in there, but that's a mess. Instead, we can just do:[oe]

Par exemple, nous avons du code physique qui gère la gravité et trace quel corps se détend sur des surfaces plates agréables et ceux qui tombent avec une mort certaine. Pour implémenter l'insigne "Faire tomber un pont", nous pourrions juste foutre le code de succès à cet endroit là, mais c'est un peu bordélique. A la place, nous pouvons  faire :

^code physics-update

[ob]<span name="subtle">All</span> it does is say, "Uh, I don't know if anyone
cares, but this thing just fell. Do with that as you will."[oe]

<span name="subtle">Tout</span> ce qu'il fait est de dire, "Je ne sais pas si quelqu'un s'en soucie, mais cette chose est en train de tomber. Faites ce que vous voulez avec ça."

<aside name="subtle">

[ob]The physics engine does have to decide what notifications to send, so it isn't
entirely decoupled. But in architecture, we're most often trying to make systems
*better*, not *perfect*.[oe]

Le moteur physique a à décider quelles notifications envoyer, donc ce n'est pas entièrement découplé. Mais en architecture, nous essayons plus souvent de rendre les systèmes *meilleurs* et non *parfaits*.

</aside>

[ob]The achievement system registers itself so that whenever the physics code sends
a notification, the achievement system receives it. It can then check to see if
the falling body is our less-than-graceful hero, and if his perch prior to this
new, unpleasant encounter with classical mechanics was a bridge. If so, it
unlocks the proper achievement with associated fireworks and fanfare, and it
does all of this with no involvement from the physics code.[oe]

Le système de succès enregistre lui-même  pour qu'à chaque fois que le code physique envoie une notification, le système de notification la reçoit. Il peut alors vérifier si le corps qui tombe est notre disgracieux de héros, et si son perchoir avant cette nouvelle FIXME, déplaisante rencontre avec la mécanique classique était un pont. Si c'était le cas, il débloque le bon succès avec une feu d'artifice et en fanfare, et il fait tout cela sans aucune implication avec le code physique.

[ob]<span name="tear">In fact</span>, we can change the set of achievements or tear
out the entire achievement system without touching a line of the physics engine.
It will still send out its notifications, oblivious to the fact that nothing is
receiving them anymore.[oe]

<span name="tear">En fait</span>, nous pouvons chanqer l'ensemble des succès ou détacher le système de succès entier sans toucher une ligne du moteur physique. Il enverra ses notifications inconscient du fait que plus personne ne les reçoit.

<aside name="tear">

[ob]Of course, if we *permanently* remove achievements and nothing else ever
listens to the physics engine's notifications, we may as well remove the
notification code too. But during the game's evolution, it's nice to have this
flexibility.[oe]

Bien sûr, si nous effaçons *en permanence* des succès et personne n'écoute les notifications du moteur physique, nous pourrions aussi bien effaçer le code de la notification également. Mais durant l'évolution du jeu, c'est bien d'avoir cette flexibilité.

</aside>

## Comment ça marche

[ob]If you don't already know how to implement the pattern, you could probably
guess from the previous description, but to keep things easy on you, I'll walk
through it quickly.[oe]

Si vous ne savez pas déjà comment implémenter le patron, vous pourriez problablement l'avoir deviné à partir de la description précédente, mais pour garder les choses simples, je vais le parcourir rapidement.

[ob][oe]

### L'observateur

[ob]We'll start with the nosy class that wants to know when another object does
something interesting. These inquisitive objects are defined by this interface:[oe]

Nous démarrerons avec la classe curieuse qui veut savoir quand un autre objet fait quelque chose d'intéressant. Ces objets curieux sont définis par cette interface :

<span name="signature"></span>

^code observer

<aside name="signature">

[ob]The parameters to `onNotify()` are up to you. That's why this is the Observer
*pattern* and not the Observer "ready-made code you can paste into your game".
Typical parameters are the object that sent the notification and a generic
"data" parameter you stuff other details into.[oe]

Les paramètres de `onNotify()` dépendent de vous. C'est pourquoi on parle du patron Observateur et pas de l'observateur "code tout prêt que vous pouvez coller dans votre jeu". Les paramètres typiques sont l'objet qui a envoyé la notification et un paramètre de *données* génériques de votre cru contenant d'autres détails.

[ob]If you're coding in a language with generics or templates, you'll probably use
them here, but it's also fine to tailor them to your specific use case. Here,
I'm just hardcoding it to take a game entity and an enum that describes what
happened.[oe]

Si vous codez dans un langaqe avec des génériques ou des modèles, vous les utiliserez probablement ici, mais il est bien aussi de les façonner pour votre cas d'utilisation spécifique. Ici, j'ai juste codé ça en dur pour prendre une entité du jeu et une énumération qui décrivent ce qui est arrivé.

</aside>

[ob]Any concrete class that implements this becomes an observer. In our example,
that's the achievement system, so we'd have something like so:[oe]

Toute classe concrète qui implante ceci devient un observateur. Dans notre exemple, c'est le système de succès, donc nous devrions avoir quelque chose comme :

^code achievement-observer

### Le sujet

[ob]The notification method is invoked by the object being observed. In Gang of Four
parlance, that object is called the "subject". It has two jobs. First, it holds
the list of observers that are waiting oh-so-patiently for a missive from it:[oe]

La méthode de notification est invoquée par l'objet observé. Dans le jargon du Gang of Four, cet objet est appelé le "sujet". Il a deux fonctions. Premièrement, il garde la liste des observateurs qui attendent si impatiemment une missive de lui :

<span name="stl"></span>

^code subject-list

<aside name="stl">

[ob]In real code, you would use a dynamically-sized collection instead of a dumb
array. I'm sticking with the basics here for people coming from other languages
who don't know C++'s standard library.[oe]

Dans du vrai code, vous devriez utiliser une collection dynamiquement dimensionnée au lieu d'un simple tableau. Je m'en tiens aux bases ici pour des gens venant d'autres langages qui ne connaissent pas la bibliothèque standard du C++

</aside>

[ob]The important bit is that the subject exposes a *public* API for modifying that
list:[oe]

L'important est que le sujet expose une API publique pour modifier cette liste :

^code subject-register

[ob]That allows outside code to control who receives notifications. The subject
communicates with the observers, but it isn't *coupled* to them. In our example,
no line of physics code will mention achievements. Yet, it can still talk to the
achievements system. That's the clever part about this pattern.[oe]

Cela autorise du code extérieur à controller qui reçoit des notifications. Le sujet communique avec les observateurs, mais il n'est pas *couplé* à eux. Dans notre exemple, aucune ligne de code physique ne mentionnera des succès. Pourtant, il peut quand même parler au système de succès. C'est la partie intelligente de ce patron.

[ob]It's also important that the subject has a *list* of observers instead of a
single one. It makes sure that observers aren't implicitly coupled to *each
other*. For example, say the audio engine also observes the fall event so that
it can play an appropriate sound. If the subject only supported one observer,
when the audio engine registered itself, that would *un*-register the
achievements system.[oe]

Il est aussi important que le sujet ait une liste d'observateurs plutôt qu'un seul. Cela nous assure que ces observateurs ne sont pas implicitement couplés *les uns avec les autres*. Par exemple, disons que le moteur de son observe aussi l'évènement de chute pour qu'il puisse jouer un son approprié. Si le sujet supportait seulement un observateur, quand le moteur de son s'enregistre, cela désabonnerait le système de succès.

[ob]That means those two systems would interfere with each other -- and in a
particularly nasty way, since the second would disable the first. Supporting a
list of observers ensures that each observer is treated independently from the
others. As far as they know, each is the only thing in the world with eyes on
the subject.[oe]

Cela veut dire que ces deux systèmes devrait interférer l'un avec l'autre -- et en particulier d'une mauvaise manière, car le second devrait désactiver le premier. Supporter une liste d'observateurs assure que chaque observateur est traité indépendamment des autres. Pour eux, chacun est le seul a voir les yeux rivés sur le sujet.

[ob]The other job of the subject is sending notifications:[oe]

L'autre fonctions du sujet est d'envoyer des notifications :

<span name="concurrent"></span>

^code subject-notify

<aside name="concurrent">

[ob]Note that this code assumes observers don't modify the list in their `onNotify()`
methods. A more robust implementation would either prevent or gracefully handle
concurrent modification like that.
[oe]

Notez que ce code suppose que les observateurs ne modifient pas la liste dans leur méthode `onNotify()`. Une implémentation plus robuste devrait soit empécher ou gracieusement gérer des modifications concurrentes comme cela.

</aside>

[ob][oe]

### Une physique observable

[ob]Now, we just need to hook all of this into the physics engine so that it can send
notifications and the achievement system can wire itself up to receive them.
We'll stay close to the original *Design Patterns* recipe and <span
name="event">inherit</span> `Subject`:[oe]

Maintenant, nous avons juste besoin d'accrocher tout ça au moteur physique pour qu'il puisse envoyer des notifications et que le système de succès puisse se relier pour les recevoir. Nous resterons près de la recette originale de *Design Patterns* et <span
name="event">héritons</span> de `Subject` :

^code physics-inherit

[ob]This lets us make `notify()` in `Subject` protected. That way the derived
physics engine class can call it to send notifications, but code outside of it
cannot. Meanwhile, `addObserver()` and `removeObserver()` are public, so
anything that can get to the physics system can observe it.[oe]

Ceci nous permet de rendre `notify()` protégé in `Subject`. De cette manière la classe du moteur physique dérivée peut l'appeler pour envoyer des notifications, mais le code extérieur ne le peut. Pendant ce temps, `addObserver()` et `removeObserver()` sont publiques, donc n'importe quoi qui peut avoir accès au moteur physique peut l'observer.

<aside name="event">

[ob]In real code, I would avoid using inheritance here. Instead, I'd make `Physics`
*have* an instance of `Subject`. Instead of observing the physics engine itself,
the subject would be a separate "falling event" object. Observers could register
themselves using something like:[oe]

Dans du vrai code, J'empêcherais l'utilisation de l'héritage ici. A la place, je ferais en sorte que `Physics` ait une instance de `Subject`. Au lieu d'observer le moteur physique lui-même, le sujet devrait être un objet "évènement de chute" séparé. Les observateurs pourraient s'enregistrer eux-mêmes en utilisant quelque chose comme :

^code physics-event

[ob]To me, this is the difference between "observer" systems and "event" systems.
With the former, you observe *the thing that did something interesting*. With
the latter, you observe an object that represents *the interesting thing that
happened*.[oe]

Pour moi, ceci est la différence entre les systèmes d'"observateur" et les systèmes d'"évènement". Avec l'ancien, vous observiez *la chose qui a fait quelque chose d'intéressant*. Avec le nouveau, vous observez un objet qui représente *la chose intéressante qui est arrivé*.

</aside>

[ob]Now, when the physics engine does something noteworthy, it calls `notify()`
like in the motivating example before. That walks the observer list and gives
them all the heads up.[oe]

Maintenant, quand le moteur physique fait quelque chose de notable, il appelle `notify()` comme dans l'exemple motivant d'avant. Cela parcourt la liste des observateurs et les met tous au courant.

<img src="images/observer-list.png" alt="A Subject containing a list of Observer pointers. The first two point to Achievements and Audio." />

[ob]Pretty simple, right? Just one class that maintains a list of pointers to
instances of some interface. It's hard to believe that something so
straightforward is the communication backbone of countless programs and app
frameworks.[oe]

Assez simple, non ? Juste une classe qui maintient une liste de pointeurs vers des instances de certaines interfaces. C'est dur de croire que quelque chose de si simple est la colonne vertébrale de communication d'un nombre infini de programmes et de frameworks d'application.

[ob]But the Observer pattern isn't without its detractors. When I've asked other
game programmers what
they think about this pattern, they bring up a few complaints. Let's see what we
can do to address them, if anything.[oe]

Mais le patron observateur n'est pas sans ses détracteurs. Quand j'ai demandé à d'autres programmeurs de jeu ce qu'ils pensaient de ce patron, ils m'ont rapporté quelque plaintes. Voyons ce que nous pouvons faire pour les résoudre, si tant est qu'on le puisse.

## "C'est trop lent"

[ob]I hear this a lot, often from programmers who don't actually know the details of
the pattern. They have a default assumption that anything that smells like a
"design pattern" must involve piles of classes and indirection and other
creative ways of squandering CPU cycles.[oe]

J'entends cela beaucoup, souvent de la part des programmeurs qui ne connaissent pas vraiment les détails du patron. Ils font l'hypothèse par défaut que tout cela sent le " patron de conception " doit impliquer des piles de classes et d'indirection et autre manière créative de gaspillage de cycle CPU.

[ob]The Observer pattern gets a particularly bad rap here because it's been known to
hang around with some shady characters named "events", <span
name="names">"messages"</span>, and even "data binding". Some of those systems
*can* be slow (often deliberately, and for good reason). They involve things
like queuing or doing dynamic allocation for each notification.[oe]

Le patron Observateur prend particulièrement un mauvais coup ici, par ce qu'il est connu pour tourner en rond avec des personnages sombre nommé "événement", "messages", et même "Data binding". Certains de ses systèmes *peut* être lent (souvent délibérément,et pour une bonne raison). Ils impliquent des choses comme des files d'attentes ou de faire de l'allocation dynamique pour chaque notifications.

<aside name="names">

[ob]This is why I think documenting patterns is important. When we get fuzzy about
terminology, we lose the ability to communicate clearly and succinctly. You say,
"Observer", and someone hears "Events" or "Messaging" because either no one
bothered to write down the difference or they didn't happen to read it.[oe]

Ceci est pourquoi je pense que documenter les patrons est important. Quand nous sommes troublé par la terminologie, nous perdons la capacité de communiquer clairement et succinctement. Vous me direz, "Observateur", et quelqu'un entend "évènement" ou "messagerie" par ce que aucun n'a pris la peine d'écrire la différence ou qu'il ne sont pas arrivé à la lire.

[ob]That's what I'm trying to do with this book. To cover my bases, I've got a
chapter on events and messages too: <a href="event-queue.html"
class="pattern">Event Queue</a>.[oe]

C'est ce que j'essaie de faire dans ce livre. Pour couvrir mes bases, j'ai un chapitre sur les évènements et les messages aussi: <a href="event-queue.html" class="pattern">File d'évènement</a>.

</aside>

[ob]But, now that you've seen how the pattern is actually implemented, you know that
isn't the case. Sending a notification is simply walking a list and calling some
virtual methods. Granted, it's a *bit* slower than a statically dispatched
call, but that cost is negligible in all but the most performance-critical
code.[oe]

Mais maintenant que vous avez vu comment le patron est vraiment implanté, vous savez que ce n'est pas le cas. Envoyer une notification est simplement parcourir une liste et simplement appeler des méthodes virtuelles. D'accord c'est *un petit peu* plus lent que un appel envoyé statiquement, mais ce coût est négligeable partout sauf dans le code ou la performance est critique.

[ob]I find this pattern fits best outside of hot code paths anyway, so you can
usually afford the dynamic dispatch. Aside from that, there's virtually no
overhead. We aren't allocating objects for messages. There's no queueing. It's
just an indirection over a synchronous method call.[oe]

Je trouve que ce patron s'adapte bien en dehors des cas du code critique, donc vous pouvez habituellement vous accorder l'envoi dynamique. D'un autre côté il y a aucun surcout virtuellement parlant. Nous n'allouons pas des objets pour des messages. Il n'y a aucunes file d'attente. C'est juste une indirection, sur un appel de méthode synchrone. 

### C'est trop *rapide* ?

[ob]In fact, you have to be careful because the Observer pattern *is* synchronous.
The subject invokes its observers directly, which means it doesn't resume its
own work until all of the observers have returned from their notification
methods. A slow observer can block a subject.[oe]

En fait, vous devez être prudent par ce que le patron Observateur *est* synchrone. Le sujet invoque ses observateurs directement, ce qui veut dire qu'il ne reprend pas son propre travail jusqu'à ce que tout les observateurs soient revenu de leur méthodes de notifications. Un observateur lent peu bloquer un sujet. 

[ob]This sounds scary, but in practice, it's not the end of the world. It's just
something you have to be aware of. UI programmers -- who've been doing
event-based programming like this for ages -- have a time-worn motto for this:
"stay off the UI thread".[oe]

Ceci semble effrayant, mais en pratique, c'est pas la fin du monde. C'est quelque chose dont vous devez être conscient. Les programmeurs UI -- qui ont font de la programmation évènementielle comme ceci depuis des lustres -- ont une devise éprouvée pour ceci : "rester à l'écart du thread UI".

[ob]If you're responding to an event synchronously, you need to finish and return
control as quickly as possible so that the UI doesn't lock up. When you have
slow work to do, push it onto another thread or a work queue.[oe]

Si vous êtes en train de répondre à un évènement de manière synchrone, vous devez terminer et retourner le control aussi rapidement que possible pour l'UI ne se bloque pas. Quand vous avez un travail lent à réaliser, envoyez le à un autre thread ou une file de travaux.

[ob]You do have to be careful mixing observers with threading and explicit locks,
though. If an observer tries to grab a lock that the subject has, you can
deadlock the game. In a highly threaded engine, you may be better off with
asynchronous communication using an <a href="event-queue.html"
class="pattern">Event Queue</a>.[oe]

Vous devez être prudent en mélangeant les observateurs avec le threading et les verrous explicites, malgré tout. Si un observateur essaie de récupérer un vérrou que le sujet a, vous pouvez causer un deadlock dans le jeu. Dans un moteur lourdement threadé, vous pourriez être plus à l'aise avec la communication asynchrone en utilisant une <a href="event-queue.html" class="pattern">File d'évènement</a>.

## "Ca fait trop d'allocation dynamique"

[ob]Whole tribes of the programmer clan -- including many game developers -- have
moved onto garbage collected languages, and dynamic allocation isn't the boogie
man that it used to be. But for performance-critical software like games, memory
allocation still matters, even in managed languages. <span
name="fragment">Dynamic</span> allocation takes time, as does reclaiming memory,
even if it happens automatically.[oe]

Des hordes de clans de programmeurs -- incluant beaucoup de développeurs de jeu -- ont migrés vers des langages à mémoire managée, et l'allocation dynamique n'est pas le croque-mitaine qu'il est habitué à être. Mais pour des logiciels dont la performance est critique tels que des jeux, l'allocation mémoire importe, même dans des langages vmanagés. L'allocation <span name="fragment">dynamique</span> prend du temps, en réclamant de la mémoire, même si cela arrive automatiquement.

<aside name="fragment">

[ob]Many game developers are less worried about allocation and more worried about
*fragmentation.* When your game needs to run continuously for days without
crashing in order to get certified, an increasingly fragmented heap can prevent
you from shipping.[oe]

Beaucoup de développeurs de jeu sont moins inquiétés par l'allocation que par la *fragmentation*. Quand votre jeu a besoin de tourner continuellement pendant des jours sans planter pour être certifié, un tas de plus en plus fragmenté peut vous empêcher de vendre le jeu.

[ob]The <a href="object-pool.html" class="pattern">Object Pool</a> chapter goes into
more detail about this and a common technique for avoiding it.[oe]

Le chapitre sur la <a href="object-pool.html" class="pattern">Piscine d'objet</a> étudie en détail ceci et une technique commune pour l'éviter.

</aside>

[ob]In the example code before, I used a fixed array because I'm trying to keep
things dead simple. In real implementations, the observer list is almost always
a dynamically allocated collection that grows and shrinks as observers are
added and removed. That memory churn spooks some people.[oe]

Dans l'exemple de code d'avant, j'ai utilisé un tableau de taille fixe car j'essaie de garder les choses le plus simple possible. Dans de vraies implémentations, la liste d'observateurs est presque toujours une collection dynamiquement allouée qui grandit et rétrécit quand des observateurs sont ajoutés et effacés. Ce bidon de mémoire effraie certaines personnes.

[ob]Of course, the first thing to notice is that it only allocates memory when
observers are being wired up. *Sending* a notification requires no memory
allocation whatsoever -- it's just a method call. If you hook up your observers
at the start of the game and don't mess with them much, the amount of allocation
is minimal.[oe]

Bien sûr, la première chose à noter est que cela alloue seulement de la mémoire quand les observateurs sont câblés. *Envoyer* une notification ne nécessite aucune allocation mémoire dans tous les cas -- c'est juste un appel de méthode. Si vous branchez vos observateurs au démarrage du jeu et ne semez pas trop le désordre avec eux, le montant d'allocation est minimal.

[ob]If it's still a problem, though, I'll walk through a way to implement adding and
removing observers without any dynamic allocation at all.[oe]

Si c'est encore un problème, malgré cela, je vais vous montrer une manière d'implémenter l'ajout et l'effacement d'observateurs sans aucune allocation dynamique.

### Observateurs chaînés

[ob]In the code we've seen so far, `Subject` owns a list of pointers to each
`Observer` watching it. The `Observer` class itself has no reference to this
list. It's just a pure virtual interface. Interfaces are preferred over
concrete, stateful classes, so that's generally a good thing.[oe]

Dans le code que nous avons vu jusqu'ici, `Subject` possède une liste de pointeur vers chaque `Observer` le regardant. La classe `Observer` elle-même n'a aucune référence à cette liste. C'est juste une interface virtuelle pure. Les interfaces sont préférées au concret, des classes dynamiques, donc c'est généralement une bonne chose.

[ob]But if we *are* willing to put a bit of state in `Observer`, we can solve our
allocation problem by threading the subject's list *through the observers
themselves*. Instead of the subject having a separate collection of pointers,
the observer objects become nodes in a linked list:[oe]

Mais si vous *êtes* prêt à mettre un peu d'état dans `Observer`, nous pouvons résoudre notre problème d'allocation en enfilant la liste des sujets *sur les observateurs eux-mêmes*. Au lieu du sujet ayant une collection de pointeur à part, les objets observateur deviennent des n\oe uds dans un liste chainée :

<img src="images/observer-linked.png" alt="Une liste chaînée d'observateurs. Chacun a un champs next_ pointant vers le suivant. Un Subjet a un head_ pointant vers le premier observateur."/>

[ob]To implement this, first we'll get rid of the array in `Subject` and replace it
with a pointer to the head of the list of observers:[oe]

Pour implémenter ceci, premièrement nous nous débarasseront du tableau dans `Subject` et nous le remplacerons par un pointeur vers la tête de la liste des observateurs :

^code linked-subject

[ob]Then we'll extend `Observer` with a pointer to the next observer in the list:[oe]

Puis nous étendrons `Observer` avec un pointeur vers le prochain observateur dans la liste :

^code linked-observer

[ob]We're also making `Subject` a friend class here. The subject owns the API for adding
and removing observers, but the list it will be managing is now inside the
`Observer` class itself. The simplest way to give it the ability to poke at that
list is by making it a friend.[oe]

Nous faisons de `Subject` une classe amie ici. le sujet possède l'API pour ajouter et effacer des observateurs, mais la liste qu'il gèrera est désormais dans la classe `Observer` elle-même. La manière la plus simple de lui donner la capacité d'accèder à cette liste est d'en faire une amie.

[ob]Registering a new observer is just wiring it into the list. We'll take the easy
option and insert it at the front:[oe]

Enregistrer un nouvel observateur est juste le relier à la liste. Nous prendrons l'option facile et l'insèrerons au début :

^code linked-add

[ob]The other option is to add it to the end of the linked list. Doing that adds a
bit more complexity. `Subject` has to either walk the list to find the end or
keep a separate `tail_` pointer that always points to the last node.[oe]

L'autre option est de l'ajouter à la fin de la liste chainée. Faire cela ajoute un petit peu plus de complexité. `Subject` doit parcourir la liste pour trouver la fin ou garder un pointeur `tail_` qui pointe toujours vers le dernier noeud.

[ob]Adding it to the front of the list is simpler, but does have one side effect.
When we walk the list to send a notification to every observer, the most
*recently* registered observer gets notified *first*. So if you register
observers A, B, and C, in that order, they will receive notifications in C, B, A
order.[oe]

L'ajouter au début de la liste est plus simple, mais a un effet secondaire. Quand nous parcourons la liste pour envoyer une notification à chaque observateur, le plus *récemment* enregistré est notifié en *premier*. Donc si vous enregistrez les observateurs A, B, et C, dans cet ordre, ils recevront des notifications dans l'ordre C, B, A.

[ob]In theory, this doesn't matter one way or the other. It's a tenet of good
observer discipline that two observers observing the same subject should have no
ordering dependencies relative to each other. If the ordering *does* matter, it
means those two observers have some subtle coupling that could end up biting
you.[oe]

En théorie, l'ordre n'importe pas. C'est un principe d'une bonne discipline de l'observateur que deux observateurs observant le même sujet ne devrait avoir aucune dépendance dans l'ordre de l'un par rapport à l'autre. Si l'ordre *est* important, cela signifie que ces deux observateurs ont un couplage subtil qui pourrait finir par se retourner contre vous.

[ob]Let's get removal working:[oe]

Ecrivons le code d'effacement :

<span name="remove"></span>

^code linked-remove

<aside name="remove">

[ob]Removing a node from a linked list usually requires a bit of ugly special case
handling for removing the very first node, like you see here. There's a more
elegant solution using a pointer to a pointer.[oe]

En Effaçant un noeud de la liste chainée nécessite habituellement un peu de gestion hardue du cas spécial de l'effacement du tout premier noeud, comme vous pouvez le voir ici. Il y a une solution plus élégante en utilisant un pointeur de pointeur.

[ob]I didn't do that here because it confuses at least half the people I show it to.
It's a worthwhile exercise for you to do, though: It helps you really think in
terms of pointers.[oe]

Je ne l'ai pas fait ici car ça rend confus la moitié des gens à qui je la montre. C'est un très bon exercice de le faire pour vous, néanmoins : ça vous aide vraiment à penser en terme de pointeurs.

</aside>

[ob]Because we have a singly linked list, we have to walk it to find the observer
we're removing. We'd have to do the same thing if we were using a regular array
for that matter. If we use a *doubly* linked list, where each observer has a
pointer to both the observer after it and before it, we can remove an observer
in constant time. If this were real code, I'd do that.[oe]

Comme nous avons un liste simplement chainée, nous avons à la parcourir pour trouver l'observateur que nous effaçons. Nous aurions à faire la même chose si nous utilisions un tableau classique à ce but. Si nous utilisons une liste *doublement* chainée, dans laquelle chaque observateur a un pointeur vers à la fois l'observateur après et avant lui, nous pouvons effacer un observateur en temps constant. Si c'était du vrai code, je le ferais.

[ob]The <span name="chain">only</span> thing left to do is send a notification.
That's as simple as walking the list:[oe]

La <span name="chain">seule</span> chose qui reste à faire est d'envoyer une notification. C'est aussi simple que de parcourir la liste :

^code linked-notify

<aside name="chain">

[ob]Here, we walk the entire list and notify every single observer in it. This
ensures that all of the observers get equal priority and are independent of each
other.[oe]

Ici, nous parcourons la liste entière et notifions chaque observateur contenu dans celle-ci. Ceci nous assure que tous les observateurs ont une priorité égale et sont indépendant les uns des autres.

[ob]We could tweak this such that when an observer is notified, it can return a flag
indicating whether the subject should keep walking the list or stop. If you do
that, you're pretty close to having the <a href="http://en.wikipedia.org/wiki
/Chain-of-responsibility_pattern" class="gof-pattern">Chain of
Responsibility</a> pattern.[oe]

Nous pourrions adapter ceci tel que quand un observateur est notifié, il peut retourner un flag indiquant si le sujet devrait continuer à parcourir la liste ou non. Si vous faites cela, vous êtes pas loin d'avoir le patron <a href="http://en.wikipedia.org/wiki
/Chain-of-responsibility_pattern" class="gof-pattern">Chaine de Responsabilité</a>.

</aside>

[ob]Not too bad, right? A subject can have as many observers as it wants, without a
single whiff of dynamic memory. Registering and unregistering is as fast as it
was with a simple array. We have sacrificed one small feature, though.[oe]

Pas si mauvais, non ? Un sujet peut avoir autant d'observateurs qu'il veut, sans une simple bouffée d'allocation dynamique. Enregistrer et dé-enregistrer est aussi rapide que ça l'était avec un simple tableau. Nous avons sacrifié une petite fonctionnalité cependant.

[ob]Since we are using the observer object itself as a list node, that implies it
can only be part of one subject's observer list. In other words, an observer can
only observe a single subject at a time. In a more traditional implementation
where each subject has its own independent list, an observer can be in more than
one of them simultaneously.[oe]

Comme nous utilisons l'objet observateur lui-même comme un noeud de liste, cela implique qu'il peut appartenir à une seul liste d'observateur d'un sujet. En d'autres termes, un observateur peut observer seulement un sujet à la fois. Dans une implémentation plus traditionnelle où chaque sujet a sa propre liste indépendante, un observateur peut être dans plus qu'une liste simultanément.

[ob]You may be able to live with that limitation. I find it more common for a
*subject* to have multiple *observers* than vice versa. If it *is* a problem for
you, there is another more complex solution you can use that still doesn't
require dynamic allocation. It's too long to cram into this chapter, but I'll
sketch it out and let you fill in the blanks...[oe]

Vous pourriez être capable de faire avec cette limitation. Je trouve cela plus commun pour un *sujet* d'avoir de multiples *observateurs* que l'inverse. Si *c'est* un problème pour vous, il y a une autre solution plus complexe que vous pouvez utiliser qui ne nécessite pas d'allocation dynamique. Ce serait trop long de développer ça dans le chapitre, mais je vais l'esquisser et vous permettre de combler les blancs...

### Une piscine de noeuds de liste

[ob]Like before, each subject will have a linked list of observers. However, those
list nodes won't be the observer objects themselves. Instead, they'll be
separate little "list <span name="intrusive">node</span>" objects that contain a
pointer to the observer and then a pointer to the next node in the list.[oe]

Comme avant, chaque sujet aura une liste chainée d'observateurs. Cependant, ces listes de n\oe uds ne seront pas les objets observateur eux-mêmes. A la place, ils seront des objets d'une petite "liste de <span name="intrusive">noeud </span>" à part qui contient un pointeur vers l'observateur et puis un pointeur vers le noeud suivant dans la liste.

[ob][oe]

<img src="images/observer-nodes.png" alt="Une liste chainée de noeuds. Chaque noeud a un champs observer_ pointant vers un Observer, et un champ next_ pointant vers le noeud suivant dans la liste. Un champ head_ de Subject pointe vers le premier noeud." />

[ob]Since multiple nodes can all point to the same observer, that means an observer
can be in more than one subject's list at the same time. We're back to being
able to observe multiple subjects simultaneously.[oe]

Comme de multiple noeuds peuvent tous pointer vers le même observateur, cela veut dire qu'un observateur peut être dans plus qu'une liste d'un sujet à la fois. Nous revenons à la capacité d'observer de multiple sujets simultanément.

<aside name="intrusive">

[ob]Linked lists come in two flavors. In the one you learned in school, you have a
node object that contains the data. In our previous linked observer example,
that was flipped around: the *data* (in this case the observer) contained the
*node* (i.e. the `next_` pointer).[oe]

Les listes chaînées sont de deux genres. Pour la première que vous avez appris à l'école, vous avez un objet noeud qui contient les données. Dans notre exemple d'observateur chaîné précédent, c'était retourné : les *données* (l'observateur dans ce cas) contenait le *noeud* (i.e. le pointeur `next_`).

[ob]The latter style is called an "intrusive" linked list because using an object in
a list intrudes into the definition of that object itself. That makes intrusive
lists less flexible but, as we've seen, also more efficient. They're popular in
places like the Linux kernel where that trade-off makes sense.[oe]

Le dernier style est appelé une liste chaînée "intrusive" par ce qu'utiliser un objet dans un liste s'immisce dans la définition de cette objet lui-même. Cela rend les listes intrusives moins flexibles mais, comme nous l'avons vu, aussi plus efficace. Elles sont populaires en des endroits comme le noyau Linux où ce compromis fait sens.

</aside>

[ob]The way you avoid dynamic allocation is simple: since all of those nodes are the
same size and type, you pre-allocate an <a href="object-pool.html"
class="pattern">Object Pool</a> of them. That gives you a fixed-size pile of
list nodes to work with, and you can use and reuse them as you need without
having to hit an actual memory allocator.[oe]

La manière d'éviter l'allocation dynamique est simple : comme tous ces noeuds sont de la même taille et type, vous pré-allouez une <a href="object-pool.html"
class="pattern">Piscine d'Objet</a> de ceux-ci. Cela vous donne une pile de taille fixe de liste de noeuds avec laquelle vous pouvez travailler, et vous pouvez les utiliser et les réutiliser selon votre besoin sans avoir à lancer un seul allocateur de mémoire.

## Problèmes restants

[ob]I think we've banished the three boogie men used to scare people off this
pattern. As we've seen, it's simple, fast, and can be made to play nice with
memory management. But does that mean you should use observers all the time?[oe]

Je pense que nous avons bannis les trois croque-mitaines utilisés pour effrayer les gens à la vue de ce pattern. Comme nous l'avons vu, c'est simple, rapide, et peut être fait en s'accordant bien avec la gestion de la mémoire. Mais cela veut il dire que vous devriez utiliser des observateurs tout le temps ?

[ob]Now, that's a different question. Like all design patterns, the Observer pattern
isn't a cure-all. Even when implemented correctly and efficiently, it may not be
the right solution. The reason design patterns get a bad rap is because people
apply good patterns to the wrong problem and end up making things worse.[oe]

Maintenant, c'est une question différente. Comme tous les patrons de conception, le patron Observateur n'est pas une panacée. Même lorsque c'est implanté correctement et efficacement, ça pourrait ne pas être la bonne solution. La raison pour laquelle les patrons de conception ont mauvaise réputation est par ce que les gens appliquent de bons patrons au mauvais problème et ça finit par rendre les choses pires.

[ob]Two challenges remain, one technical and one at something more like the
maintainability level. We'll do the technical one first because those are always
easiest.[oe]

Deux challenges restent, un technique et un qui a plus à voir avec le niveau de maintenabilité. Nous verrons le technique en premier car ils sont toujours plus faciles.

### Détruire des sujets et des observateurs

[ob]The sample code we walked through is solid, but it
<span name="destruct">side-steps</span> an important issue: what happens when
you delete a subject or an
observer? If you carelessly call `delete` on some observer, a subject may still
have a pointer to it. That's now a dangling pointer into deallocated memory.
When that subject tries to send a notification, well... let's just say you're
not going to have a good time.[oe]

Le code exemple que nous avons parcouru est solide, mais il <span name="destruct">passe à coté</span> d'un détail important : Qu'arrive t'il quand vous supprimer un sujet ou un observateur ? Si vous appelez `delete` sans précaution sur un observateur, un sujet pourrait encore avoir un pointeur vers lui. C'est maintenant un pointeur ballant dans une mémoire dés-allouée. Quand ce sujet essaie d'envoyer une notification, et bien... permettez moi de dire que vous allez passer un sale quart d'heure.

<aside name="destruct">

[ob]Not to point fingers, but I'll note that *Design Patterns* doesn't mention this
issue at all.[oe]

C'est n'est pas pour pointer du doigt mais je noterai que *Design Patterns* ne mentionne pas du tout ce problème.

</aside>

[ob]Destroying the subject is easier since in most implementations, the observer
doesn't have any references to it. But even then, sending the subject's bits to
the memory manager's recycle bin may cause some problems. Those observers may
still be expecting to receive notifications in the future, and they don't know
that that will never happen now. They aren't observers at all, really, they just
think they are.[oe]

Détruire le sujet est plus facile depuis que dans les plupart des implémentations, l'observateur n'a aucune référence vers lui. Mais dans tous les cas, envoyer les bits du sujet à la poubelle du gestionnaire de la mémoire pourrait causer des problèmes. Ces observateurs pourrait encore s'attendre à recevoir des notifications dans le future, et ils ne savent pas que cela n'arrivera plus désormais. Ils ne sont plus des observateurs, ils croient juste qu'ils le sont.

[ob]You can deal with this in a couple of different ways. The simplest is to do what
I did and just punt on it. It's an observer's job to unregister itself from any
subjects when it gets deleted. More often than not, the observer *does* know
which subjects it's observing, so it's usually just a matter of <span
name="destructor">adding</span> a `removeObserver()` call to its destructor.[oe]

Vous pouvez traiter ceci de plusieurs manières différentes. La plus simple est de faire ce que j'ai fait et de juste de la faire à la volée. C'est le boulot d'un observateur de se dés-enregistrer lui-même de tout sujet quand il est supprimé. Plus souvent que pas, l'observateur *fait* savoir quel sujet il est en train d'observer, ainsi c'est normalement juste l'histoire <span
name="destructor">d'ajouter</span> un appel à `removeObserver()` à son destructeur.

<aside name="destructor">

[ob]As is often the case, the hard part isn't doing it, it's *remembering* to do it.[oe]

Comme c'est souvent le cas, la chose difficile n'est pas de le faire, mais de se *rappeler* de le faire.

</aside>

[ob]If you don't want to leave observers hanging when a subject gives up the ghost,
that's easy to fix. Just have the subject send one final "dying breath"
notification right before it gets destroyed. That way, any observer can receive
that and take <span name="mourn">whatever action</span> it thinks is
appropriate.[oe]

Si vous ne voulez laisser trainee des observateurs quand un sujet rend l'âme, c'est facile à corriger. Il faut juste que le sujet envoie une notification finale de "dernier souffle" juste avant qu'il soit détruit. De cette manière, tout observateur peut le recevoir et prendre <span name="mourn">la décision</span> qu'il juge appropriée.

<aside name="mourn">

[ob]Mourn, send flowers, compose elegy, etc.[oe]

Pleurez, envoyer des fleurs, composer une élégie, etc.

</aside>

[ob]People -- even those of us who've spent enough time in the company of machines
to have some of their precise nature rub off on us -- are reliably terrible at
being reliable. That's why we invented computers: they don't make the mistakes
we so often do.[oe]

Les gens -- même ceux parmi nous qui ont passé assez de temps en compagnie des machines ont une partie de leur nature qui déteint sur nous -- sont sérieusement épouvantable quand il faut être fiable. C'est pourquoi nous avons inventé les ordinateurs : ils ne font pas les erreurs que nous faisons souvent.

[ob]A safer answer is to make observers automatically unregister themselves from
every subject when they get destroyed. If you implement the logic for that once
in your base observer class, everyone using it doesn't have to remember to do it
themselves. This does add some complexity, though. It means each *observer* will
need a list of the *subjects* it's observing. You end up with pointers going in
both directions.
[oe]

Une solution plus prudente est de faire en sorte que les observateurs se désabonne automatiquement eux-mêmes de chaque sujet quand ils sont détruits. Si vous implantez la logique une seule fois dans votre classe de base d'observateur, tous ceux qui l'utilisent n'ont pas à se rappeler de le faire eux-mêmes. Ceci ajoute un peu de complexité, malgré tout. Ca veut dire que chaque *observateur* aura besoin d'une liste des *sujets* qu'il est en train d'observer. Vous vous retrouvez avec des pointeurs allant dans les deux directions à la fois.

### T'inquiète, j'ai un GC

[ob]All you cool kids with your hip modern languages with garbage collectors are
feeling pretty smug right now. Think you don't have to worry about this because
you never explicitly delete anything? Think again![oe]

Tout bon enfant que vous êtes avec vos langages modernes à la pointe avec des ramasses-miettes vous sentez assez fier de vous maintenant. Croyez vous que vous n'avez pas à vous inquiéter de ceci par ce que vous ne supprimez rien explicitement ? Réfléchissez encore !

[ob]Imagine this: you've got some UI screen that shows a bunch of stats about the
player's character like their health and stuff. When the player brings up the
screen, you instantiate a new object for it. When they close it, you just forget
about the object and let the GC clean it up.[oe]

Imaginez ceci : vous avez un écrans d'UIs qui montre un tas de statistiques sur le personnage du joueur comme sa vie et son équipement. Quand le joueur fait apparaitre l'écran, vous instanciez un nouvel objet pour ça. Quand ils le ferment, vous oubliez simplement l'objet et laissez le GC faire le nettoyage.

[ob]Every time the character takes a punch to the face (or elsewhere, I suppose), it
sends a notification. The UI screen observes that and updates the little health
bar. Great. Now what happens when the player dismisses the screen, but you don't
unregister the observer?[oe]

A chaque fois que le personnage prend un coup au visage (ou à quelconque endroit je présume), ça envoie une notification. L'écran d'UI observe ça et met à jour la petite barre de vie. Super. Maintenant qu'arrive t'il quand le joueur ferme l'écran, mais ne désabonne pas l'observateur ?

[ob]The UI isn't visible anymore, but it won't get garbage collected since the
character's observer list still has a reference to it. Every time the screen is
loaded, we add a new instance of it to that increasingly long list.[oe]

L'UI est n'est plus visible, mais ils ne sera pas garbage collecté car la liste d'observateur du personnage a encore une référence vers lui. A chaque fois que l'écran est chargé, nous ajouterons une nouvelle instance de lui à cette interminablement longue liste.

[ob]The entire time the player is playing the game, running around, and getting in
fights, the character is sending notifications that get received by *all* of
those screens. They aren't on screen, but they receive notifications and waste
CPU cycles updating invisible UI elements. If they do other things like play
sounds, you'll get noticeably wrong behavior.[oe]

Pendant tout le temps où le joueur joue au jeu, courant à droite à gauche, combattant, le personnage envoie des notifications qui sont reçues par *tous* ces écrans. Ils ne sont pas affichés, mais ils reçoivent des notifications et gaspillent des cycles CPU en mettant à jour des éléments d'UI invisibles. Si ils font d'autres choses comme jouer des sons, vous aurez des comportements erronés perceptibles.

[ob]This is such a common issue in notification systems that it has a name: the
<span name="lapsed">*lapsed listener problem*</span>. Since subjects retain
references to their listeners, you can end up with zombie UI objects lingering
in memory. The lesson here is to be disciplined about unregistration.[oe]

Ceci est un problème tellement commun dans les systèmes de notification que ça a un nom : le <span name="lapsed">*problème de l'écouteur écoulé (lapsed listener problem)*</span>. Comme les sujets retiennent des références vers leur écouteurs, vous pouvez vous retrouvez avec des objets UI zombies persistant dans la mémoire. La leçon ici est d'être discipliné sur le désabonnement.

<aside name="lapsed">

[ob]An even surer sign of its significance: it has [a Wikipedia
article](http://en.wikipedia.org/wiki/Lapsed_listener_problem).[oe]

Un signe encore plus probant de son importance : il a [un article Wikipedia](http://en.wikipedia.org/wiki/Lapsed_listener_problem).

</aside>

### Que se passe t'il ?

[ob]The other, deeper issue with the Observer pattern is a direct consequence of its
intended purpose. We use it because it helps us loosen the coupling between two
pieces of code. It lets a subject indirectly communicate with some observer
without being statically bound to it.[oe]

L'autre, profond problème avec le patron Observateur est une conséquence directe de son but visé. Nous l'utilisons car il nous aide délier le couplage entre deux morceaux de code. Il permet à un sujet de communiquer indirectement avec un observateur sans être statiquement limité à lui.

[ob]This is a real win when you're trying to reason about the subject's behavior,
and any hangers-on would be an annoying distraction. If you're poking at the
physics engine, you really don't want your editor -- or your mind -- cluttered
up with a bunch of stuff about achievements.[oe]

C'est une vraie victoire quand vous essayez de raisonner sur le comportement du sujet, et tout homme à tout faire serait une distraction ennuyeuse. Si vous poussez au moteur physique, vous ne voulez vraiment pas que votre éditeur -- ou votre esprit -- s'encombre avec un tas de chose sur les succès.

[ob]On the other hand, if your program isn't working and the bug spans some chain of
observers, reasoning about that communication flow is much more difficult. With
an explicit coupling, it's as easy as looking up the method being called. This
is child's play for your average IDE since the coupling is static.[oe]

D'autre part, si votre programme ne fonctionne pas et que le bug touche une chaine d'observateurs, raisonner sur ce flux de communication est beaucoup plus difficile. Avec un couplage explicite, c'est aussi facile que rechercher la méthode appelée. C'est un jeu d'enfant pour un IDE moyen car le couplage est statique.

[ob]But if that coupling happens through an observer list, the only way to tell who
will get notified is by seeing which observers happen to be in that list *at
runtime*. Instead of being able to *statically* reason about the communication
structure of the program, you have to reason about its *imperative, dynamic*
behavior.[oe]

Mais si ce couplage arrive par une liste d'observateur, la seule manière de dire qui sera notifié est regardant quels observateurs parviennent à être dans cette liste au *runtime*. Au lieu d'être capable de raisonner *statiquement* sur la structure de communication du programme, vous devez raisonner sur son *impératif*, son comportement *dynamique*.

[ob]My guideline for how to cope with this is pretty simple. If you often need to
think about *both* sides of some communication in order to understand a part of
the program, don't use the Observer pattern to express that linkage. Prefer
something more explicit.[oe]

Ma ligne de conduite pour faire face à ceci est assez simple. Si vous avez souvent besoin de réfléchir des deux cotés à la fois de la communication en vue de comprendre une partie du programme, n'utilisez pas le patron Observateur pour exprimer ce lien. Préférez quelque chose de plus explicite.

[ob]When you're hacking on some big program, you tend to have lumps of it that you
work on all together. We have lots of terminology for this like "separation of
concerns" and "coherence and cohesion" and "modularity", but it boils down to
"this stuff goes together and doesn't go with this other stuff".[oe]

Quand vous être en train de hacker un gros programme, vous avez tendance à avoir des morceaux de choses qui travaillent toutes ensembles. Nous avons beaucoup de terminologie pour ceci comme "séparation des préoccupations ", "cohérence et cohésion" et "modularité", mais cela se résume à "cette chose va ensemble et ne va pas avec cette autre".

[ob]The observer pattern is a great way to let those mostly unrelated lumps talk to
each other without them merging into one big lump. It's less useful *within* a
single lump of code dedicated to one feature or aspect.[oe]

Le patron observateur est un super moyen de permettre à ces morceaux apriori sans rapport de communiquer sans les fusionner en un morceau. C'est moins utile *à l'intérieur* d'un simple bout de code dédié à une seule fonctionnalité ou aspect.

[ob]That's why it fits our example well: achievements and physics are almost entirely
unrelated domains, likely implemented by different people. We want the bare
minimum of communication between them so that working on either one doesn't
require much knowledge of the other.[oe]

C'est pourquoi il s'adapte bien à notre exemple : les succès et la physique sont des domaines presque entièrement sans rapport, probablement implantés par des personnes différentes. Nous voulons le strict minimum de communication entre eux pour que ce fonctionnement sur l'un ou l'autre ne nécessite pas beaucoup de connaissance de l'autre.

## Les observateurs d'aujourd'hui

[ob]*Design Patterns* came out in <span name="90s">1994</span>. Back then,
object-oriented programming was *the* hot paradigm. Every programmer on Earth
wanted to "Learn OOP in 30 Days," and middle managers paid them based on the
number of classes they created. Engineers judged their mettle by the depth of
their inheritance hierarchies.[oe]

*Design Patterns* est sorti en <span name="90s">1994</span>. A l'époque, la programmation orientée-objet était *le* paradigme de choix. Chaque programmeur sur Terre voulait "Apprendre la POO en 30 jours", et les managers moyens les payaient en se basant sur le nombre de classes qu'ils avaient créées. Les ingénieurs jugeaient leur courage par la profondeur de leur hiérarchies d'héritage.

<aside name="90s">

[ob]That same year, Ace of Base had not one but *three* hit singles, so that may
tell you something about our taste and discernment back then.[oe]

Cette même année, Ace of Base n'avait pas eu un mais *trois* hits, donc ça démontre bien quelque chose sur notre goût et notre discernement de l'époque.

</aside>

[ob]The Observer pattern got popular during that zeitgeist, so it's no surprise that
it's class-heavy. But mainstream coders now are more comfortable with functional
programming. Having to implement an entire interface just to receive a
notification doesn't fit today's aesthetic.[oe]

Le patron Observateur fut populaire durant cette période, donc il n'est pas surprenant qu'il soit lourd en classe. Mais les codeurs du courant dominant maintenant sont plus à l'aise avec la programmation fonctionnel. Avoir à implémenter une interface entière juste pour recevoir une notification n'est pas adapté à l'esthétique d'aujourd'hui.

[ob]It feels <span name="different">heavyweight</span> and rigid. It *is*
heavyweight and rigid. For example, you can't have a single class that uses
different notification methods for different subjects.[oe]

Cela semble <span name="different">lourd</span> et rigide. *C'est* lourd et rigide. Par exemple, vous ne pouvez pas avoir une simple classe qui utilise différentes méthodes de notification pour différents sujets.

<aside name="different">

C'est pourquoi le sujet se passe habituellement lui-même à l'observateur. Car un observateur a seulement une méthode `onNotify()`, si il observe plusieurs sujets, il a besoin d'être capable de dire lequel l'a appelé.

</aside>

[ob]A more modern approach is for an "observer" to be only a reference to a method
or function. In languages with first-class functions, and especially ones with
<span name="closures">closures</span>, this is a much more common way to do
observers.[oe]

Une approche plus moderne est qu'un "observateur" soit seulement une référence à une méthode ou une fonction. Dans des langages avec des fonctions de première classe, et spécialement avec des <span name="closures">fermetures</span>, c'est une manière bien plus commune de faire des observateurs.

<aside name="closures">

De nos jours, pratiquement *tout* langage a des fermetures. C++ a réussi le challenge des fermetures dans un langage sans ramasse-miette, et même Java l'a finalement eu et les a introduites dans le JDK 8.

</aside>

[ob]For example, C# has "events" baked into the language. With those, the observer
you register is a "delegate", which is that language's term for a reference to a
method. In JavaScript's event system, observers *can* be objects supporting a
special `EventListener` protocol, but they can also just be functions. The
latter is almost always what people use.[oe]

Par exemple, C# a des "évènements" intégré à son langage. Avec ceux-ci, l'observateur que vous enregistrez est un "delagate" , qui est un terme de langage pour une référence à une méthode. Dans le système d'évènement de JavaScript, les observateurs *peuvent* être des objets supportant un protocole `EventListener` spécial, mais ils peuvent aussi être juste des fonctions. Le dernier est presque toujours ce que les gens utilisent.

[ob]If I were designing an observer system today, I'd make it <span
name="function">function-based</span> instead of class-based. Even in C++, I
would tend toward a system that let you register member function pointers as
observers instead of instances of some `Observer` interface.
[oe]

Si je conceptualisais un système d'observateur aujourd'hui, je le ferais <span name="function">basé sur les fonctions</span> au lieu de basé sur les classes. Même en C++, j'aurais tendance à aller vers un système qui vous permez d'enregistrer des pointeurs de fonction membre comme observateurs au lieu d'instances de classe d'une interface `Observer`.

<aside name="function">

[Here's][delegate] [ob]an interesting blog post on one way to implement this in C++.[oe]

[Ici][delegate] vous trouverez un article de blog intéressant sur une manière d'implémenter ceci in C++.

[delegate]: http://molecularmusings.wordpress.com/2011/09/19/generic-type-safe-delegates-and-events-in-c/

</aside>

## Les observateurs de demain

[ob]Event systems and other observer-like patterns are incredibly common these days.
They're a well-worn path. But if you write a few large apps using them, you
start to notice something. A lot of the code in your observers ends up looking
the same. It's usually something like:[oe]

Les systèmes d'évènement et les autres patron de type observateur sont incroyablement commun de nos jours. Ils sont un sentier battu. Mais si vous utilisez quelques grosses applications les utilisant, vous commencez à noter quelque chose. Beaucoup de code dans vos observateurs finit par se ressembler. C'est habituellement quelque chose comme : 

[ob]Get notified that some state has changed.[oe]

[ob]Imperatively modify some chunk of UI to reflect the new state.[oe]

  1. Etre notifié que quelque chose a changé.
  
  2. Modifier impérativement un morceau d'UI pour refléter le nouvel état.

[ob]It's all, "Oh, the hero health is 7 now? Let me set the width of the health bar
to 70 pixels." After a while, that gets pretty tedious. Computer science
academics and software engineers have been trying to eliminate that tedium for a
*long* time. Their attempts have gone under a number of different names:
"dataflow programming", "functional reactive programming", etc.[oe]

C'est tout, "Oh, la vie du héros est 7 maintenant ? Permet moi de configurer la largeur de la barre de vie à 70 pixels". Au bout d'un moment, ça devient assez fastidieux. Les académies d'informatique et les ingénieurs logiciel ont essayé d'éliminer cet ennui depuis *longtemps*. Leurs tentatives ont pris un nombre différent de nom: "programmation de flux de données" ,"programmation fonctionnelle réactive", etc.

[ob]While there have been some successes, usually in limited domains like audio
processing or chip design, the Holy Grail still hasn't been found. In the
meantime, a less ambitious approach has started gaining traction. Many recent
application frameworks now use "data binding".[oe]

Tant qu'il y a eu des succès habituellement dans des domaines limités comme le traitement audio ou la conception de puces, le Saint Graal n'a pas encore été trouvé. Pendant ce temps, une approche moins ambitieuse à commencé à avoir de l'attraction. Beaucoup de frameworks d'application récents maintenant utilisent le "data binding".

[ob]Unlike more radical models, data binding doesn't try to entirely eliminate
imperative code and doesn't try to architect your entire application around a
giant declarative dataflow graph. What it does do is automate the busywork where
you're tweaking a UI element or calculated property to reflect a change to some
value.[oe]

A la différence de modèles plus radicaux, le data binding n'essaie pas d'éliminer entièrement le code impératif et n'essaie pas d'architecturer voir application entière autour d'un graphe géant de flux de données déclaratif. Ce qu'il fait est d'automatiser le travail ou vous modifiez un élément d'UI ou une propriété calculée pour refléter un changement d'une valeur.

[ob]Like other declarative systems, data binding is probably a bit too slow and
complex to fit inside the core of a game engine. But I would be surprised if I
didn't see it start making inroads into less critical areas of the game like
UI.[oe]

Comme d'autres systèmes déclaratifs, le data binding est probablement un petit peu trop lent et complexe pour s'adapter à l'intérieur du coeur du moteur de jeu. Mais je serai surpris si je ne l'avais pas vu commencer à faire des progrès dans des aires moins critiques du jeu comme l'UI.

[ob]In the meantime, the good old Observer pattern will still be here waiting for
us. Sure, it's not as exciting as some hot technique that manages to cram both
"functional" and "reactive" in its name, but it's dead simple and it works. To
me, those are often the two most important criteria for a solution.
[oe]

Pendant ce temps, le bon vieux patron observateur sera encore là pour nous. Bien sur ce n'est pas aussi excitant que certaines techniques récentes qui gère pour fourre à la fois "fonctionnel" et "réactif" dans son nom, mais c'est très simple et ça marche. Pour moi, se sont souvent les deux critères les plus important pour une solution.