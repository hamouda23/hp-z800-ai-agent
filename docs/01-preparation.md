# Étape 1 : Préparation du Système

## 🎯 Objectif

Vérifier que le HP Z800 est prêt pour l'installation de la stack LLM (Ollama + Open WebUI + RAG).

## ⏱️ Durée Estimée

10-15 minutes

## 📋 Prérequis

- Accès SSH ou console au serveur HP Z800
- Droits sudo
- Connexion Internet

---

## 1.1 Mise à Jour du Système

```bash
# Mettre à jour la liste des paquets
sudo apt update

# Mettre à jour les paquets installés
sudo apt upgrade -y

# Nettoyer les paquets inutiles
sudo apt autoremove -y
```

**📝 Note :** Cette étape peut prendre 5-10 minutes selon les mises à jour disponibles.

---

## 1.2 Vérification du Kernel

```bash
# Afficher la version du kernel
uname -r
```

**✅ Résultat attendu :**
```
6.8.0-40-generic
```

**📸 Capture :** Sauvegardez cette information
```bash
uname -a > ~/hp-z800-ai-agent/logs/kernel-info.txt
```

---

## 1.3 Vérification des Pilotes NVIDIA

```bash
# Vérifier que nvidia-smi fonctionne
nvidia-smi
```

**✅ Résultat attendu :**

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 535.xx.xx    Driver Version: 535.xx.xx    CUDA Version: 12.x   |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P4000        Off  | 00000000:xx:xx.x Off |                  N/A |
|  0%   xx°C    P8    xW / 105W |      0MiB /  8192MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
```

**🔍 Points à vérifier :**
- ✅ GPU détectée : **Quadro P4000**
- ✅ Mémoire GPU : **8192 MiB (8 GB)**
- ✅ Driver Version : **535.xx ou supérieur**
- ✅ CUDA Version : **12.x**

**📸 Capture :**
```bash
nvidia-smi > ~/hp-z800-ai-agent/logs/nvidia-initial.txt
```

**❌ Si nvidia-smi ne fonctionne pas :**
```bash
# Réinstaller les pilotes
sudo ubuntu-drivers autoinstall
sudo reboot
```

---

## 1.4 Vérification de Conda

```bash
# Vérifier la version de Conda
conda --version

# Afficher le chemin d'installation
conda info --base

# Lister les environnements existants
conda env list
```

**✅ Résultat attendu :**
```
conda 23.x.x (ou supérieur)

# base environment : /chemin/vers/conda

# conda environments:
#
base                  *  /chemin/vers/conda
env1                     /chemin/vers/conda/envs/env1
env2                     /chemin/vers/conda/envs/env2
```

**📝 Note :** Vous avez 2 environnements avec différentes versions de Python. On va en créer un nouveau pour le fine-tuning plus tard.

---

## 1.5 Vérification de l'Espace Disque

```bash
# Afficher l'espace disponible sur tous les disques
df -h
```

**✅ Vérifications importantes :**

| Partition | Usage | Espace Libre Requis | Status |
|-----------|-------|---------------------|--------|
| `/` (Système) | Ollama, Docker | **20 GB minimum** | ☐ |
| Disque 2 (Conda/ML) | Modèles, fine-tuning | **50 GB minimum** | ☐ |

**📸 Capture :**
```bash
df -h > ~/hp-z800-ai-agent/logs/disk-space.txt
```

**⚠️ Si l'espace est insuffisant :**
```bash
# Nettoyer les paquets
sudo apt clean
sudo apt autoremove -y

# Nettoyer les logs Docker (si installé)
docker system prune -a

