# Netdata Monitoring

Description
-----------

Ce dépôt contient une configuration minimale pour lancer Netdata en conteneur Docker afin de surveiller en temps réel la machine hôte et les conteneurs. La configuration fournie expose Netdata en mode `host` (accès direct au réseau de la machine) et monte des volumes permettant d'accéder aux informations système nécessaires (proc, sys, logs, docker.sock, etc.).

Prérequis
---------

- Docker (version récente, 20.10+ recommandée)
- Docker Compose (v2 ou le plugin `docker compose`) ou équivalent
- Accès root / sudo pour manipuler Docker et monter des volumes système

Remarque sur la GPU/NVIDIA
--------------------------

Le fichier `docker-compose.yml` contient une section `deploy.resources.reservations.devices` destinée à la réservation de GPU pour Docker Swarm / orchestrateurs. Si vous n'utilisez pas Swarm, cette section est ignorée par `docker compose`. Pour activer les GPU avec Docker classique, utilisez l'option `--gpus` ou configurez le runtime NVIDIA (ex: `runtime: nvidia`) selon votre version et installation NVIDIA Container Toolkit.

Contenu clé de `docker-compose.yml`
----------------------------------

- Service: `netdata` (image `netdata/netdata:latest`)
- Mode réseau: `network_mode: host` (Netdata écoute normalement sur le port `19999` sur l'hôte)
- Volumes persistants définis: `netdataconfig`, `netdatalib`, `netdatacache`
- Volumes montés en lecture seule depuis l'hôte pour collecte de métriques: `/proc`, `/sys`, `/var/log`, `/var/run/docker.sock`, `/` (monté en `:ro,rslave`) etc.
- Capabilités ajoutées: `SYS_PTRACE`, `SYS_ADMIN`
- `security_opt`: `apparmor=unconfined`

Ces options donnent à Netdata un accès étendu pour lire métriques et logs système. Elles sont nécessaires pour la visibilité complète, mais elles augmentent la surface d'attaque — voir la section Sécurité.

Installation et exécution
-------------------------

1. Cloner le dépôt (si ce n'est pas déjà fait):

```bash
git clone <repo-url> netdata_monitoring
cd netdata_monitoring
```

2. Démarrer Netdata en arrière-plan:

```bash
docker compose up -d
```

Ou, si vous utilisez la commande classique `docker-compose`:

```bash
docker-compose up -d
```

3. Vérifier les logs:

```bash
docker compose logs -f netdata
```

4. Accéder à l'interface web Netdata depuis un navigateur:

```
http://<IP_DE_VOTRE_HOTE>:19999
```

Arrêt et suppression
--------------------

Pour arrêter et supprimer les conteneurs créés:

```bash
docker compose down
```

Sécurité et bonnes pratiques
---------------------------

- `network_mode: host` donne un accès réseau complet au conteneur — assurez-vous que le service Netdata est correctement restreint au niveau du pare-feu si exposé vers des réseaux non fiables.
- Les montages en lecture seule réduisent le risque d'écriture mais certains montages donnent un accès sensible aux informations système. Ne lancez pas ce conteneur sur des hôtes multi-tenant non sécurisés.
- Évitez d'exposer Netdata publiquement sans authentification / reverse-proxy + authentification (ex: nginx + OAuth, HTTP basic ou Auth proxy).

Dépannage
---------

- Si Netdata ne démarre pas, vérifiez les logs via `docker compose logs netdata`.
- Si vous avez des erreurs liées aux permissions sur `docker.sock`, assurez-vous que le socket est présent et lisible par le conteneur (le compose actuel monte `docker.sock` en `:ro`).
- Pour utiliser la GPU: installez le NVIDIA Container Toolkit et utilisez la configuration adaptée (`--gpus` ou runtime). La section `deploy` du compose est pour Swarm.

Contribuer
---------

Les contributions sont bienvenues. Ouvrez une issue pour discuter des changements majeurs, puis un merge request/PR pour les corrections et améliorations (ex: ajout d'un reverse-proxy, authentification, dashboards complémentaires, ou automatisation de déploiement).

Licence
-------

Ajoutez ici la licence du projet si vous en avez une (ex: MIT). Par défaut, il n'y a pas de licence incluse.

Contact
-------

Pour toute question, ouvrez une issue dans le dépôt ou contactez le mainteneur du projet.

Fichier principal de configuration
---------------------------------

`docker-compose.yml` — conteneur Netdata avec volumes persistants et accès aux informations système.

---
*README généré automatiquement — modifiez-le pour ajouter des instructions spécifiques à votre infrastructure.*
