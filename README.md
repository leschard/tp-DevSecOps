# TP Sécurité DevOps

## I) Exécuter un container

Téléchargez l'image `nginx`

```bash
docker pull nginx
```

Lancez le container `nginx`

```bash
docker run -d -p 80:80 nginx

```

* `-d` exécute le container en arrière plan.
* `-p 80:80` permet de _binder_ le port 80 du container sur le port 80 du système hôte.

Envoyez une requête vers le container

```bash
curl http://localhost:80
```

## II) S'échapper d'un container

### Préparation de l'environement

Créez un fichier `key` sur le système hôte

```bash
touch /etc/key
```

Ecrivez une donnée confidentielle dans ce ficher `key`.

```bash
echo "secret" > /etc/key
```

Seul `root` peut lire le fichier `key`

```bash
chmod 600 /etc/key
```

### S'échapper du container

Récupérez l'image `alpine` (une distribution Linux légère)

```bash
docker pull alpine
```

Lancez un container à base de l'image `alpine`. La configuration `-v /:/mnt` permet de monter le répertoire `/` du système hôte à l'emplacement `/mnt` à l'intérieur du container.

```bash
docker run -it -v /:/mnt alpine /bin/sh
```

(`-it` nous permet d'avoir un terminal dans le container)

Allez dans le répertoire `/mnt/etc/` du container (ce qui correspond au répertoire `/etc/` du système hôte grâce au montage `-v /:/mnt` effectué).

```bash
cd /mnt/etc/
```

Peut-on afficher le contenu du fichier confidentiel présent sur le système hôte ?

```bash
cat key
```

Quelles sont les permissions du fichier ?

```bash
ls -lah /mnt/etc/key
```

Peut-on afficher les _hashes_ des mots de passe des utilisateurs du système hôte ?

```bash
cat /mnt/etc/shadow
```

Pour passer à la suite, je sors du container en tapant `Contrôle + D`.

## III) Scanner les images des containers

Récupération du scanner Trivy

```bash
docker pull aquasec/trivy
```

Récupération d'une image à tester

```bash
docker pull alpine:3.9.4
```

Scan de l'image par Trivy

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy alpine:3.9.4
```

* `--rm` permet de supprimer le container après éxécution.
* `-v $PWD:/root/.cache/` permet de sauvegarder l'index des vulnérabilités pour éviter d'avoir à le télécharger à chaque nouvelle éxécution.

Scan uniquement des vulnérabilités `HIGH` et `CRITICAL`

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy --severity HIGH,CRITICAL alpine:3.9.4
```

* `--severity` permet de définir le niveau de sévérité des vulnérabilités à rechercher.

Scan uniquement des vulnérabilités `HIGH`

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy --severity HIGH alpine:3.9.4
```

Scan uniquement des vulnérabilités `CRITICAL`

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy --severity CRITICAL alpine:3.9.4
```

## IV) Restrictions de _capabilities_ dans les containers

### IV.1) Capability CHOWN

Exécutez un container avec l'image `alpine`

```bash
docker run --rm -it alpine /bin/sh
```

Quel utilisateur suis-je à l'intérieur du container ?
Puis-je modifier l'utilisateur propriétaire de `/etc/shadow` ?

```bash
id
chown nobody /etc/shadow
```

Je sors du container en tapant `Contrôle + D`.

Exécutez un container avec l'image `alpine` avec l'option `--cap-drop CHOWN`.

```bash
docker run --rm -it --cap-drop CHOWN alpine /bin/sh
```

* `--cap-drop CHOWN` permet de retire les _capabilities_ `CHOWN` (changer de propriétaire).

Quel utilisateur suis-je à l'intérieur du container ?
Puis-je modifier l'utilisateur propriétaire de `/etc/shadow` ?

```bash
id
chown nobody /etc/shadow
```

### IV.1) Capability NET_RAW

Exécutez un container avec l'image `alpine`

```bash
docker run --rm -it alpine /bin/sh
```

J'installe `tcpdump` dans le container.

```bash
apk add tcpdump
```

Je lance `tcpdump` en interception de l'interface du container `eth0`

```bash
tcpdump -i eth0 -vv
```

Je ferme `tcpdump` en tapant `Contrôle + C`.
Je sors du container en tapant `Contrôle + D`.

```bash
docker run --rm -it --cap-drop NET_RAW alpine /bin/sh
```

J'installe `tcpdump` dans le container.

```bash
apk add tcpdump
```

Je lance `tcpdump` en interception de l'interface réseau `eth0` du container.
Que se passe-t-il ?

```bash
tcpdump -i eth0
```

Et pourtant quel utilisateur suis-je ?

```bash
id
```

Je ferme `tcpdump` en tapant `Contrôle + C`.
Je sors du container en tapant `Contrôle + D`.
