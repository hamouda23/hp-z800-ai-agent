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

---

## 2.2 Vérification de l'Installation

```bash
# Vérifier la version installée
ollama --version

# Exemple de sortie attendue :
# ollama version is 0.x.xx
```

**📸 Capture :**
```bash
ollama --version > ~/hp-z800-ai-agent/logs/ollama-version.txt
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
     Loaded: loaded (/etc/systemd/system/ollama.service; enabled; vendor preset: enabled)
     Active: active (running) since [DATE]
       Docs: https://ollama.com/docs
   Main PID: xxxxx (ollama)
      Tasks: xx
     Memory: xxx.xM
        CPU: xxxms
     CGroup: /system.slice/ollama.service
             └─xxxxx /usr/local/bin/ollama serve
```

**🔍 Points à vérifier :**
- ✅ **Active:** `active (running)` en vert
- ✅ **Loaded:** `enabled` (démarrage auto)
- ✅ Pas de messages d'erreur en rouge

**📸 Capture :**
```bash
sudo systemctl status ollama --no-pager > ~/hp-z800-ai-agent/logs/ollama-status.txt
```

---

## 2.4 Vérification du Port

```bash
# Vérifier qu'Ollama écoute sur le port 11434
sudo netstat -tlnp | grep ollama
```

**✅ Résultat attendu :**
```
tcp        0      0 127.0.0.1:11434         0.0.0.0:*               LISTEN      xxxxx/ollama
```

**📝 Note :** Pour l'instant, Ollama écoute uniquement sur `127.0.0.1` (localhost). On va le configurer pour l'accès distant à l'étape 3.

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

**Test plus complet :**
```bash
# Vérifier que l'API répond
curl http://localhost:11434/api/version
```

**✅ Résultat attendu :**
```json
{
  "version": "0.x.xx"
}
```

---

## 2.6 Vérification de l'Accès GPU

```bash
# Vérifier les variables d'environnement CUDA
sudo systemctl show ollama | grep -i cuda

# Vérifier les logs pour voir si GPU est détectée
sudo journalctl -u ollama -n 50 --no-pager | grep -i gpu
sudo journalctl -u ollama -n 50 --no-pager | grep -i cuda
```

**✅ Résultat attendu :**
- Vous devriez voir des mentions de CUDA ou GPU dans les logs
- Pas de messages d'erreur concernant CUDA

**📸 Capture :**
```bash
sudo journalctl -u ollama -n 100 --no-pager > ~/hp-z800-ai-agent/logs/ollama-logs-initial.txt
```

---

## 2.7 Emplacement des Fichiers

```bash
# Vérifier où Ollama stocke les modèles
ls -la /usr/share/ollama/.ollama/

# Structure attendue :
# models/     <- Les modèles téléchargés seront ici
```

**💾 Taille des modèles à prévoir :**
- Mistral 7B : ~4.1 GB
- nomic-embed-text : ~274 MB
- Autres modèles : variable

**⚠️ Important :** Vérifiez que vous avez au moins **20 GB libres** sur la partition qui contient `/usr/share/ollama/`

```bash
# Vérifier l'espace disponible
df -h /usr/share/ollama/
```

---

## 2.8 Configuration du Service (Visualisation)

```bash
# Voir la configuration du service systemd
sudo systemctl cat ollama
```

**📋 Fichier de configuration par défaut :**
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

## 2.9 Vérification des Logs en Temps Réel

```bash
# Suivre les logs en direct
sudo journalctl -u ollama -f
```

**Appuyez sur Ctrl+C pour arrêter**

**📝 À observer :**
- Aucun message d'erreur
- Service qui tourne normalement
- Pas de crash ou redémarrage

---

## 2.10 Test de Redémarrage

```bash
# Redémarrer le service
sudo systemctl restart ollama

# Attendre 2 secondes
sleep 2

# Vérifier qu'il a redémarré correctement
sudo systemctl status ollama

# Tester l'API à nouveau
curl http://localhost:11434/api/tags
```

**✅ Si tout fonctionne :** Le service redémarre sans erreur et l'API répond.

---

## ✅ Checklist de Validation

Avant de passer à l'étape suivante, vérifiez :

