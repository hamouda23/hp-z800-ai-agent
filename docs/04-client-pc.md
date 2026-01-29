# Étape 4 : Installation et Configuration du Client PC (Msty)

## 🎯 Objectif

Installer et configurer Msty sur votre PC pour avoir une interface graphique moderne pour discuter avec Mistral.

## ⏱️ Durée Estimée

5-10 minutes

## 📋 Prérequis

- ✅ Ollama installé et fonctionnel sur le serveur Z800
- ✅ Mistral téléchargé
- ✅ Accès réseau configuré (Étape 3 complétée)
- ✅ IP du serveur connue (exemple: `192.168.1.108`)

---

## 🎨 Qu'est-ce que Msty ?

**Msty** est un client graphique moderne pour interagir avec des modèles LLM locaux comme Ollama.

### ✅ Avantages
- Interface utilisateur élégante et intuitive
- Support multi-modèles
- Historique des conversations
- Personnalisation des prompts
- Gratuit et léger
- Compatible Windows, Mac, Linux

---

## 💻 Installation de Msty

### Windows

**Méthode 1 : Winget (Recommandé)**
```powershell
# Ouvrir PowerShell
winget install Msty
```

**Méthode 2 : Téléchargement Direct**
1. Visitez : https://msty.app
2. Cliquez sur **Download for Windows**
3. Exécutez le fichier `.exe`
4. Suivez l'assistant d'installation

---

### macOS

**Méthode 1 : Homebrew**
```bash
brew install --cask msty
```

**Méthode 2 : Téléchargement Direct**
1. Visitez : https://msty.app
2. Cliquez sur **Download for Mac**
3. Ouvrez le fichier `.dmg`
4. Glissez Msty dans Applications

---

### Linux

**AppImage (Toutes distributions)**
1. Visitez : https://msty.app
2. Téléchargez le fichier `.AppImage`
3. Rendez-le exécutable :
```bash
chmod +x Msty-*.AppImage
./Msty-*.AppImage
```

**Snap (Ubuntu/Debian)**
```bash
sudo snap install msty
```

---

## ⚙️ Configuration de Msty

### 1️⃣ Lancer Msty

Ouvrez l'application Msty sur votre PC.

---

### 2️⃣ Localiser les Paramètres

La configuration peut varier selon la version. Cherchez dans :

**Option A : Menu Principal**
- **File** → **Settings** ou **Preferences**
- **Tools** → **Options**
- **Msty** → **Preferences** (macOS)

**Option B : Icônes dans l'Interface**
- Cherchez : ⚙️ (roue dentée)
- Ou : ⋮ (trois points)
- Ou : ☰ (hamburger menu)

**Option C : Raccourci Clavier**
- Windows/Linux : `Ctrl + ,` (virgule)
- macOS : `Cmd + ,` (virgule)

---

### 3️⃣ Configurer la Connexion Ollama

Dans les paramètres, cherchez une section nommée :
- **"Ollama"**
- **"Server"**
- **"API Configuration"**
- **"Connection Settings"**

**Entrez l'URL de votre serveur :**

**Si vous utilisez la Méthode 1 (Accès Direct) :**
```
http://192.168.1.108:11434
```
(Remplacez `192.168.1.108` par l'IP de VOTRE serveur Z800)

**Si vous utilisez la Méthode 2 (Tunnel SSH) :**
```
http://localhost:11434
```
(Assurez-vous que le tunnel SSH est actif)

---

### 4️⃣ Tester la Connexion

Cherchez un bouton :
- **"Test Connection"**
- **"Verify"**
- **"Check Server"**

Cliquez dessus.

**✅ Succès :**
- Message : "Connection successful"
- Liste des modèles apparaît
- Vous voyez "mistral:latest"

**❌ Échec :**
- Vérifiez l'URL (pas d'espace, bon port)
- Vérifiez que le serveur Ollama tourne
- Vérifiez le firewall (Étape 3)

---

### 5️⃣ Sélectionner Mistral

1. Dans l'interface principale de Msty
2. Cherchez une liste déroulante ou menu de sélection de modèle
3. Sélectionnez **"mistral:latest"** ou **"mistral"**

---

## 💬 Utilisation de Msty

### Démarrer une Conversation

1. **Nouvelle conversation :**
   - Cliquez sur **"+ New Chat"** ou **"New Conversation"**
   
2. **Tapez votre message :**
   ```
   Bonjour ! Peux-tu te présenter en français ?
   ```

3. **Envoyez** (bouton ou `Entrée`)

**✅ Mistral devrait répondre directement depuis votre serveur Z800 !**

---

### Fonctionnalités Utiles

#### Historique des Conversations
- Toutes vos conversations sont sauvegardées localement
- Accédez-y via la barre latérale

#### Paramètres de Génération
Vous pouvez ajuster :
- **Temperature** : Créativité (0.1 = précis, 1.0 = créatif)
- **Max Tokens** : Longueur maximale de la réponse
- **Top-p** : Diversité des réponses

#### System Prompt (Optionnel)
Définissez un comportement par défaut :
```
Tu es un assistant expert en programmation Python.
```

#### Copier/Exporter
- Copiez les réponses
- Exportez les conversations en Markdown

---

## 🔧 Configuration Avancée (Fichier Config)

Si vous ne trouvez pas les paramètres dans l'interface, éditez le fichier de configuration :

### Windows
```
C:\Users\VotreNom\AppData\Roaming\Msty\config.json
```

### macOS
```
~/Library/Application Support/Msty/config.json
```

### Linux
```
~/.config/Msty/config.json
```

**Ouvrez avec un éditeur de texte et cherchez :**
```json
{
  "ollamaUrl": "http://localhost:11434",
  ...
}
```

