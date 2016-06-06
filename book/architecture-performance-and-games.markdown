^title Architecture, Performance, and Games
^section Introduction

[ob]Before we plunge headfirst into a pile of patterns, I thought it might help to
give you some context about how I think about software architecture and how it
applies to games. It may help you understand the rest of this book better. If
nothing else, when you get dragged into an <span name="O_ammo">argument</span>
about how terrible (or awesome) design patterns and software architecture are,
it will give you some ammo to use.[oe]

Avant que nous plongions la tête la première dans une pile de patrons, je pense que ça pourrait aider de donner un aperçu de comment je vois l'architecture logiciel et comment elle s'applique au jeux. Cela pourrait vous aider à mieux comprendre le reste de ce livre. Sinon, quand vous tenterez <span name="ammo">d'argumenter</span> si un patron de conception et une architecture logiciel sont nuls (ou géniaux), ça vous donnera quelques munitions à utiliser.


<aside name="O_ammo">

[ob]Note that I didn't presume which side you're taking in that fight. Like any arms
dealer, I have wares for sale to all combatants.[oe]

</aside>

<aside name="ammo">

Notez que je n'ai pas présumé de quel coté vous êtes dans ce combat. Comme tout bon vendeur d'armes, J'ai du matériel à vendre pour tout combattants.

</aside>

[ob]## What is Software Architecture?[oe]

## Qu'est qu'une architecture logiciel ?

[ob]<span name="O_won't">If</span> you read this book cover to cover, you won't come
away knowing the linear algebra behind 3D graphics or the calculus behind game
physics. It won't show you how to alpha-beta prune your AI's search tree or
simulate a room's reverberation in your audio playback.[oe]

<span name="won't">Si</span> vous lisez ce livre du début à la fin, vous ne viendrez pas à bout de l'algèbre linéaire derrière la 3D ou le calcul impliqué dans la physique des jeux. Ca ne vous montrera pas comment faire un élagage alpha-beta de l'arbre de recherche de votre IA ou simuler la réverbération d'une chambre dans la lecture de votre audio.

<aside name="O_won't">

[ob]Wow, this paragraph would make a terrible ad for the book.[oe]

</aside>

<aside name="won't">

Ouah ! Ce paragraphe pourrait faire une pub terrible pour ce livre.

</aside>

[ob]Instead, this book is about the code *between* all of that. It's less about
writing code
than it is about *organizing* it. Every program has *some* organization, even if
it's just "jam the whole thing into `main()` and see what happens", so I think
it's more interesting to talk about what makes for *good* organization. How do
we tell a good architecture from a bad one?[oe]

A la place, ce livre parle du code *entre* tout ça. C'est moins sur l'écriture du code que sur son *organisation*. Tout programme a une *certaine* organisation, même si c'est juste  "tout foutre dans `main()` et voir ce qui arrive", donc je crois qu'il est plus intéressant de parler de ce qui fait une *bonne* organisation. Comment parler d'une bonne architecture à partir d'une mauvaise ?

[ob]I've been mulling over this question for about five years. Of course, like you,
I have an intuition about good design. We've all suffered through codebases so
<span name="O_suffered">bad</span>, the best you could hope to do for them is take
them out back and put them out of their misery.[oe]

<aside name="O_suffered">

[ob]Let's admit it, most of us are *responsible* for a few of those.[oe]

</aside>

J'ai ressassé cette question pendant cinq années. Bien sûr, comme vous, j'ai une intuition d'une bonne conception. Nous avons tous souffert de ces codebases si <span name="suffered">mauvais</span>, le meilleur que vous pourriez espérer faire est de les reprendre et d'y enlever leur misère.

<aside name="suffered">

Admettons le, la plupart de nous en est *responsable* pour une petite partie.

</aside>

[ob]A lucky few have had the opposite experience, a chance to work with beautifully
designed code. The kind of codebase that feels like a perfectly appointed luxury
hotel festooned with concierges waiting eagerly on your every whim. What's the
difference between the two?[oe]

Une poignée de chanceux ont eu une expérience opposée, une chance de travailler avec du code superbement conçu. Le genre de codebase qui ressemble à un hôtel de luxe parfaitement équipé, orné de concierges attendant avec impatience tous vos caprices. Quelle est la différence entre les deux ?

[ob]### What is *good* software architecture?[oe]

### Qu'est qu'une *bonne* architecture ?

[ob]For me, good design means that when I make a change, it's as if the entire
program was crafted in anticipation of it. I can solve a task with just a few
choice function calls that slot in perfectly, leaving not the slightest ripple
on the placid surface of the code.[oe]

Pour moi, une bonne conception signifie que quand je change quelque chose, c'est comme si le programme entier était fabriqué en anticipation de ce changement. Je peux résoudre une tâche en choisissant d'appeler quelques fonctions qui s'encastrent parfaitement, ne laissant pas la moindre ondulation sur la surface calme du code.

[ob]That sounds pretty, but it's not exactly actionable. "Just write your code so
that changes don't disturb its placid surface." Right.[oe]

Ceci semble suffisant, mais ce n'est pas tout à fait réalisable. "Ecrire votre code en sorte que des changements ne perturbent pas cette surface calme". D'accord.

[ob]Let me break that down a bit. The first key piece is that *architecture is about
change*. Someone has to be modifying the codebase. If no one is touching the
code -- whether because it's perfect and complete or so wretched no one will
sully their text editor with it -- its design is irrelevant. The measure of a
design is how easily it accommodates changes. With no changes, it's a runner who
never leaves the starting line.[oe]

