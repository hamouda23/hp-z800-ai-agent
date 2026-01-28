# Étape 3 : Configuration Réseau pour Accès Distant

## 🎯 Objectif

Configurer Ollama pour qu'il soit accessible depuis votre PC, avec 2 méthodes selon votre niveau de sécurité souhaité.

## ⏱️ Durée Estimée

10-15 minutes

## 📋 Prérequis

- ✅ Ollama installé et fonctionnel
- ✅ Mistral téléchargé
- ✅ Accès sudo
- ✅ Connaître l'IP de votre serveur Z800

---

## 🔍 Trouver l'IP de Votre Serveur

```bash
# Afficher toutes les IP
ip addr show | grep "inet " | grep -v 127.0.0.1

# OU simplement
hostname -I
```

**📝 Notez votre IP :** `192.168.x.x` ou `10.x.x.x`

**Exemple :** `192.168.1.100`

---

## 🌐 MÉTHODE 1 : Accès Direct (Réseau Local)

### ✅ Avantages
- Simple à configurer
- Accès direct sans latence
- Parfait pour réseau domestique/privé

### ⚠️ Inconvénients
- Ollama accessible par tous sur le réseau local
- Ne pas utiliser si serveur exposé sur Internet

---

### 1.1 Configurer Ollama pour Écouter sur Toutes les Interfaces

```bash
# Éditer la configuration du service
sudo systemctl edit ollama
```

**Un éditeur va s'ouvrir. Ajoutez ces lignes :**

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

**Sauvegardez et quittez :**
- Nano : `Ctrl+X`, puis `Y`, puis `Enter`
- Vim : `:wq`

---

### 1.2 Recharger et Redémarrer Ollama

```bash
# Recharger la configuration systemd
sudo systemctl daemon-reload

# Redémarrer Ollama
sudo systemctl restart ollama

# Vérifier le statut
sudo systemctl status ollama
```

**✅ Le service doit être "active (running)"**

---

### 1.3 Vérifier que le Port Écoute sur Toutes les Interfaces

```bash
# Vérifier avec ss
sudo ss -tlnp | grep 11434
```

**✅ Résultat attendu :**
```
tcp   LISTEN 0   4096   0.0.0.0:11434   0.0.0.0:*   users:(("ollama",pid=xxxxx,fd=3))
```

**🔍 Point important :** Vous devez voir `0.0.0.0:11434` (et non plus `127.0.0.1:11434`)

---

### 1.4 Configurer le Firewall (UFW)

```bash
# Vérifier le statut du firewall
sudo ufw status

# Si inactif, l'activer
sudo ufw enable

# Autoriser SSH (IMPORTANT avant d'activer UFW !)
sudo ufw allow 22/tcp

# Option A : Autoriser UNIQUEMENT votre PC
sudo ufw allow from VOTRE-IP-PC to any port 11434
# Exemple : sudo ufw allow from 192.168.1.50 to any port 11434

# Option B : Autoriser tout le sous-réseau local
sudo ufw allow from 192.168.1.0/24 to any port 11434

# Vérifier les règles
sudo ufw status numbered
```

**✅ Vous devriez voir :**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
11434                      ALLOW       192.168.1.50
```

---

### 1.5 Tester depuis Votre PC

**Sur votre PC (Windows/Mac/Linux) :**

```bash
# Remplacer 192.168.1.100 par l'IP de votre Z800
curl http://192.168.1.100:11434/api/tags
```

**✅ Résultat attendu :**
```json
{
  "models": [
    {
      "name": "mistral:latest",
      "model": "mistral:latest",
      "size": 4109860608,
      ...
    }
  ]
}
```

**🎉 Si vous voyez ça, l'accès direct fonctionne !**

---

### 1.6 Test avec une Question

```bash
# Depuis votre PC
curl http://192.168.1.100:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "Bonjour! Peux-tu te présenter?",
  "stream": false
}'
```

---

## 🔐 MÉTHODE 2 : Tunnel SSH (Plus Sécurisé)

### ✅ Avantages
- Très sécurisé (chiffré)
- Pas besoin d'ouvrir de ports
- Parfait si serveur exposé sur Internet
- Authentification SSH (clé ou mot de passe)

### ⚠️ Inconvénients
- Nécessite un tunnel actif
- Légère latence additionnelle

---

### 2.1 Garder Ollama sur Localhost (Configuration par Défaut)

**Si vous avez suivi la Méthode 1, annulez-la :**

```bash
# Supprimer l'override
sudo systemctl revert ollama