**Modifiez en :**
```json
{
  "ollamaUrl": "http://192.168.1.108:11434",
  ...
}
```

**Sauvegardez et relancez Msty.**

---

## 🧪 Tests et Validation

### Test 1 : Conversation Simple

**Vous :**
```
Explique-moi ce qu'est un LLM en une phrase.
```

**Mistral devrait répondre :**
```
Un LLM (Large Language Model) est un modèle d'intelligence artificielle 
entraîné sur d'énormes quantités de texte pour comprendre et générer 
du langage naturel de manière cohérente.
```

---

### Test 2 : Code

**Vous :**
```
Écris une fonction Python qui calcule la suite de Fibonacci.
```

**Mistral devrait générer du code :**
```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

---

### Test 3 : Plusieurs Langues

**Vous :**
```
Hello! Can you introduce yourself in English?
```

**Mistral devrait répondre en anglais.**

---

## 📊 Monitoring de l'Utilisation

### Depuis le Serveur Z800

**Vérifier que les requêtes arrivent :**
```bash
# Voir les logs en temps réel
sudo journalctl -u ollama -f

# Vous devriez voir des lignes comme :
# [GIN] 2026/01/28 - 12:34:56 | 200 | 2.5s | 192.168.1.147 | POST "/api/generate"
```

**Vérifier l'utilisation GPU :**
```bash
# Pendant qu'une requête est en cours
nvidia-smi

# Vous devriez voir :
# GPU-Util : 90-100%
# Memory-Usage : ~4-6 GB
```

---

## 🎨 Alternatives à Msty

Si Msty ne vous convient pas, essayez :

### Jan (Open Source)
**Installation :**
- https://jan.ai
- Interface similaire à ChatGPT
- Support de multiples backends

**Configuration :**
1. Télécharger et installer
2. Settings → Ollama Server
3. URL : `http://192.168.1.108:11434`

---

### LM Studio
**Installation :**
- https://lmstudio.ai
- Très complet, interface professionnelle
- Gestion de modèles intégrée

**Configuration :**
1. Télécharger et installer
2. Preferences → Remote Server
3. URL : `http://192.168.1.108:11434`

---

### Enchanted (macOS/iOS)
**Installation :**
- https://enchanted.to
- Natif Apple, très rapide
- Synchronisation iCloud

---

## ✅ Checklist de Validation

- [ ] Msty installé sur votre PC
- [ ] Configuration serveur Ollama ajoutée
- [ ] Test de connexion réussi
- [ ] Modèle "mistral" visible dans la liste
- [ ] Première conversation fonctionnelle
- [ ] Réponses fluides et rapides
- [ ] Historique sauvegardé

---

## 🆘 Dépannage

### Problème : "Cannot connect to server"

**Vérifications :**
```bash
# Sur le serveur Z800
sudo systemctl status ollama
sudo ss -tlnp | grep 11434

# Depuis votre PC
ping 192.168.1.108
curl http://192.168.1.108:11434/api/tags
```

**Solutions :**
- Vérifiez l'URL dans Msty (pas d'espace, bon port)
- Vérifiez le firewall (Étape 3)
- Si tunnel SSH : vérifiez qu'il est actif

---

### Problème : Modèles ne s'affichent pas

**Vérifications :**
```bash
# Sur le serveur
ollama list

# L'API doit retourner les modèles
curl http://192.168.1.108:11434/api/tags
```

**Solution :**
- Redémarrez Msty
- Vérifiez que Mistral est bien téléchargé
- Reconnectez le serveur dans Msty

---

### Problème : Réponses très lentes

**Vérifications :**
```bash
# Sur le serveur - vérifier GPU
nvidia-smi

# Vérifier la charge système
htop
```

**Causes possibles :**
- GPU non utilisée (revérifiez les pilotes NVIDIA)
- RAM insuffisante (vérifiez le swap)
- Autre processus utilise la GPU
- Réseau lent (si accès distant)

---

### Problème : Msty ne trouve pas le fichier config

**Créez-le manuellement :**

**Windows :**
```powershell
mkdir "$env:APPDATA\Msty"
echo '{"ollamaUrl":"http://192.168.1.108:11434"}' > "$env:APPDATA\Msty\config.json"
```

**macOS/Linux :**
```bash
mkdir -p ~/.config/Msty
echo '{"ollamaUrl":"http://192.168.1.108:11434"}' > ~/.config/Msty/config.json
```

---

## 📝 Résumé

À la fin de cette étape :

```
✅ PC Client configuré
✅ Msty installé et connecté au Z800
✅ Interface graphique moderne
✅ Conversations avec Mistral fonctionnelles
✅ Historique local sauvegardé

Architecture complète :
PC (Msty) ←→ Réseau ←→ Z800 (Ollama + Mistral)
```

---

## ➡️ Prochaines Étapes (Optionnelles)

### Pour aller plus loin :

1. **[Configuration RAG](05-configuration-rag.md)** - Interroger vos documents
2. **[Fine-tuning Setup](06-finetuning-setup.md)** - Personnaliser Mistral
3. **[Modèles Additionnels](07-modeles-additionnels.md)** - Télécharger d'autres LLMs

---

## 🎉 Félicitations !

Vous avez maintenant une stack LLM complète et fonctionnelle :

```
✅ Serveur Z800 avec Ollama natif
✅ Mistral 7B opérationnel
✅ Accès réseau configuré et sécurisé
✅ Interface graphique Msty
✅ GPU Quadro P4000 utilisée à 100%
✅ 0 overhead Docker
✅ Performance maximale
```

**Profitez de votre assistant IA local et privé ! 🚀**

---

**💡 Astuce :** Pour ajouter d'autres PCs, retournez à l'Étape 3 et ajoutez simplement leur IP dans le firewall UFW.
