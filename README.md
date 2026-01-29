# hp-z800-ai-agent

## 🎯 Objectif du Projet

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
4. **[Téléchargement Modèles](docs/04-client-pc.md)** - Msty
5. **[Installation Client PC](docs/05-client-pc.md)** - Msty, Jan ou scripts
6. **[Configuration RAG](docs/06-configuration-rag.md)** - Documents et embeddings
7. **[Setup Fine-tuning](docs/07-finetuning-setup.md)** - Environnement Conda
8. **[Optimisation Swap](docs/08-optimisation-swap.md)** - Gestion mémoire

## 💡 Utilisation

### Depuis Votre PC - Interface Graphique (Msty/Jan)

1. Ouvrir Msty
2. Settings → Ollama Server → `http://IP-Z800:11434`
3. Sélectionner le modèle "mistral"
4. Commencer à chatter !

### Depuis Votre PC - Scripts Python

```python
# chat-simple.py
import requests

OLLAMA_URL = "http://192.168.1.XXX:11434"  # Votre IP Z800

def ask_mistral(prompt):
    response = requests.post(
        f"{OLLAMA_URL}/api/generate",
        json={
            "model": "mistral",
            "prompt": prompt,
            "stream": False
        }
    )
    return response.json()['response']

# Utilisation
result = ask_mistral("Qu'est-ce que le machine learning?")
print(result)
```

### API REST Directe

```bash
# Depuis votre PC
curl http://IP-Z800:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "Bonjour!",
  "stream": false
}'
```

## 🎓 Fonctionnalités

### Chat de Base

- ✅ Interface graphique moderne (Msty/Jan)
- ✅ Scripts Python personnalisables
- ✅ API REST compatible OpenAI
- ✅ Streaming des réponses
- ✅ Multi-modèles

### RAG (Retrieval Augmented Generation)

Interrogez vos documents :

```python
# rag-query.py
import requests

def rag_query(question, context):
    prompt = f"""Contexte: {context}
    
Question: {question}

Réponds en te basant uniquement sur le contexte fourni."""
    
    response = requests.post(
        f"{OLLAMA_URL}/api/generate",
        json={"model": "mistral", "prompt": prompt, "stream": False}
    )
    return response.json()['response']
```

Formats supportés :
- PDF, DOCX, TXT, MD, CSV
- Extraction automatique avec Python
- Embeddings avec nomic-embed-text

### Fine-tuning