# Redémarrer
sudo systemctl daemon-reload
sudo systemctl restart ollama

# Vérifier qu'on est revenu sur 127.0.0.1
sudo ss -tlnp | grep 11434
# Doit afficher : 127.0.0.1:11434
```

---

### 2.2 Créer le Tunnel SSH depuis Votre PC

**Sur votre PC Windows :**

```powershell
# PowerShell ou CMD
ssh -L 11434:localhost:11434 samir@192.168.1.100

# Laissez cette fenêtre ouverte
```

**Sur votre PC Mac/Linux :**

```bash
# Terminal
ssh -L 11434:localhost:11434 samir@192.168.1.100

# Laissez ce terminal ouvert
```

**📝 Explication :**
- `-L 11434:localhost:11434` : Redirige le port local 11434 vers le port 11434 du serveur
- `samir@192.168.1.100` : Votre utilisateur et IP du Z800

---

### 2.3 Tester depuis Votre PC

**Dans un AUTRE terminal/fenêtre sur votre PC :**

```bash
# Maintenant vous accédez via localhost sur VOTRE PC
curl http://localhost:11434/api/tags
```

**✅ Résultat attendu :**
```json
{
  "models": [
    {
      "name": "mistral:latest",
      ...
    }
  ]
}
```

**🎉 Le tunnel fonctionne !**

---

### 2.4 Tunnel SSH Persistant (Optionnel)

Pour ne pas avoir à relancer le tunnel à chaque fois :

**Sur votre PC, créez un fichier `~/.ssh/config` :**

```bash
# Linux/Mac
nano ~/.ssh/config

# Windows (dans C:\Users\VotreNom\.ssh\config)
notepad C:\Users\VotreNom\.ssh\config
```

**Ajoutez :**

```
Host z800-ollama
    HostName 192.168.1.100
    User samir
    LocalForward 11434 localhost:11434
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

**Maintenant, connectez-vous simplement avec :**

```bash
ssh z800-ollama

# Le tunnel est automatiquement créé !
```

---

## 📊 Comparaison des Deux Méthodes

| Critère | Méthode 1 (Direct) | Méthode 2 (SSH Tunnel) |
|---------|-------------------|------------------------|
| **Sécurité** | ⚠️ Moyen | ✅ Élevé |
| **Simplicité** | ✅ Simple | ⚠️ Tunnel à maintenir |
| **Performance** | ✅ Rapide | ✅ Légère latence |
| **Firewall** | Doit ouvrir port 11434 | Seulement SSH (22) |
| **Usage recommandé** | Réseau local privé | Serveur exposé Internet |

---

## 🎯 Quelle Méthode Choisir ?

### **Utilisez Méthode 1 si :**
- ✅ Serveur sur réseau local domestique/bureau
- ✅ Pas d'accès depuis Internet
- ✅ Vous voulez la simplicité

### **Utilisez Méthode 2 si :**
- ✅ Serveur accessible depuis Internet
- ✅ Sécurité maximale requise
- ✅ Vous utilisez déjà SSH régulièrement

### **Utilisez les DEUX si :**
- ✅ Méthode 1 pour le réseau local (rapide)
- ✅ Méthode 2 pour l'accès distant (sécurisé)

---

## ✅ Checklist de Validation

### Pour Méthode 1 (Accès Direct)