# Vérifier les gros fichiers
du -sh /* 2>/dev/null | sort -h | tail -10
```

---

## 1.6 Vérification de la Mémoire RAM

```bash
# Afficher la mémoire disponible
free -h
```

**✅ Résultat attendu :**
```
              total        used        free      shared  buff/cache   available
Mem:           12Gi        2Gi        8Gi        100Mi       1.5Gi        9Gi
Swap:         2.0Gi          0B       2.0Gi
```

**🔍 Points à vérifier :**
- ✅ Total : **~12 GB**
- ✅ Available : **Au moins 4 GB** (pour Mistral 7B)
- ✅ Swap configuré : **Recommandé 2-4 GB**

**⚠️ Si pas de swap ou swap insuffisant :**
```bash
# Créer un fichier swap de 4GB
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Rendre permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Vérifier
free -h
```

**📸 Capture :**
```bash
free -h > ~/hp-z800-ai-agent/logs/memory-info.txt
```

---

## 1.7 Vérification du Réseau

```bash
# Afficher les interfaces réseau
ip addr show

# Trouver l'IP du serveur (pour accès distant)
ip addr show | grep "inet " | grep -v 127.0.0.1
```

**📝 Notez votre IP :** `192.168.x.x` ou `10.x.x.x`

**📸 Capture :**
```bash
ip addr show > ~/hp-z800-ai-agent/logs/network-info.txt
```

**🌐 Test de connectivité Internet :**
```bash
# Ping Google DNS
ping -c 4 8.8.8.8

# Test résolution DNS
ping -c 4 google.com
```

---

## 1.8 Installation des Dépendances de Base

```bash
# Installer les outils essentiels
sudo apt install -y \
    curl \
    wget \
    git \
    build-essential \
    ca-certificates \
    gnupg \
    lsb-release \
    net-tools
```

---

## 1.9 Vérification des Ports

```bash
# Vérifier que les ports nécessaires sont libres
sudo netstat -tlnp | grep -E ':(3000|11434)'
```

**✅ Résultat attendu :** Aucune sortie (les ports sont libres)

**❌ Si les ports sont occupés :**
```bash
# Identifier le processus
sudo lsof -i :3000
sudo lsof -i :11434

# Arrêter le processus si nécessaire
sudo systemctl stop nom-du-service
```

---

## 1.10 Création de la Structure de Logs

```bash
# Créer le dossier de logs si pas déjà fait
mkdir -p ~/hp-z800-ai-agent/logs

# Créer un fichier de résumé
cat > ~/hp-z800-ai-agent/logs/preparation-summary.txt << EOF
========================================
Préparation HP Z800 - $(date)
========================================

Kernel: $(uname -r)
NVIDIA Driver: $(nvidia-smi --query-gpu=driver_version --format=csv,noheader)
CUDA Version: $(nvidia-smi | grep "CUDA Version" | awk '{print $9}')
Conda Version: $(conda --version)
Python (base): $(python --version 2>&1)

Disque système libre: $(df -h / | awk 'NR==2 {print $4}')
RAM disponible: $(free -h | awk 'NR==2 {print $7}')

IP du serveur: $(ip addr show | grep "inet " | grep -v 127.0.0.1 | awk '{print $2}' | head -1)

========================================
EOF

# Afficher le résumé
cat ~/hp-z800-ai-agent/logs/preparation-summary.txt
```

---

## ✅ Checklist de Validation

Cochez chaque élément avant de passer à l'étape suivante :

```bash
# Exécutez cette commande pour valider tout automatiquement
cat > ~/hp-z800-ai-agent/logs/validation.sh << 'EOF'
#!/bin/bash
echo "=== Validation de la Préparation ==="
echo ""

# 1. Système à jour
echo -n "✓ Système à jour: "
apt list --upgradable 2>/dev/null | grep -q "Listing..." && echo "OK" || echo "ERREUR"

# 2. NVIDIA
echo -n "✓ NVIDIA détectée: "
nvidia-smi > /dev/null 2>&1 && echo "OK" || echo "ERREUR"

# 3. Conda
echo -n "✓ Conda installé: "
conda --version > /dev/null 2>&1 && echo "OK" || echo "ERREUR"

# 4. Espace disque (/)
DISK_FREE=$(df / | awk 'NR==2 {print $4}')
echo -n "✓ Espace disque /: "
[ $DISK_FREE -gt 20000000 ] && echo "OK ($DISK_FREE KB)" || echo "ATTENTION ($DISK_FREE KB)"

# 5. Mémoire disponible
MEM_AVAIL=$(free | awk 'NR==2 {print $7}')
echo -n "✓ Mémoire disponible: "
[ $MEM_AVAIL -gt 4000000 ] && echo "OK ($MEM_AVAIL KB)" || echo "ATTENTION ($MEM_AVAIL KB)"

# 6. Ports libres
echo -n "✓ Port 3000 libre: "
sudo netstat -tlnp | grep -q ":3000" && echo "OCCUPÉ" || echo "OK"

echo -n "✓ Port 11434 libre: "
sudo netstat -tlnp | grep -q ":11434" && echo "OCCUPÉ" || echo "OK"

echo ""
echo "=== Validation Terminée ==="
EOF

chmod +x ~/hp-z800-ai-agent/logs/validation.sh
~/hp-z800-ai-agent/logs/validation.sh
```

**Checklist manuelle :**

- [ ] Système Ubuntu 22.04 à jour
- [ ] Kernel 6.8.0 ou supérieur
- [ ] nvidia-smi fonctionne
- [ ] Quadro P4000 détectée (8 GB)
- [ ] Conda installé et fonctionnel
- [ ] Au moins 20 GB libres sur /
- [ ] Au moins 50 GB libres sur Disque ML
- [ ] Au moins 4 GB RAM disponible
- [ ] Swap configuré (2-4 GB)
- [ ] Connexion Internet OK
- [ ] Ports 3000 et 11434 libres
- [ ] IP du serveur identifiée

---

## 📊 Résumé de la Configuration

À la fin de cette étape, vous devriez avoir :

```
HP Z800 Configuration:
├── OS: Ubuntu Server 22.04 LTS (kernel 6.8.0-40)
├── GPU: NVIDIA Quadro P4000 (8GB) ✅
├── RAM: ~12 GB (4+ GB disponible) ✅
├── Disk: 20+ GB libre sur / ✅
├── Conda: Installé avec 2 environnements ✅
├── Network: IP identifiée ✅
└── Ports: 3000 et 11434 disponibles ✅
```

---



---

## 🆘 Problèmes Courants

### Problème : nvidia-smi ne fonctionne pas

```bash
# Vérifier si les pilotes sont installés
dpkg -l | grep nvidia

# Réinstaller
sudo ubuntu-drivers autoinstall
sudo reboot
```

### Problème : Espace disque insuffisant

```bash
# Identifier les gros dossiers
sudo du -sh /* 2>/dev/null | sort -h | tail -10

# Nettoyer
sudo apt clean
sudo journalctl --vacuum-time=7d
```

### Problème : Mémoire insuffisante

```bash
# Vérifier les processus gourmands
top -o %MEM | head -20

# Créer/augmenter le swap (voir section 1.6)
```

---

**📝 N'oubliez pas de sauvegarder tous les logs dans `~/hp-z800-ai-agent/logs/`**
