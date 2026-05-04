# Étendre une partition Linux sans LVM

> **VM VMware vSphere — Debian 12 — Table MBR**  
> Cas d'usage : disque étendu côté vSphere, partition non mise à jour

---

## Contexte et prérequis

Ce tutoriel s'applique dans la situation suivante :

- VM VMware vSphere avec Debian 12
- Table de partitions MBR (dos) — pas de LVM, pas de GPT
- Disque étendu côté vSphere mais Linux ne voit pas la nouvelle taille
- Partition racine `/dev/sda1` pleine ou presque pleine
- Swap sur `/dev/sda5` (partition logique dans une étendue `/dev/sda2`)

> ⚠️ **Faites TOUJOURS un snapshot vSphere avant de commencer.** En cas de problème, vous pouvez revenir en arrière en quelques minutes.

> ⚠️ **Notez le secteur de début de sda1 (généralement `2048`).** Cette valeur est critique : ne jamais la changer.

---

## Étape 0 — Vérifier l'état initial

Avant toute manipulation, vérifiez la situation réelle du disque et des partitions :

```bash
lsblk
df -h
fdisk -l /dev/sda
```

Dans la sortie de `fdisk`, notez :
- La taille totale du disque `/dev/sda`
- Le secteur de début de `/dev/sda1` (doit être `2048`)
- Les numéros et tailles de toutes les partitions

---

## Étape 1 — Faire détecter la nouvelle taille par Linux

Après avoir étendu le disque côté vSphere, Linux ne détecte pas automatiquement le changement. Forcez un rescan :

```bash
echo 1 > /sys/class/block/sda/device/rescan
sleep 2
fdisk -l /dev/sda | head -3
```

Vérifiez que le disque affiche maintenant la bonne taille. Si Linux voit toujours l'ancienne taille, tentez le rescan via le bus SCSI :

```bash
for host in /sys/class/scsi_host/host*/; do echo "- - -" > "${host}scan"; done
echo 1 > /sys/class/block/sda/device/rescan
fdisk -l /dev/sda | head -3
```

