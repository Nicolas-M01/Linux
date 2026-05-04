# rsync

## Préparer 2 machines Linux, ici :
1 Ubuntu (Client)
1 Debian (Serveur)

## Installer rsync sur les 2 machines :  
`dpkg -l | grep rsync` pour vérifier s'il est installé.  
``apt install rsync``  


## Installer openSSH serveur/client  
Sur Debian il est déjà installé et activé par défaut. Mais au cas où :  

* Debian sera le serveur :  
`apt install sshd`  
``systemctl enable sshd``  
`systemctl status sshd` : il devrait être activé.  
Configurer fichier de conf. : `/etc/ssh/sshd_config`  
Décommenter et modif `PermitRootlogin yes` pour se connecter en root.  
`systemctl restart sshd`  

* Ubuntu :
  Le client ssh est installé par défaut, mais au cas où `apt install ssh`.
  ``systemctl enable ssh``
  ``systemctl status ssh``
  Il devrait être activé.  

## Mettre les 2 machines sur le même réseau  

* Debian : `nano /etc/network/interfaces`  
* Ubuntu : interface graphique.  

Une fois les machines sur le même réseau et pour Debian : `systemctl restat networking.service`  
* Vérifier avec un ping.  
* Vérifier connection SSH de Ubuntu > Debian :  
`sudo ssh root@192.168.1.250` (dans mon cas sur le réseau 192.168.1.0/24).  

### Tout est prêt à synchroniser !  

## Synchronisation avec Rsync  
Créer des dossiers, fichiers, etc... sur le client  
Dans mon cas dans "Documents", j'ai créé les fichiers : "Skyrim", "Solitude" et "Vivec"  
![Capture d'écran 2025-01-13 163246](https://github.com/user-attachments/assets/d3c38170-f1dd-4d80-a648-00792e68704f)  



Envoyer depusi le client :  
**``sudo rsync -av Documents/ root@192.168.1.250:/root/ > /home/velarion/rsync``**  
en droit admin je lance "rsync", "-av" : "v" pour verbose pour avoir les infos, et "a" pour copier avec récursivité. Je copie le dossier "Documents/" et ce qu'il contient vers "root@192.168.1.250" (machine serveur) dans son dossier "/root/", et j'envoie les infos dans un ficher "rsync" dans "/home/velarion".  
![Capture d'écran 2025-01-13 163020](https://github.com/user-attachments/assets/3e301b0c-2245-4034-9197-17e1c24ac07d)  

Puis côté serveur  
![Capture d'écran 2025-01-13 163323](https://github.com/user-attachments/assets/e618a4e5-6aca-45f1-a090-4bbb96a9e2fe)  



Sur Serveur, supprimer fichier "Skyrim".  
Relancer une synchro sur le client :  
![Capture d'écran 2025-01-13 162811](https://github.com/user-attachments/assets/6fdaebe7-c5dc-484b-8026-48da68379595)


### Ca fonctionne !  
