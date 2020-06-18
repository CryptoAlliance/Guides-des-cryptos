##### Table des matières

- [Prérequis](#Prérequis)
- [Préparation](#Préparation)
- [Récupérer les sources de ALQO](#Récupérer-les-sources-de-ALQO)
- [Compiler les dépendances](#Compiler-les-dépendances)
- [Compiler le daemon et le cli](#Compiler-le-daemon-et-le-cli)
- [Au tour du Raspberry](#Au-tour-du-Raspberry)

# Prérequis

Nous allons voir comment compiler le daemon et le cli du node [ALQO](https://alqo.app "ALQO") pour un Raspberry Pi sous [Raspbian](https://www.raspberrypi.org/downloads/raspbian/ "Raspbian").

Le prérequis sont :

- Avoir Raspbian déjà installé et prêt à l'emploi sur le Raspberry Pi.
- Avoir un PC (ou une machine virtuel ou un container) avec Linux, idéalement Debian (ex : si vous utilisez Raspbian Stretch, utilisez Debian Stretch sur le PC).

# Préparation

Commencez par installer quelques outils nécessaires pour la compilation **sur le PC**.

`sudo apt-get install git automake build-essential libtool autotools-dev autoconf curl g++-arm-linux-gnueabihf pkg-config`

# Récupérer les sources de ALQO

Récupérez le projet à partir de github.

`git clone https://github.com/ALQOCRYPTO/ALQO.git`

Le répertoire du projet porte le nom "ALQO" (original n'est-ce pas ?).

# Compiler les dépendances

Une application fait très souvent appel à des [bibliothèques logicielle](https://fr.wikipedia.org/wiki/Biblioth%C3%A8que_logicielle "bibliothèques logicielle")s afin de ne pas réinventer la roue. Ce sont donc des morceaux de code extérieurs au code source de l'application mais qui sont nécessaires à celui-ci. Dans un premier temps, nous allons donc compiler ces bibliothèques.

Déplacez vous dans le répertoire "depends" du projet.

`cd ALQO/depends`

Lancez la compilation.

`make HOST="arm-linux-gnueabihf" NO_QT=1 -j$(nproc)`

# Compiler le daemon et le cli

Retournez à la racine du répertoire du projet et lancez le script autogen.

```bash
cd ..
./autogen.sh
```

Exécutez le script configure.

`./configure --prefix=$PWD/depends/arm-linux-gnueabihf --enable-qt=no --enable-tests=no`

S'il se termine sans erreur, passez à la compilation proprement dite.

`make -j$(nproc)`

# Au tour du Raspberry

Récupérez les exécutables alqod et alqo-cli fraîchement générés et transférez les sur votre Raspberry (via clef usb, ftp, sftp, …). Ils se trouvent dans le répertoire "src" du projet. Pour ce guide nous avons recréé un dossier ALQO sur le raspberry et placé les exécutables dedans, libre à vous de choisir votre propre organisation.

Assurez vous d'avoir la permission d'exécuter vos exécutables.

`chmod 744 ALQO/alqo*`

Voilà vous êtes prêt à utiliser le daemon et le cli de la même manière que si vous étiez sur un pc en x86.

------------
[![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png "Creative Commons License")](http://creativecommons.org/licenses/by-sa/4.0/ "Creative Commons License")
Cette œuvre est mise à disposition selon les termes de la Licence [Creative Commons Attribution - Partage dans les Mêmes Conditions 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/deed.fr "Licence Creative Commons Attribution - Partage dans les Mêmes Conditions 4.0 International").
