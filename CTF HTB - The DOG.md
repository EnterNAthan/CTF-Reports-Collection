# CTF Hack the Box - The DOG

target = 10.10.11.58

## Recon Part

Comme tout bon CTF faut se faire la main sur les services lancées 

- nmap 

````bat
nmap -Pn -p- --min-rate 2000 -sC -sV target
````

````bat
Starting Nmap 7.93 ( https://nmap.org ) at 2025-03-13 10:57 CET
Warning: 10.10.11.58 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.58
Host is up (0.095s latency).
Not shown: 60082 closed tcp ports (reset), 5451 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 972ad22c898ad3ed4dac00d21e8749a7 (RSA)
|   256 277c3ceb0f26e962590f0fb138c9ae2b (ECDSA)
|_  256 9388474c69af7216094cba771e3b3beb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 194.56 seconds
````

- On peux observer 2  services lancées SSH et WEB 
- on à des clef SSH public mais pas de vulnérabilité trouvé après d'autre scan --script==vuln
- Je me concentre sur la partit web 
- En parcourant le site web je découvre que c'est un site Backdrop CMS :

![image-20250313112715592](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20250313112715592.png)

Ce CMS est connu pour de nombreuse vulnérabilité mon premier réflexe est de chercher sur exploit db :

![image-20250313112815323](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20250313112815323.png)

quelque exploit intéressant notamment une RCE et j'ai vue dans un rapport récent que plusieurs LFI sont potentiellement faisable. 



- Je continue le Recon avec énumération des sous dossiers :

  ````bat
  dirsearch -r -w /usr/share/wordlists/seclists/Discovery/Web-Content/quickhits.txt -u "http://10.10.11.58/"
  ````

  

![image-20250313112956703](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20250313112956703.png)

Je découvre que le .git est apparent et qu'il y'a potentiellement de la divulgations de données ou de  code source dans le git.



- Je fais un derniers tour dans le robots.txt et je découvre des dossiers très intéressants : 

![image-20250313113154148](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20250313113154148.png)

### Conclusion de la partit reconnaissance

Nous avons découvert de multiples surface d'attaque sur le service web et aucune sur le services ssh étant donnée la difficulté du CTF je vais m'orienté sur la partit WEB pour exploitation : 

-  .git 
- BackDrop  CVE
- Robots.txt et sous dossiers

## Exploitation 

