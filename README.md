# Snake-EI3-2024

## But du jeu
Ce jeu reprend les base du célèbre jeu ​Snake ​en plaçant dans une arène deux serpents devant se déplacer en évitant les murs, l’adversaire ainsi que lui-même. Vous devrez ainsi écrire le code permettant de contrôler l’un de ces serpents et faire en sorte que celui-ci reste en vie le plus longtemps pour gagner la partie.
C’est un concours de programmation, où on fera s’affronter les programmes de tous les participants dans un grand tournoi. Que le meilleur gagne !!

## Règles de base
- Taille de l’arène : l’arène est de taille variable ​L x H​, transmise par le serveur au début du jeu
- Position de départ : le ​joueur 0 (qui joue le premier) commence à la position (2, H/2) et le ​joueur 1​ à la position ​(L-3, H/2)​.
- Changement de taille du serpent : Chaque serpent gagne une case de taille tous les 10 tours ​(tour 0 compris).

*Tous les mouvements seront exécutés sur un serveur distant qui renverra les informations de mises à jour aux joueurs via l’API fournit. Vous ne pourrez donc pas exécuter de mouvements invalides.*

## Utilisation de l’API
Pour communiquer avec le serveur de jeu vous aurez accès à une API (Application Programming Interface) vous permettant d’envoyer et de recevoir les informations utiles au bon déroulement de la partie.
Dans cette API en langage C, on vous fournit le code source nécessaire pour communiquer avec le serveur (quelques fonctions simples pour initier la communication, démarrer une partie, jouer un coup, recevoir le coup de l’adversaire, etc.)

## Repository
Ce repository contient les 4 fichiers nécessaires pour jouer au Snake en utilisant le serveur (CGS).
Ces fichiers ne doivent pas être modifiés, mais juste utilisés. Vous n'aurez qu'à inclure `snakeAPI.h` dans vos programmes et utiliser les fonctions qui y sont définis (le code est dans `snakeAPI.c` qui lui-même dépend de `clientAPI.c` et `clientAPI.h`). Il vous faudra donc aussi compiler ces deux fichiers `.c` et les linker pour faire votre exécutable

## API
Les fonctions sont détaillées dans les commentaires du fichier `snakeAPI.h`
### `void connectToServer(char* serverName, int port, char* name)`
Initialise la connexion avec le serveur (ou quitte le programme si elle ne peut être établie).
- `serverName` ​: Adresse du serveur 
- `port` ​: Port du serveur de jeu (Serveur de test : ​8080​)
- `name` ​: Nom de votre joueur (Attention, le nom doit être unique et identifiable pour chaque participant​)

Le serveur de jeu tourne sur la machine `pc5039` (port 1234), qui est accessible depuis les machines de l'école. Si vous voulez utiliser votre propre machine, il faudra mettre en place un tunnel SSH en passant par la passerelle de l'école `pc5056.polytech.upmc.fr` (cela se fait avec la commande `ssh -N -p 22 <login>@pc5056.polytech.upmc.fr -L 1234:pc5039:1234&` où `<login>` est votre login de Sorbonne Université; le mot de passe associé doit vous être demandé au moment de créer le tunnel)

### `void closeConnection()` 
Ferme la connection avec le serveur (doit être effectuée pour éviter tout problème)


