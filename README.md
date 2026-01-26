# Installation d'un Agent LLM sur HP Z800 Workstation

## 📊 Spécifications du Serveur

| Composant | Détails |
|-----------|---------|
| **Modèle** | HP Z800 Workstation (2009-2010) |
| **CPU** | 2× Intel Xeon E5640 @ 2.67 GHz (8 cœurs, 16 threads) |
| **RAM** | 12 GB DDR3 ECC (~4 GB utilisables actuellement) |
| **GPU** | NVIDIA Quadro P4000 (8 GB GDDR5, 1792 CUDA cores) |
| **Wi-Fi** | Adaptateur USB Realtek RTL8192EU (ID: 0bda:818b) |
| **OS** | Ubuntu Server 22.04 LTS (noyau 6.8.0-40-generic HWE) |
| **Pilotes NVIDIA** | ✅ Installés |

## 💾 Configuration des Disques

```
Disque 1: Système (/)
Disque 2: Conda & ML (pour environnements et entraînement)
```

## 🎯 Objectif du Projet

Installation et configuration d'un agent LLM local pour:
- Inférence de modèles de langage
- Expérimentation ML/AI
- Hébergement local sans dépendance cloud

## 📋 Plan d'Installation

### Phase 1: Préparation du Système ✅
- [x] Pilotes NVIDIA installés
- [ ] Vérification de l'architecture des disques
- [ ] Configuration des points de montage
- [ ] Installation des dépendances système

### Phase 2: Environnement Python & CUDA
- [ ] Installation de Conda/Miniconda sur Disque 2
- [ ] Configuration de CUDA Toolkit
- [ ] Création d'environnement virtuel pour LLM
- [ ] Installation de PyTorch avec support GPU

### Phase 3: Installation de l'Agent LLM
- [ ] Choix de la solution (Ollama recommandé)
- [ ] Installation et configuration
- [ ] Téléchargement du premier modèle
- [ ] Tests de fonctionnement

### Phase 4: Optimisation & Production
- [ ] Configuration du service systemd
- [ ] Optimisation mémoire/GPU
- [ ] Monitoring et logs
- [ ] Documentation API

## 🔧 Prérequis

```bash
# Vérification des pilotes NVIDIA
nvidia-smi

# Vérification des dis