Permettez moi de décomposer ça un peu. La première pièce clé est que une *architecture* est focalisée sur le *changement*. Quelqu'un a à modifier le codebase. Si personne ne touche le code -- si par ce que c'est parfait et complet ou si misérable que personne ne souillera son éditeur de texte avec ça -- sa conception n'est pas pertinente. L'évaluation d'une conception est fonction de combien elle rend facile les modifications. Sans aucune modification, c'est comme un coureur qui ne quitte jamais la ligne de départ.

[ob]### How do you make a change?[oe]

### Comment effectuer une modification ?

[ob]Before you can change the code to add a new feature, to fix a bug, or for whatever
reason caused you to fire up your editor, you have to understand what the
existing code is doing. You don't have to know the whole program, of course, but
you need to <span name="O_ocr">load</span> all of the relevant pieces of it into
your primate brain.[oe]

<aside name="O_ocr">

[ob]It's weird to think that this is literally an OCR process.[oe]

</aside>

Avant que vous puissiez changer le code pour ajouter une nouvelle fonctionnalité, pour corriger un bug, ou pour quelconque raison provoquée par vous-même pour lancer votre éditeur, vous devez comprendre ce que fait le code existant. Vous n'avez pas à connaitre le programme entier, bien sur, mais vous avez besoin de <span name="ocr">charger</span> tous les éléments pertinents dans votre cerveau de primate.

<aside name="ocr">

C'est étrange de penser que c'est littéralement un processus OCR.

</aside>

[ob]We tend to gloss over this step, but it's often the most time-consuming part of
programming. If you think paging some data from disk into RAM is slow, try
paging it into a simian cerebrum over a pair of optical nerves.[oe]

Nous avons tendance à bâcler cette étape, pourtant c'est souvent la partie la plus chronophage de la programmation. Si vous pensez que paginer quelques données du disque dur à la RAM est lent, essayez de paginer ça à l'intérieur d'un cerveau simien sur une paire de nerfs optiques.

[ob]Once you've got all the right context into your wetware, you think for a bit and
figure out your solution. There can be a lot of back and forth here, but often
this is relatively straightforward. Once you understand the problem and the
parts of the code it touches, the actual coding is sometimes trivial.[oe]

Une fois que vous avez tout ça dans votre boîte crânienne, vous réfléchissez un peu et vous trouvez votre solution. Il peut y avoir beaucoup d'aller-retours à ce moment là, mais souvent c'est relativement simple. Une fois que vous comprenez le problème et les parties du code qu'il touche, le vrai codage est parfois banal.

[ob]You beat your meaty fingers on the keyboard for a while until the right colored
lights blink on screen and you're done, right? Not just yet! Before you write
<span name="O_tests">tests</span> and send it off for code review, you often have
some cleanup to do.[oe]

Vous agitez vos petits doigts boudinés sur le clavier pendant un certain temps jusqu'à ce que la bonne couleur clignote sur l'écran et c'est bon, pas vrai ? Pas encore ! Pas avant que vous ayez écrit des <span name="tests">tests</span> et envoyez ça pour une revue de code, vous avez souvent un peu de nettoyage à faire.

<aside name="tests">

[ob]Did I say "tests"? Oh, yes, I did. It's hard to write unit tests for some game
code, but a large fraction of the codebase is perfectly testable.[oe]

Ai-je dit "tests" ? Oh, oui, je l'ai dit. Il est difficile d'écrire des tests unitaires pour certains codes de jeu, mais une grande partie du codebase est parfaitement testable.

[ob]I won't get on a soapbox here, but I'll ask you to consider doing more automated
testing if you aren't already. Don't you have better things to do than manually
validate stuff over and over again?[oe]

Je ne m'emporterai pas là-dessus, mais je vous demanderai d'essayer de faire des tests plus automatisés si vous ne le faites pas déjà. N'avez vous pas mieux à faire que de valider manuellement les choses encore et encore ?

</aside>

[ob]You jammed a bit more code into your game, but you don't want the next person to
come along to trip over the wrinkles you left throughout the source. Unless the
change is minor, there's usually a bit of reorganization to do to make your new
code integrate seamlessly with the rest of the program. If you do it right, the
next person to come along won't be able to tell when any line of code was
written.[oe]

Vous foutez un peu plus de code dans votre jeu, mais vous ne voulez pas que la personne suivante parcours les plis que vous avez laissé dans les sources. A moins que le changement soit mineur, il y a habituellement un peu de réorganisation à faire pour que votre nouveau code s'intègre de façon transparente avec le reste du programme. Si vous faites ça correctement, la personne suivante ne sera pas capable de dire quand quelconque ligne de code a été écrite.

[ob]In short, the flow chart for programming is something like:[oe]

En résumé, le diagramme de flux pour la programmation est quelque chose comme :

<span name="life-cycle"></span>

<img src="images/architecture-cycle.png" alt="Comprendre le problème &rarr; comprendre le code &rarr; Coder la solution &rarr; la rendre propre &rarr; et revenir au début." />

<aside name="life-cycle">

[ob]The fact that there is no escape from that loop is a little alarming now that I
think about it.[oe]

Le fait est que il n'y a aucun échappatoire dans cette boucle est un petit peu inquiétant maintenant que j'y pense.

</aside>

[ob]### How can decoupling help?[oe]

### Comment le découplage peut aider ?

[ob]While it isn't obvious, I think much of software architecture is about that
learning phase. Loading code into neurons is so painfully slow that it pays to
find strategies to reduce the volume of it. This book has an entire section on
[*decoupling* patterns](decoupling-patterns.html), and a large chunk of *Design
Patterns* is about the same idea.[oe]