### `void waitForSnakeGame(char* gameType, char* gameName, int* sizeX, int* sizeY, int* nbWalls)`
Fonction bloquante attendant que le serveur démarre le jeu (attente des participants). Cette fonction donne les informations nécessaires au démarrage de la partie, en remplissant les variables associées.
- `gameType` ​: Commande de création de la partie : on peut préciser si on veut participer à un tournoi ou jouer contre un bot existant. On peut peut aussi préciser des paramètres spéciaux afin de reproduire certaines conditions. C'est une chaine de caractère sous la forme “COMMANDE key1=val1 key2=val2 ...” (exemple : "TRAINING RANDOM_PLAYER difficulty=2 timeout=100 seed=123 start=0")
  - la commande peut être `TRAINING <BOT>` où <BOT> est le nom d'un bot (pour l'instant ​RANDOM_PLAYER ​ou SUPER_PLAYER​) ou bien “TOURNAMENT <xxxxx>“ pour participer au tournoi <xxxxx> (il vous sera possible de créer un tournoi depuis le serveur)
  - les clés peuvent prendre les valeurs suivantes:
    - `difficulty` : indique le niveau de difficulté entre 0 et 3 (relatif au nombre de murs dans l’arène)
    - `timeout` indique le temps (en seconde) avant que le serveur nous considère comme perdant (on peut le fixer quand on joue contre un bot pour s'entraîner,
il sera fixer à 10s pour le tournoi)
    - `start` indique le numéro du joueur qui commence (ici le Joueur 0, c’est-à-dire vous)
    - `seed` initialise à une certaine valeur la graine du générateur de nombre aléatoire. Fixer cette valeur permet de rejouer toujours la même partie (utile quand on déboggue son programme pour rejouer les mêmes parties, on peut récupérer la seed d'une partie en gardant les 6 derniers chiffres hexadécimaux de son numéro de partie)
- `gameName` ​: La variable dans laquelle la fonction donnera le nom de la partie
- `sizeX` ​: La variable dans laquelle la fonction donnera la largeur de l'arène
- `sizeY` ​: La variable dans laquelle la fonction donnera la hauteur de l'arène
- `nbWalls` ​: La variable dans laquelle la fonction donnera le nombre de murs de l'arène (excluant les bords de l’arène)

### `int getSnakeArena(int* walls)`
Récupère l’emplacement des murs de l’arène et le joueur qui commence.
- `walls` ​: Reçoit un tableau de taille (​4 x nbWalls​) contenant l’emplacement murs sous la forme suivante (case1.x, case1.y, case2.x, case2.y) qui indique un mur entre la case1 et la case2
Retourne 0 si vous commencez, 1 sinon

### `t_return_code getMove(t_move* move)`
Attend le mouvement de l’adversaire depuis le serveur.
- `move` ​: La variable dans laquelle la fonction donnera la direction du mouvement de l’adversaire (NORTH, WEST, SOUTH, EAST)
Retourne le type de mouvement (NORMAL_MOVE, WINNING_MOVE, LOSING_MOVE)​ qui nous indique l’adversaire a gagné, perdu ou si le jeu continue

### `t_return_code sendMove(t_move move)`
Envoie votre mouvement au serveur.
- `move` ​: La direction dans laquelle vous souhaitez vous déplacer (NORTH, WEST, SOUTH, EAST)
Retourne le type de mouvement (NORMAL_MOVE, WINNING_MOVE, LOSING_MOVE) pour savoir si ce coup nous a permis de gagner, de perdre ou si le jeu continue


### `void printArena()`
Affiche l’état actuel de l’arène qui est contenu dans le serveur.

### `void sendComment(char* comment)`
Envoie un commentaire au serveur et aux autres joueurs (utile pour vanner l’adversaire).




## 1ère étape : lire `snakeAPI.h` qui détaille les différentes fonctions utilisables

## 2ème étape : écrire un programme capable de jouer mannuellement contre RANDOM_PLAYER
Les différentes étapes sont les suivantes :
1. établir la connection avec le serveur (grâce à `connectToServer`)
2. récupérer les données d'une partie (taille de l'arène, position des murs, et qui commence)
3. une boucle de jeu pour successivement 'jouer un coup' ou 'récupérer le coup de l'adversaire' (et afficher le jeu entre chaque coup)
  -> pour jouer un coup, vous pouvez vous contenter de demander au clavier quelle est la direction
4. terminer la partie quand un coup est gagnant ou perdant

## 3ème étape : avoir une structure de donnée pour représenter l'arène et les deux snakes
On aura l'occasion d'en discuter lors du prochain CM du 14/05

## 4ème étape : déduire un coup possible
1. déterminer les coups qui sont perdants (on fonce dans un mur)
2. parmi les coups possibles, déterminer celuli qui est le plus intéressant...
  -> et là, votre programme va commencer à être intéressant. On pourra discuter ensemble des différentes stratégies possibles, et des algorithmes pour y arriver
