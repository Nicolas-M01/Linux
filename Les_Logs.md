# Les-logs

## 1. Installe un serveur web (Apache ou Nginx) sur une machine virtuelle Linux  
Vérifier que le serveur Apache2 est installé :  
![Capture d'écran 2024-12-16 110607](https://github.com/user-attachments/assets/6fea1596-8373-41fa-b78f-6759523f8df8)


Le retour de la commande indique que le serveur est installé. :white_check_mark:  

---
## 2. Configure le logging pour enregistrer les accès et les erreurs  

Par défaut, le fichier de configuration d'Apache se trouve dans ``/etc/apache2/sites-available/000-default.conf``.  
Il faut ensuite s'assurer que ces 2 lignes sont décommentées, elles enregistrent les erreurs d'Apache pour la première et les requêtes pour la deuxième :  
``ErrorLog ${APACHE_LOG_DIR}/error.log``  
``CustomLog ${APACHE_LOG_DIR}/access.log combined``  

Dans le cas où elles sont commentées, il faut décommenter et relancer le service (pour rappel ``systemctl restart apache2``). :white_check_mark:  

---
## 3. Génère du trafic sur le serveur web (utilise des outils comme curl ou un navigateur)  

Pour générer des logs, il faut lancer des requêtes.  
J'ai installé l'outil **"Curl"**, (``apt install curl``) qui permet de faire des requêtes HTTP en lignes de commandes.  
J'ai ensuite lancé plusieurs requêtes :  
``curl http://localhost`` # Cette requête va réussir. Elle sert à vérifier que mon serveur web fonctionne bien.  
``curl http://localhost/blabla`` # Cette requête va échouer, car la page est inextistante.   
:white_check_mark:  

---
## 4. Analyse les logs générés et identifie :  
pour afficher les accès au serveur Apache.  
``cat /var/log/apache2/access.log``  

* ### Une requête réussie (code 200)
 ![Capture d'écran 2024-12-16 113246](https://github.com/user-attachments/assets/f320db61-d341-4024-be44-848470dc76c0)  
La machine locale (`127.0.0.1`) a effectué une requête qui a abouti (code 200). :white_check_mark:   

* ### Une erreur 404 (page non trouvée)  
![Capture d'écran 2024-12-16 113338](https://github.com/user-attachments/assets/9684915a-e52d-4c3e-8c2b-341d5a85ba01)  
La machine locale (`127.0.0.1`) a effectué une requête qui n'a pas abouti (code 404), car la page "blabla" n'existe pas. :white_check_mark:  

* ### Les IP les plus fréquentes  
Dans mon cas il n'y a quasiment pas de traffic.  
![Capture d'écran 2024-12-16 114144](https://github.com/user-attachments/assets/7e4fe304-5ea3-4241-a3a4-ced03e444d11)  

l'unique adresse est l'adresse locale. Dans le cas d'un nombre élevé on peut utiliser des commandes bash pour trier/filtrer  :  
``awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head``  
(la première avec "awk" permet de ne garder que la première colonne, ensuite on a le chemin du fichier, puis on trie avec "sort", puis "uniq -c" va compter le nombre de lignes de chaque adresse IP, puis "sort -nr" va trier par fréquence (décroissant), puis "head" pour afficher les 10 premières.  
 ![Capture d'écran 2024-12-16 115229](https://github.com/user-attachments/assets/a41706c6-f25e-4226-a988-faba7a57ee3f)  
:white_check_mark:  