Bien que ce ne soit pas évident, je pense que beaucoup de l'architecture logiciel est cette phase d'apprentissage. Charger du code dans vos neurones est si péniblement lent que cela paie de trouver des stratégies pour en réduire le volume. Ce livre a une section entière sur les [patrons de *découplage*](decoupling-patterns.html), et un gros morceau de *Design Patterns* repose sur la même idée.

[ob]You can define "decoupling" a bunch of ways, but I think if two pieces of code
are coupled, it means you can't understand one without understanding the other.
If you *de*-couple them, you can reason about either side independently. That's
great because if only one of those pieces is relevant to your problem, you just
need to load *it* into your monkey brain and not the other half too.[oe]

Vous pouvez définir le "découplage" de différentes manières, mais je crois que si deux morceaux de code sont couplés, cela signifie que vous ne pouvez pas comprendre l'un sans comprendre l'autre. Si vous les *découplez*, vous pouvez raisonner sur chacun indépendamment. Cela est vraiment bien par ce que si seulement un de ces morceaux est pertinent pour votre problème, vous avez juste besoin de *le* charger dans votre cerveau de singe et pas l'autre moitié.

[ob]To me, this is a key goal of software architecture: **minimize the amount of
knowledge you need to have in-cranium before you can make progress.**[oe]

Pour moi, voici le but clé de l'architecture logiciel : **Minimiser le montant de connaissance dont vous avez besoin d'avoir en tête avant de pouvoir réaliser un avancée.**

[ob]The later stages come into play too, of course. Another definition of decoupling
is that a *change* to one piece of code doesn't necessitate a change to another.
We obviously need to change *something*, but the less coupling we have, the less
that change ripples throughout the rest of the game.[oe]

Les étapes suivantes rentrent en jeu, bien sûr. Une autre définition de découplage est que la *modification* d'un morceau de code ne nécessite pas la modification d'un autre. Nous avons évidemment besoin de changer *quelque chose*, mais moins de couplage nous avons, moins les changements se répercuteront au reste du jeu.

[ob]## At What Cost?[oe]

## A Quel Coût ?

[ob]This sounds great, right? Decouple everything and you'll be able to code like
the wind. Each change will mean touching only one or two select methods, and you
can dance across the surface of the codebase leaving nary a shadow.
[oe]

Ceci semble bien, n'est ce pas ? Découplez tout et vous serez capable de coder comme le vent. Chaque modification signifiera toucher seulement une ou deux méthodes, et vous pourrez danser sur la surface du codebase en ne laissant rien moins qu'une ombre.

[ob]This feeling is exactly why people get excited about abstraction, modularity,
design patterns, and software architecture. A well-architected program really is
a joyful experience to work in, and everyone loves being more productive. Good
architecture makes a *huge* difference in productivity. It's hard to overstate
how profound an effect it can have.[oe]

Ce sentiment est exactement ce pourquoi les gens sont enthousiastes à propos de l'abstraction, de la modularité, des patrons de conception, et de l'architecture logicielle. Un programme bien architecturé fait que travailler dessus est une expérience joyeuse, et chacun aime bien être plus productif. Une bonne architecture rend une *haute* différence en productivité. C'est difficile d'exagérer sur l'effet profond que ca peut avoir.

[ob]But, like all things in life, it doesn't come free. Good architecture takes real
effort and discipline. Every time you make a change or implement a feature, you
have to work hard to integrate it gracefully into the rest of the program. You
have to take great care to both <span name="maintain">organize</span> the code
well and *keep* it organized throughout the thousands of little changes that
make up a development cycle.[oe]

Mais, comme toutes choses dans la vie, ça n'est pas gratuit. Une bonne architecture demande un réel effort et une discipline. A chaque fois que vous faites une modification ou implémenter une fonctionnalité, vous devez travailler dur pour l'intégrer gracieusement au reste du programme. Vous devez prendre grand soin d'organiser à la fois le code correctement et le *garder* <span name="maintain">organisé</span> à travers les milliers de petits changements qui font votre cycle de développement.

<aside name="maintain">

[ob]The second half of this -- maintaining your design -- deserves special
attention. I've seen many programs start out beautifully and then die a death of a
thousand cuts as programmers add "just one tiny little hack" over and over
again.
[oe]

La seconde moitié de ceci -- maintenir votre conception -- mérite une attention spéciale. J'ai vu beaucoup de programmes qui démarrent superbement et puis meurt des suites de milliers de coupures car les programmeurs ajoutent "juste un tout petit hack" inlassablement.

[ob]Like gardening, it's not enough to put in new plants. You must also weed and
prune.[oe]

Tout comme le jardinage, ça ne suffit pas de planter de nouvelles plantes. Vous devez aussi désherber et tailler.

</aside>

[ob]You have to think about which parts of the program should be decoupled and
introduce abstractions at those points. Likewise, you have to determine where
extensibility should be engineered in so future changes are easier to make.[oe]

Vous devez penser à quelles parties du programme devraient être découplées et introduire des abstractions à ces endroits là. Egalement, vous devez déterminer où l'extensibilité devrait être présente pour que dans le futur les changements soient plus faciles à effectuer.

[ob]People get really excited about this. They envision future developers (or just
their future self) stepping into the codebase and finding it open-ended,
powerful, and just beckoning to be extended. They imagine The One Game Engine To
Rule Them All.[oe]

Les gens sont vraiment enthousiastes à ce sujet. Ils envisagent les futurs développeurs (ou juste eux-mêmes) allant dans le codebase et trouvant ça illimité, puissant, et invitant à être étendu. Ils imaginent l'Unique Moteur De Jeu Pour Les Gouverner Tous.

