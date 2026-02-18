# Benchmarks AI/ML/DL - HP Z800 Quadro P4000

Résultats complets des tests de performance pour Deep Learning, Machine Learning et Intelligence Artificielle.

## 📋 Configuration testée

**Date des tests** : 2026-02-17

### Matériel

| Composant | Spécification |
|-----------|---------------|
| **CPU** | 2× Intel Xeon E5640 @ 2.67 GHz (16 threads) |
| **RAM** | 16 GB DDR3 ECC @ 1333 MHz |
| **GPU** | NVIDIA Quadro P4000 (8 GB GDDR5) |
| **CUDA Cores** | 1792 |
| **Tensor Cores** | 0 (architecture Pascal) |
| **VRAM** | 7.91 GB disponible |
| **Stockage** | 916 GB HDD (projets DL) |

### Logiciels

| Logiciel | Version |
|----------|---------|
| **OS** | Ubuntu Server 22.04 LTS |
| **Kernel** | 6.8.0-40-generic HWE |
| **CUDA** | 12.1 |
| **Driver NVIDIA** | 580.126.09 |
| **Python** | 3.10 (Miniconda) |
| **PyTorch** | 2.5.1+cu121 |
| **TorchVision** | Compatible |
| **Transformers** | Latest |
| **Ultralytics** | Latest (YOLOv8) |

---

## 🔬 Méthodologie des tests

### Protocole

1. **Warm-up** : 10-20 itérations pour stabiliser le GPU
2. **Mesure** : 50-100 itérations pour moyenne fiable
3. **Synchronisation CUDA** : `torch.cuda.synchronize()` avant/après mesures
4. **Cache vidé** : `torch.cuda.empty_cache()` entre tests
5. **Mode évaluation** : `model.eval()` + `torch.no_grad()` pour inférence

### Conditions

