# hp-z800-ai-agent

🤖 Stack LLM 100% Native sur HP Z800 : Ollama + Client Distant + RAG + Fine-tuning

## 🎯 Philosophie du Projet

**Installation 100% NATIVE** - Pas de Docker, contrôle total, performance maximale.

### Pourquoi Native ?

- ✅ **0 overhead RAM** - Toute la RAM disponible pour les modèles
- ✅ **Accès GPU direct** - Performance maximale pour fine-tuning
- ✅ **Contrôle total du Swap** - Crucial avec 12 GB RAM
- ✅ **Simplicité** - Pas de complexité Docker/containers
- ✅ **Économie** - 1.5 GB RAM économisés vs Docker

### Architecture

```
┌──────────────────────────────────────────┐
│  Votre PC (Interface Utilisateur)        │
├──────────────────────────────────────────┤
│  • Msty / Jan (Interface graphique)      │
│  • Scripts Python (automatisation)       │
│  • VS Code Remote (développement)        │
│    ↓                                      │
│    └──→ API HTTP: 192.168.x.x:11434      │
└──────────────────────────────────────────┘
                    ↓ API REST
┌──────────────────────────────────────────┐
│  HP Z800 (Backend 100% Natif)            │
├──────────────────────────────────────────┤
│                                           │
│  🔷 Ollama Native (Port 11434)           │
│     ├─ mistral (Chat principal)          │
│     ├─ nomic-embed-text (RAG)            │
│     └─ Accès GPU direct                  │
│                                           │
│  🔷 Conda Environments (Disque 2)        │
│     ├─ base (Python 3.x)                 │
│     ├─ env1 (Python 3.x)                 │
│     ├─ env2 (Python 3.x)                 │
│     └─ finetuning (PyTorch + CUDA)       │
│                                           │
│  Resources:                               │
│  • RAM disponible: ~11 GB (vs 9.5 Docker)│
│  • GPU: Quadro P4000 8GB (100% perfs)    │
│  • Swap: 8 GB configuré                  │
│                                           │
└──────────────────────────────────────────┘
```

## 📊 Spécifications du Serveur

| Composant | Détails |
|-----------|---------|
| **Modèle** | HP Z800 Workstation (2009-2010) |
| **CPU** | 2× Intel Xeon E5640 @ 2.67 GHz (8 cœurs, 16 threads) |
| **RAM** | 12 GB DDR3 ECC |
| **GPU** | NVIDIA Quadro P4000 (8 GB GDDR5, 1792 CUDA cores) |
| **OS** | Ubuntu Server 22.04 LTS (noyau 6.8.0-40-generic) |
| **Storage** | Disque 1: Système / Disque 2: Conda & ML |
| **Pilotes** | NVIDIA 535+ avec CUDA 12.x |

## 🚀 Installation Rapide

### Sur le Serveur HP Z800

```bash
# Cloner le projet
git clone https://github.com/VOTRE-USERNAME/hp-z800-ai-agent.git
cd hp-z800-ai-agent

# Lancer l'installation complète
chmod +x scripts/install-all-native.sh
./scripts/install-all-native.sh

# L'IP du serveur sera affichée à la fin
```

### Sur Votre PC

**Option A : Client Graphique (Recommandé)**

```bash
# Windows
winget install Msty

# macOS
brew install --cask msty

# Linux
# Télécharger depuis https://msty.app
```

Puis configurer : `http://IP-DU-Z800:11434`

**Option B : Scripts Python**

```bash
pip install requests
python examples/chat-client.py
```

## 📚 Installation Manuelle (Étape par Étape)

Pour une installation contrôlée, suivez ces guides dans l'ordre :

1. **[Préparation Système](docs/01-preparation.md)** - Vérifications et prérequis
2. **[Installation Ollama](docs/02-installation-ollama.md)** - Backend LLM natif
3. **[Configuration Réseau](docs/03-configuration-reseau.md)** - Accès distant sécurisé
4. **[Installation Client PC](docs/04-client-pc.md)** - Msty, Jan ou scripts
5. **[Configuration RAG ](docs/05-configuration-rag.md)** - Documents et embeddings
6. **[RAG Avancé avec ChromaDB](docs/06-rag-avance-chromadb.md)** - RAG Avancé
7. **Setup Fine-tuning** *(à venir)* - Environnement Conda
8. **Optimisation Swap** *(à venir)* - Gestion mémoire

## 💡 Utilisation

### Accès à l'Interface Web

```
http://IP-DU-Z800:11434
```

**Depuis Msty :**
1. Ouvrir Msty
2. Settings → Ollama Server → `http://IP-Z800:11434`
3. Sélectionner le modèle "mistral"
4. Commencer à discuter !

### Upload de Documents (RAG)

*(Documentation à venir)*

### API REST (pour scripts/intégrations)

```bash
# Génération de texte
curl http://IP-DU-Z800:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "Explique-moi le RAG",
  "stream": false
}'
```