> 💡 Si après ces commandes le disque affiche toujours l'ancienne taille, un **reboot est nécessaire**. Vérifiez aussi côté vSphere que la tâche d'extension est bien terminée et qu'il n'y a plus de snapshots actifs (ils bloquent l'extension).

---

## Étape 2 — Désactiver le swap

Avant de toucher aux partitions, désactivez le swap. Il sera recréé à la fin.

```bash
swapoff /dev/sda5
```

Vérifiez que le swap est bien désactivé :

```bash
free -h
# La ligne 'Échange' doit afficher 0B
```

---

## Étape 3 — Repartitionner avec fdisk

> ⚠️ Cette étape supprime et recrée les partitions. Le contenu du filesystem n'est **pas** effacé. Ne changez jamais le secteur de début de sda1 (`2048`).

Lancez fdisk :

```bash
fdisk /dev/sda
```

L'avertissement *"disque actuellement utilisé"* est normal — `sda1` est montée en tant que `/`. Continuez.

### 3.1 — Afficher la table actuelle (pour référence)

```
p
```

Notez le secteur de début de `sda1` (`2048`) avant de continuer.

### 3.2 — Supprimer les partitions dans l'ordre

Supprimez d'abord la partition logique swap (`sda5`), puis l'étendue (`sda2`), puis `sda1` :

```
d → 5
d → 2
d → 1
```

### 3.3 — Recréer sda1 plus grande

Créez une nouvelle partition primaire. Remplacez `+97G` par la taille souhaitée selon votre disque (laissez 2-3 Go pour le swap à la fin) :

```
n → p → 1 → 2048 → +97G
```

fdisk détecte une signature ext4 existante et demande si vous voulez la supprimer :

```
Voulez-vous supprimer la signature ? [O]ui/[N]on : N
```

> ⚠️ **Toujours répondre `N` (Non).** Répondre `O` effacerait votre système de fichiers.

### 3.4 — Remettre le flag bootable sur sda1

```
a → 1
```

### 3.5 — Recréer la partition étendue pour le swap

```
n → e → 2 → Entrée → Entrée
```

(Entrée deux fois pour prendre tout l'espace restant jusqu'à la fin du disque)

### 3.6 — Créer la partition logique swap (sda5)

```
n → Entrée → Entrée
```

### 3.7 — Changer le type de sda5 en swap

```
t → 5 → 82
```

### 3.8 — Vérifier puis écrire

Affichez la table finale pour vérifier :

```
p
```

Vérifiez :
- `sda1` : flag `*` (bootable), début à `2048`, type `Linux`
- `sda2` : partition `Étendue`
- `sda5` : type `partition d'échange Linux`

Si tout est correct, écrivez la table :

```
w
```

---

## Étape 4 — Étendre le système de fichiers

La partition est maintenant plus grande mais le filesystem ext4 occupe toujours l'ancien espace. `resize2fs` fonctionne **à chaud** sur une partition montée :

```bash
resize2fs /dev/sda1
```

Le message *"on-line resizing required"* est normal — `/` est monté. L'opération prend quelques secondes.

> 💡 Si vous obtenez une erreur `bad magic number`, le secteur de début de sda1 a peut-être changé. Vérifiez avec `fdisk -l` et comparez avec la valeur initiale (`2048`).

---

## Étape 5 — Recréer et activer le swap

```bash
mkswap /dev/sda5
swapon /dev/sda5
free -h
```

---

## Étape 6 — Mettre à jour /etc/fstab

`mkswap` génère un **nouvel UUID** pour la partition swap. Il faut mettre à jour `/etc/fstab` pour que le swap soit monté automatiquement au démarrage.

Récupérez le nouvel UUID :

```bash
blkid /dev/sda5
```

Regardez l'ancien UUID dans fstab :

```bash
cat /etc/fstab | grep swap
```

Remplacez l'ancien UUID par le nouveau :

```bash
sed -i 's/ANCIEN-UUID/NOUVEL-UUID/' /etc/fstab

# Vérifiez
cat /etc/fstab | grep swap
```

---

## Étape 7 — Vérification finale

```bash
df -h /
lsblk
free -h
swapon --show
```

Résultat attendu :
- `sda1` avec la nouvelle taille et l'espace libre attendu
- Le swap actif sur `sda5`
- L'utilisation du filesystem descendue en dessous de 80%

---

## Récapitulatif des commandes

| # | Action | Commande |
|---|--------|----------|
| 1 | Rescan disque Linux | `echo 1 > /sys/class/block/sda/device/rescan` |
| 2 | Désactiver le swap | `swapoff /dev/sda5` |
| 3 | Supprimer les partitions | `fdisk /dev/sda` → `d 5` → `d 2` → `d 1` |
| 4 | Recréer sda1 + flag boot | `n p 1 2048 +97G` → `N` → `a 1` |
| 5 | Recréer le swap | `n e 2` → `n` → `t 5 82` → `w` |
| 6 | Étendre le filesystem | `resize2fs /dev/sda1` |
| 7 | Recréer le swap | `mkswap /dev/sda5` + `swapon /dev/sda5` |
| 8 | MAJ UUID swap dans fstab | `sed -i 's/ANCIEN/NOUVEAU/' /etc/fstab` |

---

## Après l'opération

1. **Supprimez le snapshot vSphere** — il consomme de l'espace et dégrade légèrement les performances I/O tant qu'il existe.
2. Vérifiez que Prometheus redémarre correctement si le disque plein l'avait impacté.
3. Pensez à configurer des alertes sur l'espace disque pour anticiper la prochaine fois (seuil à 80% recommandé).

---

## Dépannage

**Le disque Linux ne voit pas la nouvelle taille après vSphere**
- Vérifiez que la modification vSphere est bien sauvegardée (tâche terminée dans le panneau des tâches récentes)
- Vérifiez qu'il n'y a plus de snapshots actifs sur la VM (ils bloquent l'extension)
- Vérifiez que c'est bien la bonne VM qui a été modifiée
- En dernier recours : rebootez la VM

**resize2fs retourne une erreur**
- `bad magic number` : le secteur de début de sda1 a changé — revenez au snapshot
- `Device or resource busy` : normal si `/` est monté, resize2fs gère ça automatiquement

**Le système ne démarre plus après l'opération**
- Revertez le snapshot vSphere
- Ou bootez sur un live CD et recréez les partitions avec les bons secteurs de début

---

*Testé sur Debian 12 — VMware vSphere — Table MBR — Mai 2026*
