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

## 🔍 Étape Préliminaire : Trouver l'IP de Votre Serveur

```bash
# Afficher l'IP principale
hostname -I | awk '{print $1}'

# OU voir toutes les interfaces
ip addr show | grep "inet " | grep -v 127.0.0.1
```

**📝 Notez votre IP :** Exemple `192.168.1.108`

Dans ce guide, nous utiliserons `192.168.1.108` comme exemple.

---

## 🌐 MÉTHODE 1 : Accès Direct (Réseau Local)

### ✅ Avantages
- Simple à configurer
- Accès direct sans latence
- Parfait pour réseau domestique/privé
- Performance maximale

### ⚠️ Inconvénients
- Ollama accessible par tous sur le réseau local
- Ne pas utiliser si serveur exposé sur Internet public

### ⚙️ Quand Utiliser Cette Méthode ?
- ✅ Serveur sur réseau local domestique/bureau
- ✅ Pas d'accès depuis Internet
- ✅ Tous les appareils sont de confiance

---

## 📋 Configuration Méthode 1

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
- **Nano :** `Ctrl+X`, puis `Y`, puis `Enter`
- **Vim :** Appuyez sur `Esc`, tapez `:wq`, puis `Enter`

**💡 Alternative si l'édition ne fonctionne pas :**
```bash
# Créer le fichier d'override manuellement
sudo mkdir -p /etc/systemd/system/ollama.service.d/

sudo nano /etc/systemd/system/ollama.service.d/override.conf
```

Ajoutez :
```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

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

**✅ Résultat attendu (l'un de ces formats) :**
```
LISTEN 0  4096  0.0.0.0:11434  0.0.0.0:*  users:(("ollama",pid=xxxxx,fd=3))
```
**OU**
```
LISTEN 0  4096  *:11434  *:*  users:(("ollama",pid=xxxxx,fd=3))
```

**🔍 Point important :** 
- ✅ Vous devez voir `0.0.0.0:11434` ou `*:11434`
- ❌ Si vous voyez `127.0.0.1:11434`, la configuration n'a pas pris effet

---

### 1.4 Configurer le Firewall (UFW)

```bash
# Vérifier le statut du firewall
sudo ufw status

# Si inactif, l'activer (IMPORTANT : autoriser SSH d'abord !)
sudo ufw allow 22/tcp
sudo ufw enable
```

**Option A : Autoriser UN SEUL PC (Recommandé)**
```bash
# Remplacez par l'IP de VOTRE PC
sudo ufw allow from 192.168.1.147 to any port 11434

# Exemple pour plusieurs PCs
sudo ufw allow from 192.168.1.147 to any port 11434
sudo ufw allow from 192.168.1.161 to any port 11434
```

**Option B : Autoriser Tout le Sous-Réseau Local**
```bash
# Autoriser tous les appareils 192.168.1.x
sudo ufw allow from 192.168.1.0/24 to any port 11434
```

**Vérifier les règles :**
```bash
sudo ufw status numbered
```

**✅ Vous devriez voir :**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
11434                      ALLOW       192.168.1.147
11434                      ALLOW       192.168.1.161
```

---

### 1.5 Tester depuis Votre PC

**Sur votre PC (Windows/Mac/Linux) :**

**Windows PowerShell :**
```powershell
# Remplacer 192.168.1.108 par l'IP de votre Z800
curl http://192.168.1.108:11434/api/tags
```

**Linux/Mac :**
```bash
curl http://192.168.1.108:11434/api/tags
```

**✅ Résultat attendu :**
```json
{
  "models": [
    {
      "name": "mistral:latest",
      "model": "mistral:latest",
      "size": 4372824384,
      ...
    }
  ]
}
```

**🎉 Si vous voyez la liste des modèles, l'accès direct fonctionne !**

---

### 1.6 Test avec une Question

```bash
# Depuis votre PC
curl http://192.168.1.108:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "Bonjour ! Peux-tu te présenter ?",
  "stream": false
}'
```

---

## 🔐 MÉTHODE 2 : Tunnel SSH (Plus Sécurisé)

### ✅ Avantages
- Très sécurisé (chiffrement SSH)
- Pas besoin d'ouvrir de ports dans le firewall
- Parfait si serveur exposé sur Internet
- Authentification SSH (clé ou mot de passe)
- Pas de risque d'accès non autorisé

