

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

Les droits pour les fichiers doivent être :

```bash
sudo chown root:grafana /etc/grafana/monitoring.key
sudo chmod 640 /etc/grafana/monitoring.key

sudo chmod 644 /etc/grafana/monitoring.crt
```


Puis redémarre Grafana.

Accès :

https://192.168.1.100:3000
Le HTTP n'est plus accessible.

---

## 🔹 Utilisation avec Prometheus

Pour Prometheus, créer un fichier `web.yml` dans `/etc/prometheus/`:  

```yml
tls_server_config:
  cert_file: monitoring.crt
  key_file: monitoring.key
```

Copier les fichiers certificats et clé privée dans le dossier Prometheus :  
``sudo cp monitoring.key /etc/prometheus/``  
``sudo cp monitoring.crt /etc/prometheus/``  

Dans le fichier systemd de Prometheus il faut rajouter :

```bash
--web.config.file=/etc/prometheus/web.yml
```

Les droits pour les fichiers doivent être :

```bash
sudo chown root:prometheus /etc/prometheus/monitoring.key
sudo chmod 640 /etc/prometheus/monitoring.key

sudo chmod 644 /etc/prometheus/monitoring.crt
```

Puis il faut redémarrer systemd et le service :  
```bash
sudo systemctl daemon-reload
sudo systemctl restart prometheus
sudo systemctl status prometheus
```
---

Dorénavant la connexion sur Prometheus s'exécute uniquement en HTTPS.



## 🔹 Utilisation avec Alertmanager





---