[ob]But this is where it starts to get tricky. Whenever you add a layer of
abstraction or a place where extensibility is supported, you're *speculating*
that you will need that flexibility later. You're adding code and complexity to
your game that takes time to develop, debug, and maintain.[oe]

Mais c'est ici que ça commence à se corser. A chaque fois que vous ajoutez une couche d'abstraction ou un endroit où l'extensibilité est supportée, vous êtes en train de *spéculer* si vous aurez besoin de cette flexibilité plus tard. Vous ajoutez du code et de la complexité à votre jeu qui prend du temps à développer, à déboguer, et à maintenir.

[ob]That effort pays off if you guess right and end up touching that code later. But
<span name="yagni">predicting</span> the future is *hard*, and when that
modularity doesn't end up being helpful, it quickly becomes actively harmful.
After all, it is more code you have to deal with.[oe]

Cet effort se révèle payant si yous avez vu juste et que vous finissez par retoucher ce code plus tard. Mais <span name="yagni">prédire</span> le futur est *difficile*, et quand la modularité finit par ne pas être utile, ça devient rapidement nuisible. Après tout, ça fait plus de code à gérer.

<aside name="yagni">

[ob]Some folks coined the term "YAGNI" -- [You aren't gonna need
it](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it) -- as a mantra to use
to fight this urge to speculate about what your future self may want.[oe]

Des gens ont inventé le terme " YAGNI " -- [You aren't gonna need it (vous n'en aurez pas besoin)](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it) -- comme un mantra à utiliser pour combattre ce besoin de spéculer sur ce que votre futur peut vouloir.

</aside>

[ob]When people get overzealous about this, you get a codebase whose architecture
has spiraled out of control. You've got interfaces and abstractions everywhere.
Plug-in systems, abstract base classes, virtual methods galore, and all sorts of
extension points.[oe]

Quand les gens sont sur-zélés en ça, vous obtenez un codebase pour dont l'architecture est devenue hors de contrôle. Vous avez des interfaces et des abstractions de partout. Systèmes de plugins, classes de base abstraite, méthodes virtuelles en abondance, et toutes sortes de points d'extension.

[ob]It takes you forever to trace through all of that scaffolding to find some real
code that does something. When you need to make a change, sure, there's probably
an interface there to help, but good luck finding it. In theory, all of this
decoupling means you have less code to understand before you can extend it, but
the layers of abstraction themselves end up filling your mental scratch disk.[oe]

Ca vous prendra une éternité pour vous frayer un chemin à travers ces échafaudages pour trouver du code qui fait quelque chose. Quand vous avez besoin de faire une modification, bien sûr, il y a surement une interface à la rescousse, mais bonne chance pour la trouver. En théorie, tout ce découplage signifie que vous avez moins de code à comprendre avant que vous puissiez l'étendre, mais les couches d'abstraction elles-même finissent par remplir votre disque mental de travail.

[ob]Codebases like this are what turn people *against* software architecture, and
design patterns in particular. It's easy to get so wrapped up in the code itself
that you lose sight of the fact that you're trying to ship a *game*. The siren
song of extensibility sucks in countless developers who spend years working on
an "engine" without ever figuring out what it's an engine *for*.[oe]

Des codebases comme ceux-ci sont ceux qui braquent les gens *contre* l'architecture logicielle, et les patrons de conception en particulier. C'est facile d'être plongé dans le code à un point de perdre de vue le fait que vous êtes en train d'essayer de sortir un *jeu*. Le chant des sirènes de l'extensibilité engloutit d'innombrables développeurs qui passent des années à travailler sur un " moteur " sans jamais comprendre que c'est un moteur *pour* faire un jeu.

[ob]## Performance and Speed[oe]

## Performance et vitesse

[ob]There's another critique of software architecture and abstraction that you hear
sometimes, especially in game development: that it hurts your game's
performance. Many patterns that make your code more flexible rely on virtual
dispatch, interfaces, pointers, messages, and <span name="templates">other
mechanisms</span> that all have at least some runtime cost.[oe]

Il y a une autre critique de l'architecture logicielle et de l'abstraction que vous entendez parfois, spécialement dans le développement de jeu : que ça nuit aux performances de votre jeu. Beaucoup de patrons qui rendent votre code plus flexible repose sur l'envoi virtuel, les interfaces, les pointeurs, les messages, et <span name="templates">autres mécanismes</span> qui ont tous au moins quelque coût de temps d'exécution.

<aside name="templates">

[ob]One interesting counter-example is templates in C++. Template metaprogramming
can sometimes give you the abstraction of interfaces without any penalty at
runtime.[oe]

Un contre exemple intéressant est les templates en C++. La métaprogrammation par les templates peut parfois vous donner l'abstraction des interfaces sans pénalité à l'exécution.

[ob]There's a spectrum of flexibility here. When you write code to call a concrete
method in some class, you're fixing that class at *author* time -- you've
hard-coded which class you call into. When you go through a virtual method or
interface, the class that gets called isn't known until *runtime*. That's much
more flexible but implies some runtime overhead.[oe]

Il y a un spectre de flexibilité ici. Quand vous écrivez du code pour appeler une méthode concrète dans une classe, vous fixez cette classe au temps de l'*auteur* -- vous avez codé en dur quelle classe sera appelée. Quand vous utilisez une méthode virtuelle ou une interface, la classe qui sera appelée n'est connu qu'à l'*exécution*. Cela est beaucoup plus flexible mais implique un coût supplémentaire en temps d'exécution.