### ⚠️ Inconvénients
- Nécessite un tunnel SSH actif
- Légère latence additionnelle (négligeable sur réseau local)
- Un peu plus complexe à configurer

### ⚙️ Quand Utiliser Cette Méthode ?
- ✅ Serveur accessible depuis Internet
- ✅ Sécurité maximale requise
- ✅ Vous utilisez déjà SSH régulièrement
- ✅ Pas de confiance totale dans le réseau local

---

## 📋 Configuration Méthode 2

### 2.1 Garder Ollama sur Localhost (Configuration par Défaut)

**Si vous avez suivi la Méthode 1, annulez-la :**

```bash
# Supprimer l'override
sudo systemctl revert ollama

# OU supprimer le fichier manuellement
sudo rm -f /etc/systemd/system/ollama.service.d/override.conf

# Redémarrer
sudo systemctl daemon-reload
sudo systemctl restart ollama

# Vérifier qu'on est revenu sur 127.0.0.1
sudo ss -tlnp | grep 11434
```

**✅ Résultat attendu :**
```
LISTEN 0  4096  127.0.0.1:11434  0.0.0.0:*
```

---

### 2.2 Créer le Tunnel SSH depuis Votre PC

**Sur votre PC Windows (PowerShell ou CMD) :**

```powershell
# Syntaxe : ssh -L port_local:destination:port_distant user@serveur
ssh -L 11434:localhost:11434 samir@192.168.1.108

# Entrez votre mot de passe
# Laissez cette fenêtre ouverte (le tunnel est actif tant que la connexion SSH est ouverte)
```

**Sur votre PC Mac/Linux :**

```bash
ssh -L 11434:localhost:11434 samir@192.168.1.108

# Laissez ce terminal ouvert
```

**📝 Explication :**
- `-L 11434:localhost:11434` : Redirige le port 11434 de VOTRE PC vers le port 11434 du serveur
- `samir@192.168.1.108` : Votre utilisateur SSH et l'IP du serveur

---

### 2.3 Tester depuis Votre PC

**Dans un AUTRE terminal/fenêtre PowerShell sur votre PC :**

```bash
# Maintenant vous accédez via localhost sur VOTRE PC
# Le tunnel redirige automatiquement vers le serveur
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

Pour ne pas avoir à retaper la commande à chaque fois :

**Sur votre PC, créez/éditez le fichier SSH config :**

**Linux/Mac :**
```bash
nano ~/.ssh/config
```

**Windows :**
```powershell
# Dans PowerShell
notepad $HOME\.ssh\config
```

**Ajoutez cette configuration :**

```
Host z800-ollama
    HostName 192.168.1.108
    User samir
    LocalForward 11434 localhost:11434
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

**Maintenant, connectez-vous simplement avec :**

```bash
ssh z800-ollama

# Le tunnel est automatiquement créé !
# Ollama accessible sur http://localhost:11434
```

---

### 2.5 Tunnel SSH en Arrière-Plan (Avancé)

**Pour garder le tunnel actif sans fenêtre ouverte :**

```bash
# Linux/Mac
ssh -f -N -L 11434:localhost:11434 samir@192.168.1.108

# -f : Passe en arrière-plan
# -N : Ne pas exécuter de commande distante
```

**Pour arrêter le tunnel :**
```bash
# Trouver le processus
ps aux | grep "ssh.*11434"

# Tuer le processus
kill <PID>
```

---

## 📊 Comparaison des Deux Méthodes

| Critère | Méthode 1 (Direct) | Méthode 2 (SSH Tunnel) |
|---------|-------------------|------------------------|
| **Sécurité** | ⚠️ Moyen | ✅ Élevé (chiffrement) |
| **Simplicité** | ✅ Simple | ⚠️ Tunnel à maintenir |
| **Performance** | ✅ Rapide | ✅ Légère latence |
| **Configuration Firewall** | Port 11434 ouvert | Seulement SSH (22) |
| **Accès multi-appareils** | ✅ Facile | ⚠️ Tunnel par appareil |
| **Usage recommandé** | Réseau local privé | Internet / sécurité max |

---

