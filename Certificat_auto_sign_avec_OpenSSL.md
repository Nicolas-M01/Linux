

# Création d'un certificat auto signé avec OpenSSL

Ce certificat est réalisé en CLI, sur une Debian.
Il permet de chiffrer le trafic HTTP en HTTPS.

Algorythme de hashage : sha256
Clé RSA de 4096 bits.
le [alt_names] permet de lier le certificat au serveur 192.168.0.100 uniquement.  




## 🔹 1. Créer le fichier de configuration `monitoring.cnf` :  

```ini
[req]
default_bits = 4096
prompt = no
default_md = sha256
distinguished_name = dn
x509_extensions = v3_req

[dn]
CN = 192.168.0.100

[v3_req]
subjectAltName = @alt_names

[alt_names]
IP.1 = 192.168.0.100
```


---

## 🔹 2. Générer la clé et le certificat

Ici on va créer le certificat avec openssl.  
Cette commande crée un certificat de type X509 en générant le certificat et la clé privée qui va signer le hash du certificat.  
La validité est de 10ans, l'algorythme utilisé pour les clé asymétriques est le RSA en 4096 bits.  
Les fichier de sortie seront :  
* Le certificat : `monitoring.crt`  
* La clé privée : `monitoring.key`  
On indique le fichier de conf du dessus pour qu'il le remplisse avec l'IP et les autres détails concernant le chiffrement.  

```bash
openssl req -x509 -nodes -days 3650 \
-newkey rsa:4096 \
-keyout monitoring.key \
-out monitoring.crt \
-config monitoring.cnf
```

✅ **Le résultat :**


![alt text](<Images/Capture d'écran 2026-06-18 152523.png>)

--- 

## 🔹 3. Vérifier le certificat

>**`openssl x509 -in monitoring.crt -text -noout`**  
On y voit particulièrement les infos du début, la clé publique, et la signature.  

![alt text](<Images/Capture d'écran 2026-06-18 154351.png>)
![alt text](<Images/Capture d'écran 2026-06-18 154407.png>)

---

## 🔹 Utilisation avec Grafana

Pour Grafana, modifie `grafana.ini` (dans `/etc/grafana/`), il faut décommenter et modifier, en prenant soin de copier la clé privé et le certificat dans le dossier que l'on renseigne (ici `/etc/grafana/`) :  

```ini
[server]
protocol = https
http_port = 3000
cert_file = /etc/grafana/monitoring.crt
cert_key = /etc/grafana/monitoring.key
```

Puis redémarre Grafana.

Accès :

https://192.168.1.100:3000


---

## 🔹 Utilisation avec Prometheus




---

## 🔹 Utilisation avec Alertmanager





---