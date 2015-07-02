# Live coding Event sourcing et CQRS #

## Étape 1 : Event sourcing ##

### Introduction des évènements ###

On commence par tester la fonctionalité d'ajout d'un joueur : `should_add_one_player()`. Ce qu'on constate c'est que
les assertions `assertThat(result).isTrue()` et `assertThat(game.howManyPlayers()).isEqualTo(1)` ne sont pas
suffisantes : on aimerait aussi tester que le nom du joueur est correctement pris en compte. Or il y a des sorties dans
la console. Ce qu'on pourrait écrire c'est donc quelque chose du genre :

    assertThat(eventStore.events()).containsExactly(
            new PlayerWasAdded("seb"),
            new PlayersCount(1));

On ajoute un champ `@Rule EventStore`. Il réclame une fonction `String -> Event` qu'on peut implémenter dans l'interface
`Event` : `static Event fromLine(String line)`. Commencer par instancier un `UnknownEvent`, puis utiliser
des expressions régulières pour générer les deux évènements.

Transition : nous allons découvrir quelques fonctionnalités d'un event collector à partir des tests.

### L'EventStore doit pouvoir enregistrer les évènements à un moment donné ###

On duplique le test précédent en le nommant `should_add_two_players`. On ajoute l'appel à `game.add("pj")` dans le
_given_. Le test ne passe pas car on ne peut pas décider du moment où l'event collector va collecter.

Appel de `eventsStore.start()` dans le _when_.

### L'EventStore doit pouvoir filtrer les évènements ###

On aimerait constater facilement que le jeu n'interdit pas l'ajout de joueurs ayant le même nom. On duplique le test
précédent en le nommant `should_add_players_with_same_name`. On ajoute deux fois "pj". On met alors dans les events le
filtre sur `PlayerWasAdded.class` pour que notre assertion soit claire.

### L'EventStore doit pouvoir effectuer des actions avant et après traitement ###

On crée un nouveau test en dupliquant le précédent en le nommant `should_be_playable_with_two_players`. Ce test ne va
pas du tout utiliser les events. Donc on aimerait ne pas avoir d'affichage sur la sortie standard. Pour cela on ne va
ajouter les pré et post processeurs seulement quand on active le mode debug. Au tout début du test, on va ajouter
l'appel à `eventStore.debug()` qui devra faire exactement ce qu'on faisait jusqu'à présent par défaut dans le
constructeur de `EventCollector` :

    preProcessors.add(line -> console.format("%50s -> ", line));
    postProcessors.add(console::println);

## Étape 2 : CQRS ##
 
Pourquoi introduire CQRS ? Le code actuel pourrait voir ses méthodes publiques ségrégées en deux parties. On déclare
donc que `Game` implémente les interfaces `Commands` et `Queries` :
 
- `Commands`
   - `boolean add(String)`
   - `void roll(int)`
   - `boolean wasCorrectlyAnswered()`
   - `boolean wrongAnswer()`
- `Queries`
   - `boolean isPlayable()`
   - `int howManyPlayers()`
 
Par rapport aux queries ce qu'on observe ici c'est que `boolean isPlayable()` n'est utilisé que dans les tests. C'est du
code mort. On le supprime. `int howManyPlayers()` n'est utilisé que dans cette classe. Autrement dit elle est privée.
Donc l'interface `Queries` pourrait être vide.
 
Game est donc une implémentation du circuit d'écriture du système.
  
Notre PO nous demande maintenant d'écrire un client qui serait capable de récupérer l'état du jeu à tout moment. Nous
allons donc écrire un nouveau composant qui va effectuer cette tâche en s'abonnant aux évènements écrits par le circuit
d'écriture.
 
## Étape 3 ##

Partir du client graphique et montrer le mode rejoue des évènements depuis le début vs récupère l'état courant du jeu.
