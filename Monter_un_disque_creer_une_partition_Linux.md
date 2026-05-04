# 🖥️ Admin Linux : Monter un disque et créer une partition

## 📚 Sommaire

- [📸 Captures d’écran](#-captures-décran)
- [🔌 1. Disque branché et détection](#-1-disque-branché-et-détection)
- [🛠️ 2. Partitionnement avec cfdisk](#-2-partitionnement-avec-cfdisk)
- [📝 3. Création et écriture de la partition](#-3-création-et-écriture-de-la-partition)
- [💾 4. Enregistrement des modifications](#-4-enregistrement-des-modifications)
- [🔍 5. Vérification du disque](#-5-vérification-du-disque)
- [📂 6. Création du système de fichiers](#-6-création-du-système-de-fichiers)
- [📁 7. Création du point de montage](#-7-création-du-point-de-montage)
- [🔗 8. Montage du disque](#-8-montage-du-disque)
- [🔐 9. Automatiser le montage au démarrage](#-9-automatiser-le-montage-au-démarrage)
- [🔄 10. Vérification après redémarrage](#-10-vérification-après-redémarrage)




![Capture d'écran 2025-06-15 155803](https://github.com/user-attachments/assets/f964b9c5-c8c0-4393-8e48-6884cdf53f9f)  

![Capture d'écran 2025-06-15 155817](https://github.com/user-attachments/assets/70643438-6ca5-4cef-ad41-51d364bd8b8e)  

![Capture d'écran 2025-06-15 160220](https://github.com/user-attachments/assets/6819e7ae-0667-42ca-a2a6-637820a40af7)  
![Capture d'écran 2025-06-15 160227](https://github.com/user-attachments/assets/a2977e27-2a57-4498-b6e2-d11451a6c40b)  
![Capture d'écran 2025-06-15 160242](https://github.com/user-attachments/assets/49aed69f-ca9e-4660-acae-efdd78f7020b)  
![Capture d'écran 2025-06-15 160401](https://github.com/user-attachments/assets/033f81e5-b9d1-4d73-bf47-59f682c91217)  
![Capture d'écran 2025-06-15 160459](https://github.com/user-attachments/assets/db577207-2fe2-4bf4-a64c-c0a3b83c979e)  

## 🔌 1. Disque branché et détection  
Le disque est créé et "branché". On peut démarrer la machine.  
Avec la commande suivante, on voit qu'il est physiquement détecté : `lsblk`  

![Capture d'écran 2025-06-15 160703](https://github.com/user-attachments/assets/71ab4fc8-8a9e-4680-9b62-87e1e383dde9)  

## 🛠️ 2. Partitionnement avec cfdisk  
Puis avec `cfdisk`, donc en mode "semi graphique" on va le partitionner :  
( On peut le faire aussi avec "fdisk", en CLI pur)  
![Capture d'écran 2025-06-15 161024](https://github.com/user-attachments/assets/2bbd8ca3-a9ce-4aaa-b5ec-ae205604b80f)  

On choisit notre table de partition. Ici on prend "GPT", car c'est le nouveau standard, plus fiable, utilisé avec les systèmes modernes (Windows 10/11, Linux, macOS).  
> ℹ️ MBR (aussi appelé "DOS partition table") : ancien standard, limité à 4 partitions primaires et 2 To de disque.)  

![Capture d'écran 2025-06-15 161034](https://github.com/user-attachments/assets/472a23f4-cd65-4dcf-8457-f7c6262d3f91)  

## 📝 3. Création et écriture de la partition  
On écrit sur toute la partition dans notre cas :  
![Capture d'écran 2025-06-15 161231](https://github.com/user-attachments/assets/8df43488-2751-4718-a096-1df0041edc78)  

## 💾 4. Enregistrement des modifications  
Puis on enregistre la nouvelle table de partitions :  
![Capture d'écran 2025-06-15 161242](https://github.com/user-attachments/assets/ccc2303b-aa94-4977-839d-2bc729d68f1d)  

![Capture d'écran 2025-06-15 161257](https://github.com/user-attachments/assets/97aaf206-cf1d-424b-b4ff-46da80fae1ef)  

## 🔍 5. Vérification du disque  
On verifie que notre disque est bien présent :  
![Capture d'écran 2025-06-15 161318](https://github.com/user-attachments/assets/683fab94-ccd3-4467-a193-67c952d1832c)  

## 📂 6. Création du système de fichiers  
On crée notre système de fichiers et on lui attribue une "étiquette". en `ext4`, qui est le dernier format de fichiers sur linux.  
![Capture d'écran 2025-06-15 162500](https://github.com/user-attachments/assets/82613663-aaf9-4243-aa2f-bccdefffc7fe)  

## 📁 7. Création du point de montage  
On crée un dossier pour y monter notre disque dans le dossier courant de l'utilisateur :  
![Capture d'écran 2025-06-15 163108](https://github.com/user-attachments/assets/26a367e4-bd82-46db-8468-5ddef8ae6a1b)  

![Capture d'écran 2025-06-15 163204](https://github.com/user-attachments/assets/79e193a3-cb53-4497-9148-d95ff36e4620)  

## 🔗 8. Montage du disque  
Je monte le disque (le disque "sdb1" dans mon dossier "data" qui se trouve dans mon dossier local" :  
![Capture d'écran 2025-06-15 164150](https://github.com/user-attachments/assets/20810fed-71ad-43b0-a0ab-19c02e4610f7)  

## 🔐 9. Automatiser le montage au démarrage  
La commande "blkid", permet lister les périphériques de stockage et notamment de lister les "UUID". Je vais ensuite envoyer la ligne concerant mon "sdb1" dans le fichier "/etc/fstab" pour pouvoir redémarrer le disque automatiquement au démarrage.  
![Capture d'écran 2025-06-15 171541](https://github.com/user-attachments/assets/30f6b6f4-738d-4f41-9fa2-8c1ca6910af4)  

### Edition du fichier `/etc/fstab` :  
![Capture d'écran 2025-06-15 171602](https://github.com/user-attachments/assets/281e8ac1-a531-49f6-8004-188cf9e7d2bb)  

Ne garder que les parties recnonnues par le fichier, on peut s'inspirer des lignes du dessus pour la syntaxe :  
![Capture d'écran 2025-06-15 171707](https://github.com/user-attachments/assets/fdeba528-52b9-493b-ba0d-09958d7f21a5)  

## 🔄 10. Vérification après redémarrage  
Après redémarrage du PC, on peut effectuer la commande "mount" avec un filtre pour récupérer les disque "sd*", et on voit qu'il est bien actif au lancement de la machine :  
![Capture d'écran 2025-06-15 172155](https://github.com/user-attachments/assets/52af8379-8b6c-4af1-bada-0897d5ca78d2)  