```python
# Client Python
import requests

def ask_mistral(prompt):
    response = requests.post(
        "http://IP-DU-Z800:11434/api/generate",
        json={"model": "mistral", "prompt": prompt, "stream": False}
    )
    return response.json()['response']

print(ask_mistral("Qu'est-ce que le machine learning?"))
```

## 🎓 Fonctionnalités

### Chat de Base

- ✅ Interface graphique moderne (Msty/Jan)
- ✅ Scripts Python personnalisables
- ✅ API REST compatible OpenAI
- ✅ Streaming des réponses
- ✅ Multi-modèles

### RAG (Retrieval Augmented Generation)

*(Documentation à venir)*

Interrogez vos propres documents :
- Documentation technique
- Manuels d'utilisation
- Articles de recherche
- Notes personnelles
- Bases de connaissances

### Fine-tuning

*(Documentation à venir)*

Environnement Conda configuré pour :
- Fine-tuner Mistral sur vos données
- Créer des modèles spécialisés
- Entraîner des adapters LoRA

## 📁 Structure du Projet

```
hp-z800-ai-agent/
├── README.md                      # Ce fichier
├── docs/                          # Documentation détaillée
│   ├── 01-preparation.md
│   ├── 02-installation-ollama.md
│   ├── 03-configuration-reseau.md
│   └── 04-client-pc.md
├── scripts/                       # Scripts d'installation
│   ├── validate-ollama.sh
│   └── test-network-access.sh
├── examples/                      # Exemples de code
│   ├── chat-client.py
│   └── rag-example.py
└── logs/                          # Logs d'installation
    └── .gitkeep
```

## 🛠️ Modèles Disponibles

| Modèle | Taille | Usage | RAM Requise |
|--------|--------|-------|-------------|
| **mistral** | 4.1 GB | Chat principal | 6 GB |
| **nomic-embed-text** | 274 MB | Embeddings RAG | 1 GB |
| llama3.2 | 2 GB | Alternative légère | 4 GB |
| codellama | 3.8 GB | Code generation | 6 GB |

```bash
# Télécharger un nouveau modèle
ollama pull nom-du-modele

# Lister les modèles installés
ollama list
```

## 🔒 Sécurité

### Configuration Firewall

```bash
# Autoriser uniquement votre IP
sudo ufw allow from VOTRE-IP to any port 11434
sudo ufw allow from VOTRE-IP to any port 11434
sudo ufw enable
```

### Tunnel SSH (Alternative sécurisée)

```bash
# Depuis votre PC
ssh -L 11434:localhost:11434 user@IP-Z800

# Puis accédez à http://localhost:11434
```

## 📊 Monitoring

```bash
# Vérifier l'utilisation GPU
watch -n 1 nvidia-smi

# Logs Ollama
sudo journalctl -u ollama -f

# Statut des services
sudo systemctl status ollama
```

## 🆘 Dépannage

### Problème : Impossible de se connecter depuis le PC

```bash
# Sur le serveur
sudo ss -tlnp | grep 11434
sudo ufw status
curl http://localhost:11434/api/tags

# Depuis le PC
ping IP-DU-Z800
```

Voir la documentation complète dans chaque fichier `docs/`.

## 📖 Ressources

- [Documentation Ollama](https://github.com/ollama/ollama)
- [API Reference](https://github.com/ollama/ollama/blob/main/docs/api.md)
- [Msty - Client PC](https://msty.app)
- [Jan - Client alternatif](https://jan.ai)
- [Guide Fine-tuning LLMs](https://huggingface.co/docs/transformers/training)

## 🤝 Contribution

Ce projet documente l'installation 100% native d'une stack LLM sur ancien hardware workstation. 

**Objectifs** :
- Maximiser les performances avec hardware limité
- Éviter la complexité Docker
- Contrôle total pour fine-tuning
- Documentation détaillée pour reproductibilité

## 📝 License

MIT

## ✅ Checklist d'Installation

### Sur le Serveur Z800
- [ ] Ubuntu 22.04 à jour
- [ ] Pilotes NVIDIA installés
- [ ] Ollama installé en natif
- [ ] Modèles téléchargés (mistral)
- [ ] Configuration réseau (0.0.0.0:11434)
- [ ] Firewall configuré
- [ ] Swap optimisé (8 GB)

### Sur Votre PC
- [ ] Msty installé
- [ ] Connexion au serveur configurée
- [ ] Test de chat réussi
- [ ] Scripts Python fonctionnels (optionnel)

## 🎯 Roadmap

- [x] Installation Ollama native
- [x] Configuration accès distant
- [x] Documentation client PC (Msty)
- [ ] Configuration RAG
- [ ] Setup fine-tuning complet
- [ ] Datasets exemple
- [ ] Guide LoRA/QLoRA
- [ ] Scripts d'entraînement
- [ ] Monitoring avancé

---

**⚡ Fait avec passion sur un HP Z800 de 2009 - Preuve que le vieux hardware peut encore servir !**