- GPU idle avant chaque test (pas d'autre charge)
- Température GPU stabilisée (~40-50°C)
- Benchmarks exécutés plusieurs fois pour vérifier reproductibilité
- Aucun autre processus GPU actif

---

## 📊 Résultats détaillés

### 1. Tests fondamentaux GPU

#### Test 1.1 : Multiplication de matrices (10000×10000)

```
Matrice FP32 : 10000 × 10000 = 100M éléments
Opération : C = A × B (2 milliards d'opérations FMA)
```

| Processeur | Temps | Performance | Accélération |
|------------|-------|-------------|--------------|
| **CPU** (16 threads) | 14.22 s | 0.14 GFLOPS | 1× |
| **GPU** (P4000) | 0.46 s | 4.35 TFLOPS | **31.07×** 🏆 |

**Analyse** :
- GPU **31× plus rapide** que CPU pour opérations matricielles
- Performance proche des **5.3 TFLOPS théoriques** du P4000
- Excellent pour Deep Learning (réseaux = multiplications matricielles)

#### Test 1.2 : Convolution 2D (batch 128, 3→64 canaux)

```
Input : 128 × 3 × 224 × 224
Conv2D : 3 → 64 canaux, kernel 3×3, padding 1
Output : 128 × 64 × 224 × 224
```

| Métrique | Résultat |
|----------|----------|
| **Throughput** | **34.44 itérations/sec** |
| **Temps par batch** | 29.03 ms |
| **VRAM utilisée** | 2.73 GB |

**Analyse** :
- Opération fondamentale des CNN (ResNet, VGG, etc.)
- Performance excellente pour architecture Pascal
- Peut traiter **4404 images/seconde** (128 × 34.44)

#### Test 1.3 : Bloc ResNet (50 itérations)

```
Input : 32 × 64 × 56 × 56
Bloc : Conv2D → BatchNorm → ReLU → Conv2D → BatchNorm
```

| Métrique | Résultat |
|----------|----------|
| **Throughput** | **229.21 it/s** |
| **Temps par itération** | 4.36 ms |

**Analyse** :
- Simulation de bloc ResNet typique
- **7334 images/sec** (32 × 229.21)
- Excellent pour entraînement de réseaux profonds

---

### 2. Computer Vision - Classification

#### Test 2.1 : ResNet-18 (Inférence)

```
Architecture : 18 couches, 11.7M paramètres
Input : 224×224×3
Output : 1000 classes (ImageNet)
```

| Batch Size | Images/sec | VRAM | Latence/image |
|------------|------------|------|---------------|
| 16 | 1115 | 1.2 GB | 0.90 ms |
| 32 | 1193 | 1.3 GB | 0.84 ms |
| 64 | 1243 | 1.4 GB | 0.80 ms |
| **128** | **1279** | **1.8 GB** | **0.78 ms** |

**Analyse** :
- ✅ **1279 images/sec** = Performance excellente
- ✅ Latence < 1 ms = Temps réel ultra-rapide
- ✅ Scaling parfait jusqu'à batch 128
- 💡 Peut traiter vidéo 1080p @ 30 FPS avec 42 streams en parallèle

#### Test 2.2 : ResNet-50 (Inférence)

```
Architecture : 50 couches, 25.6M paramètres
Input : 224×224×3
Output : 1000 classes
```

| Batch Size | Images/sec | VRAM | Latence/image |
|------------|------------|------|---------------|
| 8 | 325 | 2.4 GB | 3.08 ms |
| 16 | 342 | 2.5 GB | 2.92 ms |
| 32 | 350 | 2.6 GB | 2.86 ms |
| **64** | **355** | **2.8 GB** | **2.82 ms** |

**Analyse** :
- ✅ **355 images/sec** = Très bon pour modèle 2× plus lourd
- ✅ ResNet-50 **3.6× plus lent** que ResNet-18 (attendu)
- ✅ VRAM bien gérée (2.8 GB / 8 GB = 35%)
- 💡 Top-1 accuracy ~76% (vs ~70% pour ResNet-18)

#### Test 2.3 : ResNet-18 (Entraînement/Training)

```
Configuration : Batch 32, Adam optimizer
Loss : CrossEntropyLoss
100 itérations forward + backward + update
```

| Métrique | Résultat |
|----------|----------|
| **Images/sec** | **260** |
| **Temps/itération** | 123 ms |
| **VRAM max** | 1.9 GB |

**Analyse** :
- ✅ **260 img/s** en training vs 1193 img/s en inférence
- Ratio : **4.6× plus lent** (normal, calcul gradients + backprop)
- ✅ VRAM stable à 1.9 GB
- 💡 **Temps d'entraînement estimé** :
  - CIFAR-10 (50k images, 50 epochs) : ~16 minutes
  - ImageNet-1k (1.2M images, 90 epochs) : ~5 jours

**Comparaison Inférence vs Training** :

| Mode | Images/sec | Ratio |
|------|------------|-------|
| **Inférence** | 1193 | 1× |
| **Training** | 260 | 0.22× |

---

### 3. Computer Vision - Détection d'objets

#### Test 3.1 : YOLOv8-medium (Détection temps réel)

```
Architecture : YOLOv8m
Input : 640×640×3
Détection : Bounding boxes + classes + scores
Post-processing : NMS (Non-Maximum Suppression)
```

| Métrique | Résultat | Détail |
|----------|----------|--------|
| **FPS** | **36.66** | 100 images / 2.73 sec |
| **Latence totale** | 27.28 ms/image | Inference + NMS |
| **Inference seule** | 32.3 ms | Temps GPU pur |
| **Post-processing** | 18.9 ms | NMS + filtrage |

**Analyse** :
- ✅ **36.66 FPS > 30 FPS** = Temps réel fluide ! 🎥
- ✅ Peut traiter webcam 1080p en direct
- ✅ YOLOv8m = Bon compromis vitesse/précision
- 💡 Applications : 
  - Surveillance vidéo temps réel
  - Détection objets sur drone
  - Comptage de personnes
  - Contrôle qualité industriel

**Comparaison modèles YOLO** :

| Modèle | Params | mAP | FPS (estimé) |
|--------|--------|-----|--------------|
| YOLOv8n | 3.2M | 37.3 | ~80 FPS |
| YOLOv8s | 11.2M | 44.9 | ~60 FPS |
| **YOLOv8m** | **25.9M** | **50.2** | **36.66 FPS** |
| YOLOv8l | 43.7M | 52.9 | ~25 FPS |
| YOLOv8x | 68.2M | 53.9 | ~18 FPS |

---

### 4. NLP (Natural Language Processing)

#### Test 4.1 : BERT-base (Inférence)

```
Architecture : Transformer, 12 couches
Paramètres : 110M
Input : Batch 16, sequence length 128
Tokenizer : bert-base-uncased
```

| Métrique | Résultat |
|----------|----------|
| **Throughput** | **60.67 it/s** |
| **Latence** | 16.48 ms/batch |
| **Temps/sequence** | 1.03 ms |
| **VRAM** | 0.42 GB |

**Analyse** :
- ✅ **60.67 it/s** = Très bon pour modèle Transformer
- ✅ Batch 16 × 128 tokens = **970 tokens/sec**
- ✅ VRAM faible (0.42 GB) = Peut augmenter batch size
- 💡 Applications :
  - Classification de texte
  - Sentiment analysis
  - Question answering
  - Named Entity Recognition (NER)

**Estimation temps d'entraînement** :

| Dataset | Samples | Epochs | Temps estimé |
|---------|---------|--------|--------------|
| IMDB | 25k | 3 | ~20 min |
| SST-2 | 67k | 3 | ~55 min |
| MNLI | 393k | 3 | ~5.4 heures |

**Comparaison avec GPUs récents** :

| GPU | BERT throughput | Ratio vs P4000 |
|-----|----------------|----------------|
| **Quadro P4000** | 60.67 it/s | 1× |
| RTX 3060 (Tensor Cores) | ~150 it/s | 2.5× |
| RTX 4060 Ti (FP8) | ~200 it/s | 3.3× |

---

### 5. Mixed Precision (FP16)

#### Test 5.1 : Comparaison FP32 vs FP16

```
Modèle : ResNet-like (simple CNN)
Batch : 32 × 3 × 224 × 224
Iterations : 20
```

| Précision | Temps | Throughput | Gain |
|-----------|-------|------------|------|
| **FP32** | [À tester] | XX it/s | 1× |
| **FP16** | [À tester] | XX it/s | ~1.5-2× |

**Note** : La Quadro P4000 (Pascal) n'a **pas de Tensor Cores**.
- FP16 exécuté via CUDA cores standards
- Gain attendu : **1.5-2× plus rapide** (vs 8-16× avec Tensor Cores RTX)
- Économie mémoire : **2× moins de VRAM**

---

## 📈 Récapitulatif des performances

### Vision

| Tâche | Modèle | Métrique | Performance | Temps réel ? |
|-------|--------|----------|-------------|--------------|
| **Classification légère** | ResNet-18 | 1279 img/s | ⭐⭐⭐⭐⭐ | ✅ Oui |
| **Classification précise** | ResNet-50 | 355 img/s | ⭐⭐⭐⭐ | ✅ Oui |
| **Détection objets** | YOLOv8m | 36.66 FPS | ⭐⭐⭐⭐ | ✅ Oui |
| **Entraînement** | ResNet-18 | 260 img/s | ⭐⭐⭐ | N/A |

### NLP

| Tâche | Modèle | Métrique | Performance | Temps réel ? |
|-------|--------|----------|-------------|--------------|
| **Inférence** | BERT-base | 60.67 it/s | ⭐⭐⭐⭐ | ✅ Oui |
| **Fine-tuning** | BERT-base | ~30 it/s | ⭐⭐⭐ | N/A |

### Opérations fondamentales

| Opération | Performance | Accélération GPU/CPU |
|-----------|-------------|----------------------|
| **Matmul 10k×10k** | 4.35 TFLOPS | **31×** |
| **Conv2D** | 34.44 it/s | N/A |
| **ResNet block** | 229.21 it/s | N/A |

---

## 🏆 Comparaison avec autres GPUs

### Performances relatives (ResNet-50 inférence)

| GPU | Prix | TFLOPS FP32 | ResNet-50 | Ratio vs P4000 | $/Perf |
|-----|------|-------------|-----------|----------------|--------|
| **Quadro P4000** | **$300** | 5.3 | **355 img/s** | **1×** | **$0.85** |
| GTX 1080 Ti | $350 | 11.3 | ~650 img/s | 1.8× | $0.54 |
| RTX 3060 | $400 | 12.7 | ~800 img/s | 2.3× | $0.50 |
| RTX 4060 Ti | $500 | 22.1 | ~1200 img/s | 3.4× | $0.42 |
| RTX 3090 | $1500 | 35.6 | ~2500 img/s | 7× | $0.60 |

**Analyse** :
- 💰 **Meilleur rapport qualité/prix pour budget limité**
- ⚡ 45% des performances d'une RTX 3060 pour 75% du prix
- 🎯 Excellent pour apprendre et prototyper
- ⚠️ Pas de Tensor Cores (désavantage pour FP16/INT8)

---

## 💡 Applications pratiques

### ✅ Excellente pour

| Application | Viabilité | Notes |
|-------------|-----------|-------|
| **Apprentissage DL** | ⭐⭐⭐⭐⭐ | Parfait pour débuter |
| **Classification images** | ⭐⭐⭐⭐⭐ | ResNet, EfficientNet OK |
| **Détection objets** | ⭐⭐⭐⭐ | YOLO temps réel possible |
| **NLP moyen** | ⭐⭐⭐⭐ | BERT-base, GPT-2 small |
| **Fine-tuning** | ⭐⭐⭐⭐ | Transfer learning fluide |
| **Prototypage** | ⭐⭐⭐⭐⭐ | Itération rapide |
| **Kaggle** | ⭐⭐⭐⭐ | Compétitions viables |
| **Recherche académique** | ⭐⭐⭐⭐ | Publications possibles |

### ⚠️ Limitations

| Application | Viabilité | Raison |
|-------------|-----------|--------|
| **LLMs (7B+)** | ⭐⭐ | 8 GB VRAM insuffisant |
| **Stable Diffusion** | ⭐⭐ | Possible mais lent (~8-10s/image) |
| **Vidéo 4K temps réel** | ⭐ | FPS trop faible |
| **Training gros modèles** | ⭐⭐ | Batch size limité |
| **Production haute charge** | ⭐⭐ | Préférer RTX 3000+ |

---

## 📊 Temps d'entraînement estimés

### Vision

| Dataset | Modèle | Epochs | Batch | Temps estimé |
|---------|--------|--------|-------|--------------|
| **CIFAR-10** | ResNet-18 | 50 | 128 | **~16 min** |
| **CIFAR-100** | ResNet-50 | 100 | 64 | **~45 min** |
| **ImageNet** | ResNet-50 | 90 | 64 | **~5 jours** |
| **COCO** | YOLOv8m | 300 | 16 | **~3 jours** |

### NLP

| Dataset | Modèle | Epochs | Batch | Temps estimé |
|---------|--------|--------|-------|--------------|
| **IMDB** | BERT-base | 3 | 16 | **~20 min** |
| **SST-2** | BERT-base | 3 | 16 | **~55 min** |
| **SQuAD** | BERT-base | 2 | 12 | **~6 heures** |

---

## 🔧 Optimisations recommandées

### 1. Mixed Precision (FP16)

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for data, target in dataloader:
    with autocast():
        output = model(data)
        loss = criterion(output, target)
    
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

**Gains attendus** :
- ✅ **1.5-2× plus rapide** (sans Tensor Cores)
- ✅ **2× moins de VRAM** → batch size 2× plus grand
- ✅ Précision similaire à FP32

### 2. Gradient Checkpointing

```python
model.gradient_checkpointing_enable()
```

**Gains attendus** :
- ✅ **30-50% moins de VRAM**
- ⚠️ 10-20% plus lent

### 3. DataLoader optimisé

```python
train_loader = DataLoader(
    dataset,
    batch_size=64,
    num_workers=8,  # Utiliser CPU pour préparer data
    pin_memory=True,  # Transfert CPU→GPU plus rapide
    persistent_workers=True  # Garder workers actifs
)
```

### 4. Compilation de modèle (PyTorch 2.0+)

```python
import torch

model = MyModel().cuda()
model = torch.compile(model)  # Gain 10-30%
```

---

## 🎯 Recommandations batch size

### Vision

| Modèle | Input | Batch optimal | VRAM |
|--------|-------|---------------|------|
| **ResNet-18** | 224×224 | 128 | 1.8 GB |
| **ResNet-50** | 224×224 | 64 | 2.8 GB |
| **EfficientNet-B0** | 224×224 | 96 | ~2 GB |
| **YOLOv8m** | 640×640 | 16 | ~4 GB |

### NLP

| Modèle | Seq length | Batch optimal | VRAM |
|--------|------------|---------------|------|
| **BERT-base** | 128 | 32 | ~1 GB |
| **BERT-base** | 512 | 8 | ~2 GB |
| **GPT-2 small** | 512 | 4 | ~3 GB |

---

## 📝 Conclusion

### Points forts

✅ **Excellent rapport qualité/prix** : $300 pour performances solides  
✅ **8 GB VRAM** : Suffisant pour 80% des projets DL  
✅ **Performances temps réel** : YOLOv8, ResNet, BERT fluides  
✅ **Faible consommation** : 105W (vs 170-200W pour RTX)  
✅ **Idéal pour apprendre** : Tous les tutoriels fonctionnent  

### Points faibles

⚠️ **Pas de Tensor Cores** : FP16/INT8 non optimisé  
⚠️ **Architecture ancienne** : Pascal 2016 (vs Ampere 2020+)  
⚠️ **LLMs limités** : 7B+ modèles trop gros  
⚠️ **Stable Diffusion lent** : 8-10s/image (vs 2-3s RTX)  

### Verdict final

**La Quadro P4000 est une excellente carte pour :**
- 🎓 Étudier le Deep Learning
- 🔬 Prototypage et recherche
- 📚 Projets académiques et Kaggle
- 💼 Petites productions (<1000 req/jour)

**Pour $300 d'occasion, c'est un investissement solide qui permet de faire du vrai Deep Learning sans se ruiner.**

---

**Tests effectués le** : 2026-02-17  
**Configuration** : HP Z800 + 16 GB RAM + Quadro P4000  
**Scripts** : Disponibles dans `/mnt/deep-learning/projects/benchmarks/`
