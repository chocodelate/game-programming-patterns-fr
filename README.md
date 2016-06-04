Ceci est le dépôt pour le livre [Game Programming Patterns][].
Issues et pull requests sont plus que bienvenue !

## Construire le livre

Le livre est écrit en Markdown (dans `book/`). Un petit script Python (`script/format.py`) convertit cela avec un fichier SASS (`asset/style.scss`) et un template HTML (`asset/template.html`) vers le HTML final (dans `html/`). Pour exécuter le script de format localement, vous aurez besoin d'avoir Python 2.7-XXX, et d'installer Python Markdown, Pygments, et SmartyPants :

    $ pip install markdown
    $ pip install pygments
    $ pip install smartypants

Vous pourriez avoir besoin de `sudo` pour celles-ci. Une fois que c'est fait, vous pouvez exécuter :

    $ python script/format.py

Soyez sûr d'exécuter ceci à partir du répertoire root du dépôt. Cela régénerera tous des chapitres et des intros de section des fichiers HTML. Si vous êtes en train d'éditer des choses, le script peut aussi être exécuté en watch mode :

    $ python script/format.py --watch

Cela surveillera le système de fichier pour les changements des fichier de markdown, SASS, ou template HTML, et les retraitera au besoin.

[game programming patterns]: http://gameprogrammingpatterns.com/
