# hp-z800-ai-agent

🤖 Stack LLM complète sur HP Z800 : Ollama + Open WebUI + RAG + Fine-tuning

## 🎯 Objectif du Projet

Transformer le HP Z800 en serveur d'IA local avec :
- **Chatbot accessible depuis n'importe quel PC** via interface web
- **RAG (Retrieval Augmented Generation)** pour interroger vos documents
- **Fine-tuning** de modèles personnalisés
- **GPU acceleration** avec NVIDIA Quadro P4000

## 📊 Spécifications du Serveur

| Composant | Détails |
|-----------|---------|
| **Modèle** | HP Z800 Workstation (2009-2010) |
| **CPU** | 2× Intel Xeon E5640 @ 2.67 GHz (8 cœurs, 16 threads) |
| **RAM** | 12 GB DDR3 ECC |
| **GPU** | NVIDIA Quadro P4000 (8 GB GDDR5, 1792 CUDA cores) |
| **OS** | Ubuntu Server 22.04 LTS (noyau 6.8.0-40-generic HWE) |
| **Storage** | Disque 1: Système / Disque 2: Conda & ML |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│           HP Z800 (Serveur Local)               │
├─────────────────────────────────────────────────┤
│                                                  │
│  🔷 Ollama (Port 11434)                         │
│     ├─ mistral (LLM principal)                  │
│     └─ nomic-embed-text (embeddings RAG)        │
│                                                  │
│  🔷 Open WebUI (Port 3000) [Docker]             │
│     ├─ Interface chat web                       │
│     ├─ Upload & gestion documents               │
│     ├─ RAG automatique                          │
│     └─ Collections de connaissances             │
│                                                  │
│  🔷 Conda Environments (Disque 2)               │
│     └─ fine-tuning (PyTorch, Transformers)      │
│                                                  │
└─────────────────────────────────────────────────┘
         ↑
         │ http://IP-Z800:3000
         │
    [Votre PC - Navigateur Web]
```

## 🚀 Installation Rapide

### Prérequis
- ✅ Ubuntu Server 22.04 LTS installé
- ✅ Pilotes NVIDIA installés
- ✅ Conda installé
- ✅ Accès sudo

### Installation Automatique

```bash
# Cloner le projet
git clone https://github.com/VOTRE-USERNAME/hp-z800-ai-agent.git
cd hp-z800-ai-agent

# Lancer l'installation complète
chmod +x scripts/install-all.sh
./scripts/install-all.sh
```

## 📚 Installation Manuelle (Étape par Étape)

Si vous préférez installer manuellement, suivez ces guides dans l'ordre :

1. **[Préparation](docs/01-preparation.md)** - Vérifications système
2. **[Installation Docker](docs/02-installation-docker.md)** - Docker & Docker Compose
3. **[Installation Ollama](docs/03-installation-ollama.md)** - Backend LLM
4. **[Installation Open WebUI](docs/04-installation-open-webui.md)** - Interface web
5. **[Configuration RAG](docs/05-configuration-rag.md)** - Modèles d'embedding
6. **[Accès Distant](docs/06-acces-distant.md)** - Configurer l'accès depuis votre PC
7. **[Fine-tuning Setup](docs/07-finetuning-setup.md)** - Environnement pour fine-tuning

## 💡 Utilisation

### Accès à l'Interface Web

```
http://IP-DU-Z800:3000
```

**Première connexion** :
1. Créer un compte (local, pas de cloud)
2. Sélectionner le modèle "mistral"
3. Commencer à discuter !

### Upload de Documents (RAG)

1. Cliquez sur **"+"** → **"Upload Files"**
2. Sélectionnez vos fichiers (PDF, DOCX, TXT, MD, CSV)
3. Le système indexe automatiquement
4. Posez des questions sur vos documents !

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

## 🎓 Fonctionnalités Avancées

### RAG (Retrieval Augmented Generation)

Interrogez vos propres documents :
- Documentation technique
- Manuels d'utilisation
- Articles de recherche
- Notes personnelles
- Bases de connaissances

### Fine-tuning (À venir)

Environnement Conda configuré pour :
- Fine-tuner Mistral sur vos données
- Créer des modèles spécialisés
- Entraîner des adapters LoRA

```bash
# Activer l'environnement
conda activate finetuning

