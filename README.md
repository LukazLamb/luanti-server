# luanti-server
Objectif : Mettre en place la fondation capable d'héberger et de gérer de multiples environnements isolés.

🚀 Étape 1 : Préparer la VM et LXC

Avant toute chose, tu as besoin d'une VM Debian (ou autre système) avec LXC installé pour créer les containers. Si tu n’as pas encore installé LXC, voici les commandes à lancer dans ta VM Debian.

Installer LXC dans la VM Debian :

<pre>sudo apt update
sudo apt install lxc lxc-templates lxd -y</pre>


Créer des containers pour Minetest : Pour chaque monde (par exemple, survie, créatif, exploration), crée un container LXC dédié.
Par exemple, pour créer un container pour Minetest Survie :

sudo lxc-create -n luanti-survie -t debian


Si tu veux ajouter d'autres mondes comme créatif, exploration, bedwars, répète la commande pour chaque monde :

sudo lxc-create -n luanti-creatif -t debian
sudo lxc-create -n luanti-exploration -t debian
sudo lxc-create -n luanti-bedwars -t debian

🚀 Étape 2 : Installer Minetest dans les containers

Démarrer les containers : Une fois que les containers sont créés, démarre-les avec :

sudo lxc-start -n luanti-survie
sudo lxc-start -n luanti-creatif
sudo lxc-start -n luanti-exploration
sudo lxc-start -n luanti-bedwars


Vérifie leur état avec :

sudo lxc-ls -f


Installer Minetest dans chaque container :
Connecte-toi à un container via lxc-attach, puis installe Minetest à l'intérieur du container :

sudo lxc-attach -n luanti-survie


Une fois dans le container, installe Minetest :

sudo apt update
sudo apt install minetest-server -y


Répète cette commande pour chaque container.

🚀 Étape 3 : Configurer Minetest pour chaque monde

Configurer le serveur Minetest dans chaque container :
Une fois Minetest installé, configure chaque monde dans /etc/minetest/minetest.conf ou le fichier de configuration global dans /var/lib/luanti/default.conf.

Par exemple, pour un monde survie, tu peux configurer le fichier minetest.conf de cette manière :

server_name = Serveur Survie
port = 30000
bind_address = 0.0.0.0
world = /var/lib/minetest/worlds/survie
max_users = 20
enable_damage = true
enable_pvp = true
creative_mode = false
default_privs = interact, shout


Cela configure le monde survie pour ton serveur Minetest.

Créer et ajouter un monde :
Crée un dossier pour ton monde dans le répertoire worlds/ de Minetest :

mkdir /var/lib/minetest/worlds/survie


Tu peux aussi ajouter une map déjà existante ou la générer en utilisant la commande de Minetest. Assure-toi que le monde contient les fichiers nécessaires, comme world.mt, map.sqlite, etc.

🚀 Étape 4 : Démarrer et tester les serveurs

Redémarrer le service Minetest pour appliquer les nouvelles configurations :

sudo systemctl restart minetest-server


Vérifier que le serveur démarre bien :
Regarde les logs de Minetest dans les fichiers de log (souvent dans /var/log/ ou un répertoire similaire) pour t'assurer qu’il n’y a pas d’erreurs. Utilise :

tail -f /var/log/luanti/minetest.log


Tester la connexion au serveur Minetest depuis ton client :
Ouvre Minetest sur ton client et connecte-toi à ton serveur via l'IP de la VM et le port que tu as configuré (par exemple, 30000).

🚀 Étape 5 : Gérer les serveurs avec un script

Maintenant que tout est configuré, voici un script simple StartService.sh pour démarrer tous tes serveurs (ou un seul) automatiquement.

📄 Script StartService.sh
#!/bin/bash

# ==========================
# SCRIPT : StartService.sh
# ==========================

# Liste des containers disponibles
containers=("luanti-survie" "luanti-creatif" "luanti-exploration" "luanti-bedwars")

echo "Voulez-vous démarrer tous les serveurs ? (o/n)"
read choice

# Option 1 → démarrer TOUT
if [[ "$choice" == "o" || "$choice" == "O" ]]; then
    echo "▶ Démarrage de tous les serveurs..."
    for c in "${containers[@]}"
    do
        echo "⚙ Démarrage du container : $c"
        lxc-start -n "$c"
        sleep 2
        lxc-attach -n "$c" -- systemctl restart luanti-server
        echo "✔ $c lancé"
    done
    echo "🔥 Tous les serveurs sont démarrés."
    exit 0
fi

# Option 2 → un seul serveur
if [[ "$choice" == "n" || "$choice" == "N" ]]; then
    echo "Entrez le nom du container à lancer :"
    read container

    # Vérification du nom
    if [[ ! " ${containers[*]} " =~ " ${container} " ]]; then
        echo "❌ ERREUR : Nom invalide → $container"
        echo "Liste valable : ${containers[*]}"
        exit 1
    fi

    # Démarrer le bon container
    echo "▶ Démarrage de $container ..."
    lxc-start -n "$container"
    sleep 2
    lxc-attach -n "$container" -- systemctl restart luanti-server
    echo "✔ Serveur $container lancé."
    exit 0
fi

# Cas invalide
echo "❌ Réponse inconnue. Utilise : o / n."
exit 1

📌 Pour utiliser le script

Crée un fichier StartService.sh :

nano StartService.sh


Colle le contenu ci-dessus dans le fichier.

Rends le script exécutable :

chmod +x StartService.sh


Exécute-le :

./StartService.sh


Le script te demandera si tu veux démarrer tous les serveurs ou un seul, et il redémarrera le serveur Minetest dans le container approprié.