[ob]Template metaprogramming is somewhere between the two. There, you make the
decision of which class to call at *compile time* when the template is
instantiated.[oe]

La métaprogrammation par les templates est quelque part entre les deux. A partir de là, vous prenez la décision de quelle classe appeler *à la compilation* quand le template est instancié.

</aside>

[ob]There's a reason for this. A lot of software architecture is about making your
program more flexible. It's about making it take less effort to change it. That
means encoding fewer assumptions in the program. You use interfaces so that your
code works with *any* class that implements it instead of just the one that does
today. You use <a href="observer.html" class="gof-pattern">observers</a> and <a
href="event-queue.html" class="pattern">messaging</a> to let two parts of the
game talk to each other so that tomorrow, it can easily be three or four.[oe]

Il y a une raison pour ceci. Une grande part de l'architecture logicielle est de rendre votre programme plus flexible. C'est faire en sorte que le changer requiert moins d'effort. Ceci veut dire coder moins d'hypothèses dans le programme. Vous pouvez utiliser des interfaces pour que votre code fonctionne avec *chaque* classe qui les implémente au lieu de la seule qui fonctionne aujourd'hui. Vous utilisez des <a href="observer.html" class="gof-pattern">observateurs</a> et de la <a
href="event-queue.html" class="pattern">messagerie</a> pour permettre à deux parties du jeu de communiquer entre elles pour que demain, ça puisse être trois ou quatre.

[ob]But performance is all about assumptions. The practice of optimization thrives
on concrete limitations. Can we safely assume we'll never have more than 256
enemies? Great, we can pack an ID into a single byte. Will we only call a method
on one concrete type here? Good, we can statically dispatch or inline it. Are
all of the entities going to be the same class? Great, we can make a nice <a
href="data-locality.html" class="pattern">contiguous array</a> of them.[oe]

Mais la performance n'est qu'hypothèses. La pratique de l'optimisation s'épanouit dans les limitations concrètes. Pouvez vous à coup sûr supposer que nous n'aurons pas plus que 256 ennemies ? Super, nous pouvons empaqueter un identifiant dans un simple octet. Appellerons nous seulement une méthode sur un type concret ici ? Bien, nous pouvons l'envoyer statiquement ou l'inliner. Est ce que les entités sont elles toutes de la même classe ? Super, nous pouvons en faire un <a
href="data-locality.html" class="pattern">tableau contigu</a> sympathique.

[ob]This doesn't mean flexibility is bad, though! It lets us change our game
quickly, and *development* speed is absolutely vital for getting to a fun
experience. No one, not even Will Wright, can come up with a balanced game
design on paper. It demands iteration and experimentation.[oe]

Ceci ne veut pas dire que le flexibilité est mauvaise cependant ! Elle nous permet de changer notre jeu rapidement, et la vitesse de *développement* est absolument vitale pour obtenir une expérience fun. Personne, pas même Will Wright, ne peut inventer un design de jeu équilibré sur le papier. Cela demande de l'itération et de l'expérimentation.

[ob]The faster you can try out ideas and see how they feel, the more you can try and
the more likely you are to find something great. Even after you've found the
right mechanics, you need plenty of time for tuning. A tiny imbalance can wreck
the fun of a game.
[oe]

Plus vite vous testez vos idées et plus vous voyez ce qu'elle valent, plus vous testez et plus il est probable que vous trouviez quelque chose de super. Même quand vous avez trouvé les bonnes mécaniques, vous avez besoin de beaucoup de temps pour les régler. Un léger déséquilibre peut briser le fun du jeu.

[ob]There's no easy answer here. Making your program more flexible so you can
prototype faster will have some performance cost. Likewise, optimizing your code
will make it less flexible.[oe]

Il n'y a pas de réponse facile. Rendre votre programme plus flexible de sorte que vous puissiez prototyper plus vite aura un certain coût sur la performance. De même, optimiser votre code le rendra moins flexible.

[ob]My experience, though, is that it's easier to make a fun game fast than it is to
make a fast game fun. One compromise is to keep the code flexible until the
design settles down and then tear out some of the abstraction later to improve
your performance.[oe]

Mon expérience, cependant, est qu'il est plus facile de faire un jeu amusant rapidement que de faire un jeu rapide amusant. Un compromis est de garder le code flexible jusqu'à ce que la conception s'installe et puis arracher un peu d'abstraction plus tard pour améliorer les performances.

[ob]## The Good in Bad Code[oe]

## Le Bon Dans Le Mauvais Code

[ob]That brings me to the next point which is that there's a time and place for
different styles of coding. Much of this book is about making maintainable,
clean code, so my allegiance is pretty clearly to doing things the "right" way,
but there's value in slapdash code too.[oe]

Cela m'amène au prochain sujet qui est qu'il y a un temps et une place pour différents styles de codage. La plupart de ce livre concerne le rendre maintenable, le code propre, donc mon allégeance est clairement de faire les choses de la " bonne " manière, mais il y a de la valeur dans un code sale aussi.

[ob]Writing well-architected code takes careful thought, and that translates to
time. Moreso, *maintaining* a good architecture over the life of a project takes
a lot of effort. You have to treat your codebase like a good camper does their
campsite: always try to leave it a little better than you found it.[oe]

L'écriture de code bien architecturé demande de la réflexion, et cela se traduit en temps. D'autant plus, *maintenir* une bonne architecture durant la vie d'un projet demande beaucoup d'efforts. Vous devez traiter votre codebase comme un bon campeur le fait de son campement : toujours essayer de le laisser dans un meilleur état que celui dans lequel vous l'avez trouvé.