# Voir docs/07-finetuning-setup.md pour plus de détails
```

## 📁 Structure du Projet

```
hp-z800-ai-agent/
├── README.md                      # Ce fichier
├── docs/                          # Documentation détaillée
│   ├── 01-preparation.md
│   ├── 02-installation-docker.md
│   ├── 03-installation-ollama.md
│   ├── 04-installation-open-webui.md
│   ├── 05-configuration-rag.md
│   ├── 06-acces-distant.md
│   ├── 07-finetuning-setup.md
│   └── troubleshooting.md
├── scripts/                       # Scripts d'installation
│   ├── install-all.sh            # Installation complète
│   ├── install-docker.sh
│   ├── install-ollama.sh
│   ├── install-open-webui.sh
│   ├── setup-finetuning-env.sh
│   ├── test-gpu.sh
│   └── monitor.sh
├── config/                        # Configurations
│   ├── docker-compose.yml
│   ├── ollama.service
│   └── open-webui.env
├── examples/                      # Exemples de code
│   ├── api-chat.py
│   ├── rag-query.py
│   └── batch-processing.py
└── logs/                          # Logs d'installation
    └── .gitkeep
```

## 🔧 Scripts Utiles

```bash
# Monitoring GPU en temps réel
./scripts/monitor.sh

# Test complet de la configuration
./scripts/test-gpu.sh

# Redémarrer tous les services
docker restart open-webui
sudo systemctl restart ollama
```

## 🛠️ Modèles Disponibles

| Modèle | Taille | Usage | RAM Requise |
|--------|--------|-------|-------------|
| **mistral** | 4.1 GB | Chat principal | 8 GB |
| **nomic-embed-text** | 274 MB | Embeddings RAG | 2 GB |
| llama3.2 | 2 GB | Alternative légère | 4 GB |
| codellama | 3.8 GB | Code generation | 8 GB |

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
sudo ufw allow from VOTRE-IP to any port 3000
sudo ufw allow from VOTRE-IP to any port 11434
sudo ufw enable
```

### Tunnel SSH (Alternative sécurisée)

```bash
# Depuis votre PC
ssh -L 3000:localhost:3000 -L 11434:localhost:11434 user@IP-Z800

# Puis accédez à http://localhost:3000
```

## 📊 Monitoring

```bash
# Vérifier l'utilisation GPU
watch -n 1 nvidia-smi

# Logs Open WebUI
docker logs -f open-webui

# Logs Ollama
sudo journalctl -u ollama -f

# Statut des services
docker ps
sudo systemctl status ollama
```

## 🆘 Dépannage

### Problème : Open WebUI ne démarre pas

```bash
docker logs open-webui
docker restart open-webui
```

### Problème : Modèle ne charge pas

```bash
# Vérifier l'espace disque
df -h

# Vérifier la mémoire
free -h

# Réinstaller le modèle
ollama rm mistral
ollama pull mistral
```

### Problème : GPU non utilisée

```bash
# Vérifier NVIDIA
nvidia-smi

# Vérifier les variables d'environnement Ollama
sudo systemctl show ollama | grep CUDA
```

Voir [docs/troubleshooting.md](docs/troubleshooting.md) pour plus de solutions.

## 📖 Ressources

- [Documentation Ollama](https://github.com/ollama/ollama)
- [Documentation Open WebUI](https://docs.openwebui.com)
- [Guide RAG avec Ollama](https://ollama.com/blog/embedding-models)
- [Fine-tuning LLMs](https://huggingface.co/docs/transformers/training)

## 🤝 Contribution

Ce projet documente l'installation complète d'une stack LLM sur ancien hardware workstation. N'hésitez pas à :
- Signaler des bugs
- Proposer des améliorations
- Partager vos configurations

## 📝 License

MIT

## ✅ Roadmap

- [x] Installation Ollama
- [x] Installation Open WebUI
- [x] Configuration RAG
- [x] Accès distant sécurisé
- [ ] Fine-tuning environment
- [ ] Datasets de fine-tuning
- [ ] Modèles custom
- [ ] API avancée
- [ ] Monitoring dashboards

---

**Fait avec ❤️ sur un HP Z800 de 2009 qui tourne encore comme une horloge**
```bash
# Vérification des pilotes NVIDIA
nvidia-smi

# Vérification des dis
