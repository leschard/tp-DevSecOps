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

## III) Scanner les images des containers

Récupération du scanner Trivy

```bash
docker pull aquasec/trivy
```

Récupération d'une image à tester

```bash
docker pull infoslack/dvwa
```

Scan de l'image par Trivy

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy infoslack/dvwa
```

* `--rm` permet de supprimer le container après éxécution.
* `-v $PWD:/root/.cache/` permet de sauvegarder l'index des vulnérabilités pour éviter d'avoir à le télécharger à chaque nouvelle éxécution.

Scan uniquement des vulnérabilités `HIGH` et `CRITICAL`

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy --severity HIGH,CRITICAL infoslack/dvwa
```

* `--severity` permet de définir le niveau de sévérité des vulnérabilités à rechercher.

Scan uniquement des vulnérabilités `HIGH`

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy --severity HIGH infoslack/dvwa
```

Scan uniquement des vulnérabilités `CRITICAL`

```bash
docker run --rm -v $PWD:/root/.cache/ aquasec/trivy --severity CRITICAL infoslack/dvwa
```