[ob]This is good when you're going to be living in and working on that code for a
long time. But, like I mentioned earlier, game design requires a lot of
experimentation and exploration. Especially early on, it's common to write code
that you *know* you'll throw away.[oe]

Ceci est bon quand vous allez vivre et travailler avec ce code pendant une longue période. Cependant, comme je l'ai mentionné plus tôt, la conception de jeu nécessite beaucoup d'expérimentation et d'exploration. Spécialement sur le tôt, il est commun d'écrire du code dont vous *savez* qui partira à la poubelle.

[ob]If you just want to find out if some gameplay idea plays right at all,
architecting it beautifully means burning more time before you actually get it
on screen and get some feedback. If it ends up not working, that time spent
making the code elegant goes to waste when you delete it.[oe]

Si vous voulez juste découvrir si une idée de gameplay est bonne, l'architecturer en toute beauté signifie brûler plus de temps avant que vous l'ayez sur l'écran et puissiez obtenir un retour. Si elle finit par ne pas fonctionner, ce temps dépensé à rendre le code élégant peut être perçu comme du gaspillage quand vous le supprimez.

[ob]Prototyping -- slapping together code that's just barely functional enough to
answer a design question -- is a perfectly legitimate programming practice.
There is a very large caveat, though. If you write throwaway code, you *must*
ensure you're able to throw it away. I've seen bad managers play this game time
and time again:[oe]

Le prototypage -- mélanger du code qui est tout juste fonctionnel pour répondre à une question de conception -- est une pratique de programmation parfaitement légitime. Il y a un gros inconvénient, malgré tout. Si vous écrivez du code poubelle, vous *devez* vous assurer que vous êtes capable de le jeter. J'ai vu de mauvais managers jouer à ce jeu maintes et maintes fois :

> [ob]Boss: "Hey, we've got this idea that we want to try out. Just a prototype, so don't feel you need to do it right. How quickly can you slap something together?"[oe]

> Patron : "Hé, On a une idée qu'on voudrait essayer. Juste un prototype, tu n'as pas besoin de faire ça bien. En combien de temps tu peux le faire ?"

> [ob]Dev: "Well, if I cut lots of corners, don't test it, don't document it, and it has tons of bugs, I can give you some temp code in a few days."[oe]

> Dev : "Voyons, si j'exclus beaucoup de coins sombres, et que je ne le teste pas, ne le documente pas, et que ca  contient des tonnes de bugs, Je peux vous produire un code en quelques jours."

> [ob]Boss: "Great!"[oe]

> Patron : "Super !"

[ob]*A few days pass...*[oe]

*Quelques jours passent...*

> [ob]Boss: "Hey, that prototype is great. Can you just spend a few hours cleaning it up a bit now and we'll call it the real thing?"[oe]

> Patron : "Hé, ce prototype est super. Peux tu passer quelques heures à le rendre un peu propre maintenant et on aura notre code final ? "

[ob]You need to make sure the people using the <span
name="throwaway">throwaway</span> code understand that even though it kind of
looks like it works, it *cannot* be maintained and *must* be rewritten. If
there's a *chance* you'll end up having to keep it around, you may have to
defensively write it well.[oe]

Vous avez besoin d'être sûr que les gens utilisant le code <span
name="throwaway">poubelle</span> comprennent que même si il fonctionne, il *ne peut pas* être maintenu et *doit* être réécrit. Si il est *probable* que vous finissiez par le garder, vous pourriez l'écrire correctement par anticipation.

<aside name="throwaway">

[ob]One trick to ensuring your prototype code isn't obliged to become real code is
to write it in a language different from the one your game uses. That way, you have to
rewrite it before it can end up in your actual game.[oe]

Un moyen de s'assurer que votre code prototype n'est pas obligé de devenir du vrai code est d'écrire ça dans un langage autre que celui que votre jeu utilise. De cette façon, vous aurez à le réécrire avant qu'il soit intégré à votre jeu.

</aside>

[ob]## Striking a Balance[oe]

## Trouver le juste milieu

[ob]We have a few forces in play:[oe]

Nous avons quelques forces en jeu :

1. [ob]<span name="speed">We</span> want nice architecture so the code is easier to understand over the lifetime of the project.[oe]
<span name="speed">Nous</span> voulons une architecture sympathique pour que le code soit plus facile à comprendre pendant la durée de vie du projet.

2. [ob]We want fast runtime performance.[oe]
Nous voulons des performances à l'exécution.
3. [ob]We want to get today's features done quickly.[oe]
Nous voulons avoir les fonctionnalités d'aujourd'hui rapidement codées.

<aside name="speed">

[ob]I think it's interesting that these are all about some kind of speed: our
long-term development speed, the game's execution speed, and our short-term
development speed.[oe]

Je crois qu'il est intéressant qu'ils soient tous relié à une sorte de vitesse : la vitesse de notre développement sur le long terme, la vitesse d'exécution du jeu, et la vitesse de notre développement sur le court terme.

</aside>

[ob]These goals are at least partially in opposition. Good architecture improves
productivity over the long term, but maintaining it means every change requires
a little more effort to keep things clean.[oe]

Ces buts sont au moins partiellement opposés. Une bonne architecture améliore la productivité sur le long terme, mais maintenir signifie que chaque changement nécessite un petit peu plus d'effort pour garder les choses propres.

[ob]The implementation that's quickest to write is rarely the quickest to *run*.
Instead, optimization takes significant engineering time. Once it's done, it
tends to calcify the codebase: highly optimized code is inflexible and very
difficult to change.[oe]

