# Étape 2 : Installation d'Ollama (100% Native)

## 🎯 Objectif

Installer Ollama en mode natif (pas Docker) pour un accès GPU direct et une performance maximale.

## ⏱️ Durée Estimée

5-10 minutes

## 📋 Prérequis

- ✅ Étape 1 (Préparation) complétée
- ✅ Pilotes NVIDIA fonctionnels
- ✅ Accès sudo
- ✅ Connexion Internet

---

## 2.1 Installation d'Ollama

```bash
# Télécharger et installer Ollama (installation native)
curl -fsSL https://ollama.com/install.sh | sh
```

**📝 Ce que fait ce script :**
- Télécharge le binaire Ollama
- L'installe dans `/usr/local/bin/ollama`
- Crée un service systemd `/etc/systemd/system/ollama.service`
- Configure l'utilisateur `ollama`
- Crée le dossier de données dans `/usr/share/ollama`

**⏱️ Temps d'installation :** ~1-2 minutes

**✅ Résultat attendu :**
```
>>> Installing ollama to /usr/local
>>> Downloading ollama-linux-amd64.tar.zst
>>> Creating ollama user...
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
>>> NVIDIA GPU installed.
```

---

## 2.2 Vérification de l'Installation

```bash
# Vérifier la version installée
ollama --version
```

**✅ Résultat attendu :**
```
ollama version is 0.15.2
```

---

## 2.3 Démarrage du Service

```bash
# Démarrer le service Ollama
sudo systemctl start ollama

# Activer le démarrage automatique au boot
sudo systemctl enable ollama

# Vérifier le statut
sudo systemctl status ollama
```

**✅ Résultat attendu :**
```
● ollama.service - Ollama Service
     Loaded: loaded (/etc/systemd/system/ollama.service; enabled)
     Active: active (running) since [DATE]
       Docs: https://ollama.com/docs
   Main PID: xxxxx (ollama)
      Tasks: 12
     Memory: 141.0M
```

**🔍 Points à vérifier :**
- ✅ **Active:** `active (running)` en vert
- ✅ **Loaded:** `enabled` (démarrage auto)
- ✅ Pas de messages d'erreur en rouge

---

## 2.4 Vérification du Port

```bash
# Vérifier qu'Ollama écoute sur le port 11434
sudo ss -tlnp | grep ollama
```

**✅ Résultat attendu :**
```
LISTEN 0  4096  127.0.0.1:11434  0.0.0.0:*  users:(("ollama",pid=xxxxx,fd=3))
```

**📝 Note :** 
- Sur Ubuntu 22.04+, utilisez `ss` au lieu de `netstat`
- Pour l'instant, Ollama écoute uniquement sur `127.0.0.1` (localhost)
- On va le configurer pour l'accès distant à l'étape 3

**💡 Si vous préférez netstat :**
```bash
# Installer net-tools (optionnel)
sudo apt install net-tools -y

# Puis utiliser
sudo netstat -tlnp | grep ollama
```

---

## 2.5 Test de l'API en Local

```bash
# Test simple de l'API
curl http://localhost:11434/api/tags
```

**✅ Résultat attendu :**
```json
{
  "models": []
}
```

**📝 Note :** La liste est vide car aucun modèle n'est encore téléchargé. C'est normal !

**Test supplémentaire :**
```bash
# Vérifier la version de l'API
curl http://localhost:11434/api/version
```

**✅ Résultat attendu :**
```json
{
  "version": "0.15.2"
}
```

---

## 2.6 Vérification de l'Accès GPU

```bash
# Vérifier les logs pour voir si la GPU est détectée
sudo journalctl -u ollama -n 50 --no-pager | grep -i "gpu\|cuda"
```

**✅ Résultat attendu :**
```
inference compute ... name=CUDA0 description="Quadro P4000" 
total="8.0 GiB" available="7.9 GiB"
```

**🎮 Points importants :**
- ✅ GPU détectée : **Quadro P4000**
- ✅ Mémoire disponible : **~7.9 GB / 8 GB**
- ✅ CUDA activé

---

## 2.7 Téléchargement du Premier Modèle (Mistral)

```bash
# Télécharger Mistral 7B (~4.1 GB)
ollama pull mistral
```

**⏱️ Durée :** 5-10 minutes selon votre connexion Internet

**✅ Progression attendue :**
```
pulling manifest
pulling 61e88e884507... 100% ▕████████████████▏ 4.1 GB
pulling 43070e2d4e53... 100% ▕████████████████▏  11 KB
pulling e6836092461f... 100% ▕████████████████▏  42 B
pulling ed11eda7790d... 100% ▕████████████████▏  30 B
pulling f9b1e3196ecf... 100% ▕████████████████▏ 483 B
verifying sha256 digest
writing manifest
success
```

**Vérifier les modèles installés :**
```bash
ollama list
```

**✅ Résultat attendu :**
```
NAME              ID              SIZE      MODIFIED
mistral:latest    6577803aa9a0    4.1 GB    X minutes ago
```

---

## 2.8 Test de Mistral

```bash
# Test rapide en mode interactif
ollama run mistral "Bonjour ! Peux-tu te présenter en une phrase ?"
```

**✅ Mistral devrait répondre en français**

**Pour quitter le mode interactif :**
```bash
/bye
```

**Test via l'API :**
```bash
curl http://localhost:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "Bonjour !",
  "stream": false
}'
```

---

## 2.9 Emplacement des Fichiers

```bash
# Vérifier où Ollama stocke les modèles
ls -lh /usr/share/ollama/.ollama/models/
```

**📁 Structure :**
- **Binaire :** `/usr/local/bin/ollama`
- **Service :** `/etc/systemd/system/ollama.service`
- **Modèles :** `/usr/share/ollama/.ollama/models/`
- **Utilisateur :** `ollama` (créé automatiquement)