## 🎯 Quelle Méthode Choisir ?

### **Utilisez Méthode 1 si :**
- ✅ Serveur sur réseau local domestique/bureau
- ✅ Pas d'accès depuis Internet
- ✅ Vous voulez la simplicité
- ✅ Performance maximale

### **Utilisez Méthode 2 si :**
- ✅ Serveur accessible depuis Internet
- ✅ Sécurité maximale requise
- ✅ Vous utilisez déjà SSH régulièrement
- ✅ Réseau non sécurisé (WiFi public, etc.)

### **Utilisez les DEUX si :**
- ✅ Méthode 1 pour accès rapide sur réseau local
- ✅ Méthode 2 pour accès distant sécurisé

---

## ✅ Scripts de Validation

### Script de Test Réseau

```bash
# Créer le script de test
cat > ~/hp-z800-ai-agent/scripts/test-network-access.sh << 'EOF'
#!/bin/bash

echo "=== Test Accès Réseau Ollama ==="
echo ""

# Détecter la configuration
if sudo ss -tlnp | grep -E "(0.0.0.0:11434|\*:11434)" > /dev/null; then
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
    echo "URL pour accès distant: http://$LOCAL_IP:11434"
    echo ""
    echo "PCs autorisés dans le firewall:"
    sudo ufw status | grep 11434
fi
EOF

chmod +x ~/hp-z800-ai-agent/scripts/test-network-access.sh

# Exécuter
~/hp-z800-ai-agent/scripts/test-network-access.sh
```

---

## 🆘 Dépannage

### Problème : "Connection refused" depuis le PC

```bash
# Sur le serveur
# 1. Vérifier qu'Ollama écoute bien
sudo ss -tlnp | grep 11434

# 2. Vérifier le firewall
sudo ufw status

# 3. Tester localement d'abord
curl http://localhost:11434/api/tags

# 4. Ping depuis le PC vers le serveur
ping 192.168.1.108
```

### Problème : Firewall bloque

```bash
# Vérifier les règles
sudo ufw status numbered

# Ajouter la bonne règle
sudo ufw allow from IP-DU-PC to any port 11434

# OU désactiver temporairement pour tester
sudo ufw disable
# Tester depuis le PC
# Puis réactiver
sudo ufw enable
```

### Problème : Configuration ne prend pas effet

```bash
# Vérifier le fichier d'override
sudo systemctl cat ollama | grep OLLAMA_HOST

# Si absent, recréer manuellement
sudo mkdir -p /etc/systemd/system/ollama.service.d/
echo -e "[Service]\nEnvironment=\"OLLAMA_HOST=0.0.0.0:11434\"" | sudo tee /etc/systemd/system/ollama.service.d/override.conf

# Recharger
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

### Problème : Tunnel SSH ne fonctionne pas

```bash
# Vérifier que SSH fonctionne
ssh samir@192.168.1.108

# Vérifier que le port local n'est pas déjà utilisé
# Windows
netstat -ano | findstr :11434

# Linux/Mac
lsof -i :11434

# Tuer le processus si nécessaire
```

### Problème : Pare-feu Windows bloque (PC client)

```bash
# Windows PowerShell (Admin)
# Autoriser les connexions sortantes sur le port 11434
New-NetFirewallRule -DisplayName "Ollama Client" -Direction Outbound -LocalPort 11434 -Protocol TCP -Action Allow
```

---

## 📝 Résumé

À la fin de cette étape, vous avez :

**Configuration serveur :**
```
✅ Ollama configuré pour accès réseau (Méthode 1)
   OU
✅ Ollama sur localhost avec tunnel SSH (Méthode 2)
✅ Firewall configuré
✅ Tests de connexion réussis
```

**Depuis votre PC :**
```
✅ Accès à l'API Ollama
✅ Communication avec Mistral
✅ Prêt pour installer un client graphique
```

---

## ➡️ Prochaine Étape

Une fois l'accès réseau configuré et testé, passez à :

**[Étape 4 : Installation Client PC (Msty)](04-client-pc.md)**

Installer et configurer Msty ou Jan sur votre PC pour avoir une interface graphique moderne !

---

**📝 Choisissez la méthode qui correspond à votre usage et testez depuis votre PC !**