```bash
# Sur le serveur Z800
sudo ss -tlnp | grep 11434
# Doit afficher: 0.0.0.0:11434

sudo ufw status
# Doit afficher: 11434 ALLOW from VOTRE-IP

# Depuis votre PC
curl http://IP-Z800:11434/api/tags
# Doit retourner la liste des modèles
```

### Pour Méthode 2 (SSH Tunnel)

```bash
# Sur le serveur Z800
sudo ss -tlnp | grep 11434
# Doit afficher: 127.0.0.1:11434

# Sur votre PC (tunnel actif)
curl http://localhost:11434/api/tags
# Doit retourner la liste des modèles
```

---

## 📝 Script de Test Automatique

```bash
# Créer le script de test
cat > ~/hp-z800-ai-agent/scripts/test-network-access.sh << 'EOF'
#!/bin/bash

echo "=== Test Accès Réseau Ollama ==="
echo ""

# Détecter la configuration
if sudo ss -tlnp | grep -q "0.0.0.0:11434"; then
    echo "Configuration détectée: ACCÈS DIRECT (0.0.0.0)"
    CONFIG="direct"
elif sudo ss -tlnp | grep -q "127.0.0.1:11434"; then
    echo "Configuration détectée: LOCALHOST (tunnel SSH requis)"
    CONFIG="tunnel"
else
    echo "❌ Ollama ne semble pas écouter sur le port 11434"
    exit 1
fi

echo ""

# Test local
echo -n "✓ Test local (127.0.0.1): "
if curl -s http://localhost:11434/api/tags > /dev/null 2>&1; then
    echo "OK"
else
    echo "❌ ERREUR"
fi

# Test IP locale (seulement si accès direct)
if [ "$CONFIG" = "direct" ]; then
    LOCAL_IP=$(hostname -I | awk '{print $1}')
    echo -n "✓ Test IP locale ($LOCAL_IP): "
    if curl -s http://$LOCAL_IP:11434/api/tags > /dev/null 2>&1; then
        echo "OK"
    else
        echo "❌ ERREUR"
    fi
    
    # Firewall
    echo -n "✓ Firewall configuré: "
    if sudo ufw status | grep -q "11434"; then
        echo "OK"
    else
        echo "⚠️  WARNING (port non autorisé dans UFW)"
    fi
fi

echo ""
echo "=== Configuration Validée ✅ ==="

if [ "$CONFIG" = "direct" ]; then
    echo ""
    echo "URL pour votre PC: http://$LOCAL_IP:11434"
fi
EOF

chmod +x ~/hp-z800-ai-agent/scripts/test-network-access.sh

# Exécuter
~/hp-z800-ai-agent/scripts/test-network-access.sh
```

---

## ➡️ Prochaine Étape

Une fois l'accès réseau configuré et testé, passez à :

**[Étape 4 : Installation Client PC](04-client-pc.md)**

Installer Msty ou Jan sur votre PC pour avoir une belle interface graphique !

---

## 🆘 Dépannage

### Problème : Impossible de se connecter depuis le PC

```bash
# Sur le serveur
# 1. Vérifier qu'Ollama écoute bien
sudo ss -tlnp | grep 11434

# 2. Vérifier le firewall
sudo ufw status

# 3. Tester localement d'abord
curl http://localhost:11434/api/tags

# 4. Ping depuis le PC
ping 192.168.1.100
```

### Problème : "Connection refused"

```bash
# Vérifier que le service tourne
sudo systemctl status ollama

# Vérifier les logs
sudo journalctl -u ollama -n 50
```

### Problème : Firewall bloque

```bash
# Désactiver temporairement pour tester
sudo ufw disable

# Tester depuis le PC
curl http://IP-Z800:11434/api/tags

# Réactiver et ajouter la bonne règle
sudo ufw enable
sudo ufw allow from VOTRE-IP to any port 11434
```

---

**📝 Sauvegardez votre choix de méthode et testez depuis votre PC !**