**💾 Espace disque utilisé :**
```bash
du -sh /usr/share/ollama/
# ~4.2 GB avec Mistral
```

---

## 2.10 Configuration du Service (Visualisation)

```bash
# Voir la configuration du service systemd
sudo systemctl cat ollama
```

**📋 Configuration par défaut :**
```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[Install]
WantedBy=default.target
```

**📝 Note :** On va modifier cette configuration à l'étape 3 pour permettre l'accès distant.

---

## ✅ Checklist de Validation

**Script de validation automatique :**

```bash
# Créer le script (si pas déjà fait)
cat > ~/hp-z800-ai-agent/scripts/validate-ollama.sh << 'EOF'
#!/bin/bash
echo "=== Validation Installation Ollama ==="
echo ""

echo -n "✓ Ollama installé: "
if command -v ollama &> /dev/null; then
    echo "OK ($(ollama --version))"
else
    echo "❌ ERREUR"
    exit 1
fi

echo -n "✓ Service running: "
if systemctl is-active --quiet ollama; then
    echo "OK"
else
    echo "❌ ERREUR"
    exit 1
fi

echo -n "✓ GPU détectée: "
if sudo journalctl -u ollama -n 100 --no-pager | grep -q "Quadro P4000"; then
    echo "OK (Quadro P4000 - 8GB)"
else
    echo "⚠️  WARNING"
fi

echo -n "✓ Port 11434 actif: "
if sudo ss -tlnp | grep -q ":11434"; then
    echo "OK"
else
    echo "❌ ERREUR"
    exit 1
fi

echo -n "✓ API fonctionnelle: "
if curl -s http://localhost:11434/api/tags > /dev/null 2>&1; then
    echo "OK"
else
    echo "❌ ERREUR"
    exit 1
fi

echo -n "✓ Mistral installé: "
if ollama list | grep -q "mistral"; then
    echo "OK"
else
    echo "⚠️  Pas encore installé"
fi

echo ""
echo "=== Installation Ollama Validée ✅ ==="
EOF

chmod +x ~/hp-z800-ai-agent/scripts/validate-ollama.sh

# Exécuter
~/hp-z800-ai-agent/scripts/validate-ollama.sh
```

**Checklist manuelle :**

- [ ] `ollama --version` fonctionne
- [ ] Service Ollama actif
- [ ] Service enabled (démarrage auto)
- [ ] Port 11434 en écoute (localhost)
- [ ] API répond
- [ ] GPU Quadro P4000 détectée
- [ ] Mistral téléchargé
- [ ] Test de chat fonctionne
- [ ] Pas d'erreurs dans les logs

---

## 📊 Résumé de l'Installation

À la fin de cette étape :

```
✅ Ollama 0.15.2 installé (natif, pas Docker)
✅ Service actif et enabled
✅ Port 11434 en écoute (localhost)
✅ GPU Quadro P4000 détectée (7.9 GB disponible)
✅ Mistral 7B téléchargé (4.1 GB)
✅ API REST fonctionnelle
✅ Chat en ligne de commande opérationnel

💾 Espace utilisé: ~4.2 GB
🧠 RAM service idle: ~140 MB
🎮 GPU prête pour inférence
```

---

## ➡️ Prochaine Étape

Une fois toutes les vérifications validées, passez à :

**[Étape 3 : Configuration Réseau](03-configuration-reseau.md)**

Vous allez configurer Ollama pour qu'il soit accessible depuis votre PC.

---

## 🆘 Dépannage

### Problème : Service ne démarre pas

```bash
# Vérifier les logs détaillés
sudo journalctl -u ollama -n 100 --no-pager

# Vérifier les permissions
sudo ls -la /usr/share/ollama/

# Redémarrer
sudo systemctl restart ollama
```

### Problème : Port 11434 déjà utilisé

```bash
# Identifier le processus
sudo lsof -i :11434
# OU
sudo ss -tlnp | grep 11434

# Arrêter le service qui occupe le port
sudo systemctl stop nom-du-service
```

### Problème : API ne répond pas

```bash
# Vérifier que le service tourne
sudo systemctl status ollama

# Vérifier les logs
sudo journalctl -u ollama -f

# Test manuel du binaire
/usr/local/bin/ollama serve
# Ctrl+C pour arrêter
```

### Problème : GPU non détectée

```bash
# Vérifier NVIDIA
nvidia-smi

# Vérifier les logs Ollama
sudo journalctl -u ollama | grep -i cuda

# Redémarrer Ollama
sudo systemctl restart ollama
```

### Problème : Téléchargement Mistral échoue

```bash
# Vérifier l'espace disque
df -h /usr/share/ollama/

# Vérifier la connexion Internet
curl -I https://ollama.com

# Réessayer
ollama pull mistral

# Utiliser un modèle plus petit si nécessaire
ollama pull llama3.2  # 2 GB au lieu de 4 GB
```

---

## 📝 Commandes Utiles

```bash
# Gestion du service
sudo systemctl start ollama
sudo systemctl stop ollama
sudo systemctl restart ollama
sudo systemctl status ollama

# Logs
sudo journalctl -u ollama -f              # Temps réel
sudo journalctl -u ollama -n 100          # 100 dernières lignes
sudo journalctl -u ollama --since today   # Logs du jour

# Modèles
ollama list                    # Lister
ollama pull nom-modele         # Télécharger
ollama rm nom-modele           # Supprimer
ollama run nom-modele          # Exécuter

# Configuration
sudo systemctl cat ollama      # Voir config
sudo systemctl edit ollama     # Modifier config
```

---

**📝 Installation native réussie ! Ollama tourne avec 0 overhead et accès GPU direct.**
