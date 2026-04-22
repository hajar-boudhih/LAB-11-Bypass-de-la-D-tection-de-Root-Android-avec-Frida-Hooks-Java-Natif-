# LAB-11-Bypass-de-la-D-tection-de-Root-Android-avec-Frida-Hooks-Java-Natif-



<img width="657" height="81" alt="image" src="https://github.com/user-attachments/assets/48d113af-7fbe-4fe6-8ce5-a253dbdf7c6e" />


<img width="447" height="75" alt="image" src="https://github.com/user-attachments/assets/01dc0ca8-d0a2-48a6-8ca6-b8302dd2b430" />


<img width="1249" height="527" alt="image" src="https://github.com/user-attachments/assets/c34e9684-7710-40d8-b881-71128b235030" />

# bypass_root:
Ce script `bypass_root.js` opère au niveau Java (couche Dalvik/ART) pour neutraliser les détections de root les plus courantes dans les applications Android. Il commence par définir une liste de chemins suspects (`suspiciousPaths`) correspondant aux emplacements typiques des binaires `su`, des applications Superuser/SuperSU, et des outils comme `busybox`. La fonction `safeContains()` permet de vérifier la présence de chaînes sensibles sans planter en cas d'erreur. À l'intérieur de `Java.perform()`, le script installe quatre hooks principaux : (1) Il modifie la propriété `Build.TAGS` pour qu'elle retourne `release-keys` au lieu de `test-keys`, ce qui trompe les applications qui vérifient si l'appareil est un build de développement (souvent associé au root). (2) Il hooke la bibliothèque RootBeer (une librairie populaire de détection de root) en interceptant ses méthodes `isRooted()` et `isRootedWithBusyBoxCheck()` pour qu'elles retournent systématiquement `false`. (3) Il intercepte la méthode `File.exists()` de la classe `java.io.File` et retourne `false` pour tous les chemins suspects, empêchant l'application de détecter la présence de binaires root sur le système de fichiers. (4) Il hooke toutes les surcharges de `Runtime.exec()` pour bloquer l'exécution de commandes shell suspectes comme `su`, `which su` ou `busybox`, en remplaçant leur exécution par une commande neutre `echo`. Ce script est particulièrement efficace pour les applications qui utilisent uniquement des détections Java, mais pour Uncrackable Level 3 qui possède également une bibliothèque native (`libfoo.so`) scannant `/proc/self/maps` pour détecter Frida, ce script Java doit être complété par des hooks natifs comme `bypass_native.js` et `anti_frida.js` pour une protection complète.

<img width="794" height="86" alt="image" src="https://github.com/user-attachments/assets/7ed9cc60-bd4f-4166-bacf-0b52ec2a47c9" />



<img width="809" height="558" alt="image" src="https://github.com/user-attachments/assets/8eae90f4-b93a-4489-88b5-742fc3b6f06e" />


<img width="453" height="826" alt="image" src="https://github.com/user-attachments/assets/0f5bf95f-92f4-44ad-ba75-a5edad6c950b" />

# bypass_native:

Ce script `bypass_native.js` opère au niveau natif (C/C++) pour interceptez les appels système que les applications Android utilisent pour détecter la présence de binaires root ou de points de montage suspects. Il fonctionne en hookant cinq fonctions critiques de la libc : `open`, `openat`, `access`, `stat` et `lstat`, qui sont toutes utilisées pour vérifier l'existence de fichiers ou dossiers sensibles. Le script définit une liste de chemins suspects (`SUS`) contenant les emplacements classiques des binaires `su` (superuser) et `busybox`, ainsi que les fichiers de montage `/proc/mounts` et `/proc/self/mounts` qui peuvent révéler la présence de Magisk ou d'autres outils root. La fonction `isSuspiciousPath()` lit le chemin depuis la mémoire (`readCString()`) et vérifie s'il correspond à un chemin suspect. La fonction `hookFunc()` utilise `Interceptor.attach()` pour surveiller chaque appel système : à l'entrée (`onEnter`), elle examine le paramètre contenant le chemin (le premier argument pour `open`, `access`, `stat`, `lstat` ; le deuxième pour `openat`) et si ce chemin est suspect, elle stocke cette information (`this.block = true`). À la sortie (`onLeave`), si l'appel a été marqué comme bloqué, elle remplace la valeur de retour par `-1` (code d'erreur standard indiquant que le fichier n'existe pas ou n'est pas accessible), ce qui trompe l'application en lui faisant croire que les binaires root ou les points de montage suspects sont absents. Ce script est crucial pour Uncrackable Level 3 car sa bibliothèque native `libfoo.so` scanne précisément `/proc/self/maps` et d'autres chemins pour détecter Frida, et en bloquant ces appels système, on empêche la bibliothèque de lire ces informations sensibles, complétant ainsi la protection Java déjà mise en place.

<img width="1256" height="577" alt="image" src="https://github.com/user-attachments/assets/5e742660-2a25-4e5f-8d77-4fe2d040761d" />

<img width="806" height="315" alt="image" src="https://github.com/user-attachments/assets/53edb4b2-9325-4160-86bf-6cf42289e49b" />

<img width="760" height="600" alt="image" src="https://github.com/user-attachments/assets/8e610dc1-0e2d-4760-89c7-45a4ff65c95a" />

# anti_frida:

Ce script `anti_frida.js` est conçu pour empêcher une application Android de détecter la présence de Frida en masquant deux indicateurs courants : les variables d'environnement et les ports réseau utilisés par Frida. La première partie hooke la méthode `System.getenv()` pour intercepter toute tentative de lecture d'une variable d'environnement contenant le mot "frida" (comme `FRIDA_SERVER`) et retourne `null` à la place de la valeur réelle, faisant croire à l'application que Frida n'est pas présent. La deuxième partie hooke la méthode `Socket.connect()` pour bloquer toute tentative de connexion aux ports 27042 et 27043 (les ports par défaut de Frida-server) en levant une exception `Connection refused`, ce qui empêche l'application de confirmer la présence de Frida via un test réseau. Ce script est particulièrement utile pour des challenges comme Uncrackable Level 3 qui cherchent activement à détecter Frida via ces méthodes, mais il ne couvre pas toutes les détections possibles (comme le scan mémoire ou la recherche de processus), d'où la nécessité de le combiner avec d'autres hooks natives pour une protection complète.

<img width="1094" height="562" alt="image" src="https://github.com/user-attachments/assets/4182188c-fe0d-4110-b4ff-756c9d547bb6" />