L'implémentation qui est la plus rapide à écrire est rarement la plus rapide à s'*exécuter*. Au lieu de ça, l'optimisation prends un temps d'ingénierie significatif. Une fois que c'est fait, cela tend à calcifier le codebase : du code hautement optimisé est inflexible et très difficile à modifier.

[ob]There's always pressure to get today's work done today and worry about
everything else tomorrow. But if we cram in features as quickly as we can, our
codebase will become a mess of hacks, bugs, and inconsistencies that saps our
future productivity.[oe]

Il y a toujours une pression pour arriver à faire le travail d'aujourd'hui aujourd'hui et s'inquiéter sur ce qu'il adviendra demain. Mais si nous faisons du bourrage de fonctionnalités aussi vite que nous le pouvons, notre codebase deviendra une désordre fait de hacks, de bugs, et d'incohérences qui sape notre productivité future.

[ob]There's no simple answer here, just trade-offs. From the email I get, this
disheartens a lot of people. Especially for novices who just want to make a
game, it's intimidating to hear, "There is no right answer, just different
flavors of wrong."[oe]

Il n'y a pas de réponses simples ici, seulement des compromis. A partir des emails que je reçois, ceci décourage beaucoup de gens. Spécialement pour des novices qui veulent juste faire un jeu, il est intimidant d'entendre, "Il n'y a aucune bonne réponse, juste différents genre mauvais goûts."

[ob]But, to me, this is exciting! Look at any field that people dedicate careers to
mastering, and in the center you will always find a set of intertwined
constraints. After all, if there was an easy answer, everyone would just do
that. A field you can master in a week is ultimately boring. You don't hear of
someone's distinguished career in <span name="ditch">ditch digging</span>[oe]

Mais, pour moi, ceci est exitant ! Rechercher des domaines dans lesquels des gens dédient leur carrière pour les maitriser, et au milieu vous trouverez toujours un ensemble de contraintes entrelacées. Après tout, si il y avait une réponse facile, chacun se contenterait de l'utiliser. Un domaine que vous pouvez maitriser en une semaine est fatiguant au bout du compte. Vous n'entendrez jamais parler de la carrière distinguée de quelqu'un en <span name="ditch">creusage de fossé</span>.

<aside name="ditch">

[ob]Maybe you do; I didn't research that analogy. For all I know, there could be avid
ditch digging hobbyists, ditch digging conventions, and a whole subculture
around it. Who am I to judge?[oe]

Peut être qu'il y en a; Je n'ai pas recherché cette analogie. Pour le peu que je sais, Il pourrait y avoir d'avides creuseurs de fossé passionnés, avec des rassemblements de creusage de fossé, et une sous-culture entière autour de ça. Qui suis-je pour juger ?

</aside>

[ob]To me, this has much in common with games themselves. A game like chess
can never be mastered because all of the pieces are so perfectly balanced
against one another. This means you can spend your life exploring the vast space
of viable strategies. A poorly designed game collapses to the one winning tactic
played over and over until you get bored and quit.[oe]

Pour moi, Ceci a beaucoup en commun avec les jeux eux-mêmes. Un jeu comme les échecs ne peut jamais être maitrisé par ce que toutes les pièces sont si parfaitement équilibrées les unes contres les autres. Ceci veut dire que vous pouvez passer votre vie à explorer le vaste champs des stratégies viables. Un jeu pauvrement conçu s'effondre à cause de l'unique tactique jouée encore et encore jusqu'à ce que vous soyez fatigué et abandonnez.

[ob]## Simplicity[oe]

## Simplicité

[ob]Lately, I feel like if there is any method that eases these constraints, it's
*simplicity*. In my code today, I try very hard to write the cleanest, most
direct solution to the problem. The kind of code where after you read it, you
understand exactly what it does and can't imagine any other possible solution.[oe]

Récemment, j'ai eu l'impression que si il y a une méthode qui facilite ces contraintes, c'est la *simplicité*. Dans mon code d'aujourd'hui, j'essaie avec grande difficulté d'écrire la solution au problème la plus propre et la plus directe. Le genre de code pour lequel après l'avoir relu, vous comprenez exactement ce que ça fait et n'imaginez pas d'autres solutions possibles.

[ob]I aim to get the data structures and algorithms right (in about that order) and
then go from there. I find if I can keep things simple, there's less code
overall. That means less code to load into my head in order to change it.[oe]

Je cherche à rendre les structures de données et les algorithmes corrects (à peu près dans cet ordre) et puis j'avance à partir de là. Je trouve que si je garde les choses simples, il y a moins de code globalement. Cela signifie moins de code à charger dans ma tête en vue de le modifier.

[ob]It often runs fast because there's simply not as much overhead and not much code
to execute. (This certainly isn't always the case though. You can pack a lot of
looping and recursion in a tiny amount of code.)[oe]

Ca va souvent plus vite simplement par ce que ça ne prend que peu de temps en plus et il y a peu de code à exécuter. (Bien que ce ne soit pas toujours le cas. Vous pouvez faire rentre beaucoup de boucles et de récursions dans un petite quantité de code.)

[ob]However, note that I'm not saying <span name="simple">simple code</span> takes
less time to *write*. You'd think it would since you end up with less total
code, but a good solution isn't an accretion of code, it's a *distillation* of
it.[oe]

Cependant, notez que je ne dis pas que le <span name="simple">code simple</span> prend moins de temps à *écrire*. Vous devriez penser que ça le devrait car vous finirez avec moins de code au total, mais une bonne solution n'est pas une accumulation de code, c'est une *distillation* de code.