Environnement Conda dédié pour :
- Fine-tuner Mistral sur vos données
- Créer des modèles spécialisés
- Entraîner des adapters LoRA/QLoRA
- Utiliser tout le RAM + Swap (jusqu'à 20 GB)

```bash
# Activer l'environnement
conda activate finetuning

# Voir docs/07-finetuning-setup.md
```

## 📁 Structure du Projet

```
hp-z800-ai-agent/
├── README.md                      # Ce fichier
├── docs/                          # Documentation détaillée
│   ├── 01-preparation.md
│   ├── 02-installation-ollama.md
│   ├── 03-configuration-reseau.md
│   ├── 04-download-models.md
│   ├── 05-client-pc.md
│   ├── 06-configuration-rag.md
│   ├── 07-finetuning-setup.md
│   ├── 08-optimisation-swap.md
│   └── troubleshooting.md
├── scripts/                       # Scripts d'installation
│   ├── install-all-native.sh     # Installation complète
│   ├── install-ollama.sh         # Ollama seul
│   ├── configure-network.sh      # Configuration réseau
│   ├── setup-swap.sh             # Configuration swap
│   ├── setup-finetuning-env.sh   # Env Conda fine-tuning
│   ├── test-gpu.sh               # Test NVIDIA
│   └── monitor.sh                # Monitoring ressources
├── examples/                      # Exemples client PC
│   ├── chat-client.py            # Client CLI simple
│   ├── chat-gui-streamlit.py     # Interface web légère
│   ├── rag-pdf.py                # RAG avec PDFs
│   ├── rag-documents.py          # RAG multi-documents
│   └── batch-processing.py       # Traitement par lots
├── config/                        # Configurations
│   ├── ollama.service            # Systemd service
│   └── firewall-rules.sh         # Règles UFW
└── logs/                          # Logs d'installation
    └── .gitkeep
```

## 🛠️ Modèles Disponibles

| Modèle | Taille | Usage | RAM Requise |
|--------|--------|-------|-------------|
| **mistral:7b** | 4.1 GB | Chat principal | 6 GB |
| **mistral:instruct** | 4.1 GB | Instructions | 6 GB |
| **nomic-embed-text** | 274 MB | Embeddings RAG | 1 GB |
| llama3.2 | 2 GB | Alternative légère | 4 GB |
| codellama:7b | 3.8 GB | Génération code | 6 GB |
| deepseek-coder | 3.8 GB | Code spécialisé | 6 GB |

```bash
# Télécharger un modèle
ollama pull nom-du-modele

# Lister les modèles
ollama list

# Supprimer un modèle
ollama rm nom-du-modele
```

## 📊 Utilisation des Ressources

### Comparaison Native vs Docker

| Composant | Native | Docker | Économie |
|-----------|--------|--------|----------|
| Ollama + Mistral | 4.1 GB | 4.5 GB | 400 MB |
| Interface | 0 MB* | 800 MB | 800 MB |
| Runtime | 0 MB | 300 MB | 300 MB |
| **Total utilisé** | **4.1 GB** | **5.6 GB** | **1.5 GB** |
| **RAM disponible** | **~8 GB** | **~6.5 GB** | **+23%** |

*Interface sur votre PC, pas sur le serveur

### Configuration Optimale

```bash
# RAM physique: 12 GB
# Ollama + modèles: ~4-5 GB
# Système Ubuntu: ~1 GB
# Disponible: ~6-7 GB

# Swap recommandé: 8 GB
# Total mémoire virtuelle: 20 GB
# → Suffisant pour fine-tuning !
```

## 🔒 Sécurité

### Firewall Configuration

```bash
# Autoriser uniquement votre PC
sudo ufw allow from VOTRE-IP-PC to any port 11434
sudo ufw allow 22/tcp  # SSH
sudo ufw enable
```

### Tunnel SSH (Alternative)

```bash
# Depuis votre PC - Accès sécurisé sans ouvrir de ports
ssh -L 11434:localhost:11434 user@IP-Z800

# Puis utiliser: http://localhost:11434
```

### Réseau Local Uniquement

```bash
# Bind Ollama sur IP locale uniquement
# Dans /etc/systemd/system/ollama.service.d/override.conf
Environment="OLLAMA_HOST=192.168.1.XXX:11434"
```

## 📊 Monitoring

```bash
# GPU en temps réel
watch -n 1 nvidia-smi

# Utilisation ressources
./scripts/monitor.sh

# Logs Ollama
sudo journalctl -u ollama -f

# Statut service
sudo systemctl status ollama
```

## 🔧 Scripts Utiles

```bash
# Test complet GPU + CUDA
./scripts/test-gpu.sh

# Configurer swap optimal
./scripts/setup-swap.sh

# Créer environnement fine-tuning
./scripts/setup-finetuning-env.sh

# Monitoring continu
./scripts/monitor.sh
```

## 🆘 Dépannage

### Problème : Impossible de se connecter depuis le PC

```bash
# Sur le Z800 - Vérifier qu'Ollama écoute sur 0.0.0.0
sudo netstat -tlnp | grep 11434
# Devrait afficher: 0.0.0.0:11434

# Vérifier le firewall
sudo ufw status

# Tester localement
curl http://localhost:11434/api/tags
```

### Problème : Modèle trop lent

```bash
# Vérifier que GPU est utilisée
nvidia-smi

# Vérifier les variables CUDA
env | grep CUDA

# Forcer l'utilisation GPU
export CUDA_VISIBLE_DEVICES=0
sudo systemctl restart ollama
```

### Problème : Manque de mémoire

```bash
# Vérifier swap
swapon --show

# Augmenter swap (voir docs/08-optimisation-swap.md)
./scripts/setup-swap.sh

# Utiliser un modèle plus léger
ollama pull llama3.2  # 2GB au lieu de 4GB
```

Voir [docs/troubleshooting.md](docs/troubleshooting.md) pour plus de solutions.

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
- [ ] Modèles téléchargés (mistral + nomic-embed-text)
- [ ] Configuration réseau (0.0.0.0:11434)
- [ ] Firewall configuré
- [ ] Swap optimisé (8 GB)
- [ ] Environnement Conda fine-tuning créé

### Sur Votre PC
- [ ] Msty ou Jan installé
- [ ] Connexion au serveur configurée
- [ ] Test de chat réussi
- [ ] Scripts Python fonctionnels (optionnel)

## 🎯 Roadmap

- [x] Installation Ollama native
- [x] Configuration accès distant
- [x] Documentation client PC
- [x] Configuration RAG
- [ ] Setup fine-tuning complet
- [ ] Datasets exemple
- [ ] Guide LoRA/QLoRA
- [ ] Scripts d'entraînement
- [ ] Monitoring avancé

---

**⚡ Fait avec passion sur un HP Z800 de 2009 - Preuve que le vieux hardware peut encore servir !**