```bash
# Script de validation automatique
cat > ~/hp-z800-ai-agent/logs/validate-ollama.sh << 'EOF'
#!/bin/bash
echo "=== Validation Installation Ollama ==="
echo ""

# 1. Version Ollama
echo -n "✓ Ollama installé: "
if command -v ollama &> /dev/null; then
    echo "OK ($(ollama --version))"
else
    echo "❌ ERREUR"
    exit 1
fi

# 2. Service actif
echo -n "✓ Service running: "
if systemctl is-active --quiet ollama; then
    echo "OK"
else
    echo "❌ ERREUR"
    exit 1
fi

# 3. Service enabled
echo -n "✓ Service enabled: "
if systemctl is-enabled --quiet ollama; then
    echo "OK"
else
    echo "⚠️  WARNING (pas de démarrage auto)"
fi

# 4. Port 11434 écoute
echo -n "✓ Port 11434 actif: "
if sudo netstat -tlnp | grep -q ":11434"; then
    echo "OK"
else
    echo "❌ ERREUR"
    exit 1
fi

# 5. API répond
echo -n "✓ API fonctionnelle: "
if curl -s http://localhost:11434/api/tags > /dev/null 2>&1; then
    echo "OK"
else
    echo "❌ ERREUR"
    exit 1
fi

# 6. GPU détectée par système
echo -n "✓ GPU accessible: "
if nvidia-smi > /dev/null 2>&1; then
    echo "OK"
else
    echo "⚠️  WARNING"
fi

echo ""
echo "=== Installation Ollama Validée ✅ ==="
echo ""
echo "Prochaine étape: Configuration réseau pour accès distant"
EOF

chmod +x ~/hp-z800-ai-agent/logs/validate-ollama.sh
~/hp-z800-ai-agent/logs/validate-ollama.sh
```

**Checklist manuelle :**

- [ ] `ollama --version` fonctionne
- [ ] Service Ollama actif (`systemctl status ollama`)
- [ ] Service enabled (démarrage auto)
- [ ] Port 11434 en écoute
- [ ] API répond (`curl http://localhost:11434/api/tags`)
- [ ] nvidia-smi détecte la GPU
- [ ] Pas d'erreurs dans les logs
- [ ] Au moins 20 GB libres pour les modèles

---

## 📊 Résumé de l'Installation

À la fin de cette étape, vous avez :

```
Ollama Installation:
├── Binaire: /usr/local/bin/ollama ✅
├── Service: ollama.service (active & enabled) ✅
├── Port: 11434 (localhost uniquement) ✅
├── API: Fonctionnelle ✅
├── Modèles: Dossier prêt (vide) ✅
├── GPU: Détectée par le système ✅
└── Logs: Aucune erreur ✅

Utilisation RAM actuelle: ~50-100 MB (service idle)
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

# Réinstaller
sudo systemctl stop ollama
sudo rm -f /usr/local/bin/ollama
curl -fsSL https://ollama.com/install.sh | sh
```

### Problème : Port 11434 déjà utilisé

```bash
# Identifier le processus
sudo lsof -i :11434

# Arrêter le service qui occupe le port
sudo systemctl stop nom-du-service

# Redémarrer Ollama
sudo systemctl restart ollama
```

### Problème : API ne répond pas

```bash
# Vérifier que le service tourne
sudo systemctl status ollama

# Vérifier les logs
sudo journalctl -u ollama -f

# Test manuel
/usr/local/bin/ollama serve
# Ctrl+C pour arrêter, puis redémarrer le service
```

### Problème : GPU non détectée

```bash
# Vérifier NVIDIA
nvidia-smi

# Réinstaller les pilotes si nécessaire
sudo ubuntu-drivers autoinstall
sudo reboot

# Après reboot, redémarrer Ollama
sudo systemctl restart ollama
```

---

## 📝 Notes Importantes

### Différences Native vs Docker

**✅ Avantages installation native :**
- Accès GPU direct (pas de NVIDIA Container Toolkit)
- 0 overhead RAM (~800 MB économisés)
- Démarrage plus rapide
- Logs système natifs
- Mise à jour simple (`curl ... | sh`)

**⚠️ À retenir :**
- Les modèles sont dans `/usr/share/ollama/.ollama/models/`
- Le service tourne sous l'utilisateur `ollama`
- Configuration dans `/etc/systemd/system/ollama.service`

### Commandes Utiles

```bash
# Démarrer/Arrêter/Redémarrer
sudo systemctl start ollama
sudo systemctl stop ollama
sudo systemctl restart ollama

# Voir les logs
sudo journalctl -u ollama -f          # Temps réel
sudo journalctl -u ollama -n 100      # 100 dernières lignes
sudo journalctl -u ollama --since today  # Logs du jour

# Vérifier la configuration
sudo systemctl cat ollama
sudo systemctl show ollama

# Désactiver le démarrage auto (si besoin)
sudo systemctl disable ollama
```

---

**📝 Sauvegardez bien tous les logs dans `~/hp-z800-ai-agent/logs/` pour référence future !**