<aside name="simple">

[ob]Blaise Pascal famously ended a letter with, "I would have written a shorter
letter, but I did not have the time."[oe]

Blaise Pascal a fameusement finit une lettre par, "J'aurais dû écrire une lettre plus courte, mais je n'en avais pas le temps." 

[ob]Another choice quote comes from Antoine de Saint-Exupery: "Perfection is
achieved, not when there is nothing more to add, but when there is nothing left
to take away."[oe]

Une autre citation de choix vient de Antoine de Saint-Exupéry : "La perfection est atteinte, non pas lorsqu'il n'y a plus rien à ajouter, mais lorsqu'il n'y a plus rien à retirer."

[ob]Closer to home, I'll note that every time I revise a chapter in this book, it
gets shorter. Some chapters are tightened by 20% by the time they're done.[oe]

Plus proche de la maison, je noterai qu'à chaque fois que je révise un chapitre de ce livre, il devient plus court. Quelques chapitres sont rétrécis de 20% par rapport au début.

</aside>

[ob]We're rarely presented with an elegant problem. Instead, it's a pile of use
cases. You want the X to do Y when Z, but W when A, and so on. In other words, a
long list of different example behaviors.[oe]

Nous sommes rarement en présence d'un problème élégant. A la place, c'est une pile de cas d'utilisation. Vous voulez que le X fasse Y quand Z, mais W quand A, et ainsi de suite. En d'autres termes, une liste de différents exemples de comportements.

[ob]The solution that takes the least mental effort is to just code up those use
cases one at a time. If you look at novice programmers, that's what they often
do: they churn out reams of conditional logic for each case that popped into
their head.[oe]

La solution qui prend le moins de d'effort mental est de coder juste ces cas d'utilisation les uns après les autres. Si vous regardez les programmeurs débutants, c'est ce qu'ils font souvent : ils pondent des masses de conditions logiques pour chaque cas auquel ils pensent.

[ob]But there's nothing elegant in that, and code in that style tends to fall over
when presented with input even slightly different than the examples the coder
considered. When we think of elegant solutions, what we often have in mind is a
*general* one: a small bit of logic that still correctly covers a large space of
use cases.[oe]

Mais il n'y a rien d'élégant là-dedans, et coder avec ce style tend à ne pas fonctionner quand on passe des entrées légèrement différentes des exemples considérés par le codeur. Quand nous pensons à des solutions élégantes, ce que nous avons à l'esprit est une solution *générale* : un petit peu de logique qui couvre encore correctement un grand espace de cas d'utilisation.

[ob]Finding that is a bit like pattern matching or solving a puzzle. It takes effort
to see through the scattering of example use cases to find the hidden order
underlying them all. It's a great feeling when you pull it off.[oe]

Trouver cela est un petit peu comme de la recherche de motif ou de la résolution de casse-têtes. Ca demande un effort pour voir parmi l'éparpillement des cas d'utilisation exemples afin de trouver la règle cachée là-dessous. C'est un bonheur quand vous la découvrez.

[ob]## Get On With It, Already[oe]

## Passons à autre chose, déjà

[ob]Almost everyone skips the introductory chapters, so I congratulate you on making
it this far. I don't have much in return for your patience, but I'll offer up a
few bits of advice that I hope may be useful to you:[oe]

Presque tout le monde zappe les chapitres d'introduction, donc je vous félicite d'être aller si loin. Je n'ai pas grand chose à vous donner en retour de votre patience, mais je donnerai quelques conseils que j'espère utiles pour vous :

* [ob]Abstraction and decoupling make evolving your program faster and easier, but
    don't waste time doing them unless you're confident the code in question needs
    that flexibility.[oe]

 *  L'abstraction et le découplage rend l'évolution de votre programme plus rapide et plus facile, mais ne gaspillez pas votre temps à les mettre en oeuvre à moins que vous soyez confiant dans le fait que le code a besoin de flexibilité.

* [ob]<span name="think">Think</span> about and design for performance throughout
    your development cycle, but put off the low-level, nitty-gritty optimizations
    that lock assumptions into your code until as late as possible.[oe]

 *  <span name="think">Réfléchir</span> et concevoir pour la performance du début à la fin du développement, mais remettez au plus tard possible le bas niveau, les optimisations concrètes qui bloquent les possibilités dans votre code.

<aside name="think">

[ob]Trust me, two months before shipping is *not* when you want to start worrying
about that nagging little "game only runs at 1 FPS" problem.[oe]

Croyez moi, deux mois avant la sortie du jeu n'est pas le moment où vous voudriez commencer à vous inquiéter de ce petit problème tenace qu'est "mon jeu tourne à 1 FPS".

</aside>

* [ob]Move quickly to explore your game's design space, but don't go so fast that
     you leave a mess behind you. You'll have to live with it, after all.[oe]

 *   Changer rapidement pour explorer l'espace de votre concept de jeu, mais n'allez pas trop vite pour ne pas laisser un chaos derrière vous. Après tout, vous aurez à vivre avec après.

* [ob]If you are going to ditch code, don't waste time making it pretty. Rock
     stars trash hotel rooms because they know they're going to check out the
     next day.[oe]

* Si vous allez jeter du code, ne gaspillez pas votre temps à le rendre beau. Les rocks stars détruisent les chambres d'hôtel par ce qu'ils savent qu'il se barrent le lendemain.

* [ob]But, most of all, **if you want to make something fun, have fun making
     it.**[oe]

 * Mais, la chose la plus importante, **si vous voulez faire quelque chose d'amusant, amusez vous à le faire.**