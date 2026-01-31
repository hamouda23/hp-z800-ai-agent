# Étape 5 : Configuration RAG (Retrieval Augmented Generation)

## 🎯 Objectif

Configurer le RAG pour permettre à Mistral d'interroger vos propres documents (PDF, DOCX, TXT, etc.) et de répondre en se basant sur leur contenu.

## ⏱️ Durée Estimée

20-30 minutes

## 📋 Prérequis

- ✅ Ollama installé et fonctionnel
- ✅ Mistral téléchargé
- ✅ Python 3.8+ disponible (via Conda)
- ✅ Accès réseau configuré

---

## 🧠 Qu'est-ce que le RAG ?

**RAG (Retrieval Augmented Generation)** permet à un LLM de :
1. **Récupérer** des informations pertinentes depuis vos documents
2. **Augmenter** le contexte avec ces informations
3. **Générer** une réponse basée sur vos documents

### Cas d'Usage
- 📚 Interroger votre documentation technique
- 📄 Analyser des rapports/articles
- 📋 Extraire des informations de contrats
- 🔍 Rechercher dans vos notes personnelles
- 📊 Analyser des données textuelles

---

## 🏗️ Architecture RAG

```
[Vos Documents]
     ↓
[Extraction Texte]
     ↓
[Découpage en Chunks]
     ↓
[Génération Embeddings] ← nomic-embed-text
     ↓
[Base Vectorielle]
     ↓
[Question Utilisateur]
     ↓
[Recherche Similarité]
     ↓
[Context + Question] → [Mistral] → [Réponse]
```

---

## 📦 Étape 1 : Télécharger le Modèle d'Embeddings

```bash
# Télécharger nomic-embed-text (~274 MB)
ollama pull nomic-embed-text

# Vérifier
ollama list
```

**✅ Résultat attendu :**
```
NAME                   ID              SIZE
mistral:latest         xxx...          4.1 GB
nomic-embed-text:latest xxx...         274 MB
```

---

## 🐍 Étape 2 : Créer un Environnement Python pour RAG

```bash
# Créer un nouvel environnement Conda
conda create -n rag python=3.11 -y

# Activer l'environnement
conda activate rag

# Installer les dépendances
pip install requests
pip install pypdf2           # Pour les PDFs
pip install python-docx      # Pour les DOCX
pip install chromadb         # Base vectorielle
pip install sentence-transformers  # Optionnel
```

---

## 📁 Étape 3 : Créer la Structure de Projet RAG

```bash
# Créer les dossiers
mkdir -p ~/hp-z800-ai-agent/rag/{documents,scripts,vectordb}

# Structure finale
# rag/
# ├── documents/        # Vos documents sources
# ├── scripts/          # Scripts Python
# └── vectordb/         # Base vectorielle (ChromaDB)
```

---

## 🔧 Étape 4 : Script d'Extraction de Texte

```bash
# Créer le script d'extraction
cat > ~/hp-z800-ai-agent/rag/scripts/extract_text.py << 'EOF'
#!/usr/bin/env python3
"""
Extrait le texte de différents formats de documents
"""

import os
from pathlib import Path
from typing import List, Dict

def extract_from_txt(file_path: str) -> str:
    """Extrait le texte d'un fichier TXT"""
    with open(file_path, 'r', encoding='utf-8') as f:
        return f.read()

def extract_from_pdf(file_path: str) -> str:
    """Extrait le texte d'un PDF"""
    try:
        import PyPDF2
        text = ""
        with open(file_path, 'rb') as f:
            pdf_reader = PyPDF2.PdfReader(f)
            for page in pdf_reader.pages:
                text += page.extract_text() + "\n"
        return text
    except ImportError:
        return "Erreur: PyPDF2 non installé. pip install pypdf2"

def extract_from_docx(file_path: str) -> str:
    """Extrait le texte d'un DOCX"""
    try:
        from docx import Document
        doc = Document(file_path)
        text = "\n".join([para.text for para in doc.paragraphs])
        return text
    except ImportError:
        return "Erreur: python-docx non installé. pip install python-docx"

def extract_text(file_path: str) -> str:
    """Détecte le format et extrait le texte"""
    ext = Path(file_path).suffix.lower()
    
    if ext == '.txt':
        return extract_from_txt(file_path)
    elif ext == '.pdf':
        return extract_from_pdf(file_path)
    elif ext == '.docx':
        return extract_from_docx(file_path)
    else:
        return f"Format non supporté: {ext}"

def chunk_text(text: str, chunk_size: int = 500, overlap: int = 50) -> List[str]:
    """Découpe le texte en chunks avec chevauchement"""
    words = text.split()
    chunks = []
    
    for i in range(0, len(words), chunk_size - overlap):
        chunk = " ".join(words[i:i + chunk_size])
        chunks.append(chunk)
    
    return chunks

if __name__ == "__main__":
    # Test
    import sys
    if len(sys.argv) > 1:
        file_path = sys.argv[1]
        text = extract_text(file_path)
        print(f"Texte extrait ({len(text)} caractères):")
        print(text[:500] + "...")
        
        chunks = chunk_text(text)
        print(f"\nDécoupé en {len(chunks)} chunks")
    else:
        print("Usage: python extract_text.py <chemin_fichier>")
EOF

chmod +x ~/hp-z800-ai-agent/rag/scripts/extract_text.py
```

---

## 🔢 Étape 5 : Script de Génération d'Embeddings

```bash
# Créer le script d'embeddings
cat > ~/hp-z800-ai-agent/rag/scripts/generate_embeddings.py << 'EOF'
#!/usr/bin/env python3
"""
Génère des embeddings avec Ollama (nomic-embed-text)
"""

import requests
import json
from typing import List

OLLAMA_URL = "http://localhost:11434"

def generate_embedding(text: str) -> List[float]:
    """Génère un embedding pour un texte"""
    response = requests.post(
        f"{OLLAMA_URL}/api/embeddings",
        json={
            "model": "nomic-embed-text",
            "prompt": text
        }
    )
    
    if response.status_code == 200:
        return response.json()['embedding']
    else:
        raise Exception(f"Erreur API: {response.status_code}")

def generate_embeddings_batch(texts: List[str]) -> List[List[float]]:
    """Génère des embeddings pour plusieurs textes"""
    embeddings = []
    total = len(texts)
    
    for i, text in enumerate(texts, 1):
        print(f"Génération embedding {i}/{total}...", end='\r')
        embedding = generate_embedding(text)
        embeddings.append(embedding)
    
    print(f"\n✅ {total} embeddings générés")
    return embeddings

if __name__ == "__main__":
    # Test
    test_text = "Ceci est un test d'embedding avec Ollama."
    embedding = generate_embedding(test_text)
    print(f"Embedding généré: dimension {len(embedding)}")
    print(f"Premiers éléments: {embedding[:5]}")
EOF

chmod +x ~/hp-z800-ai-agent/rag/scripts/generate_embeddings.py
```

---

## 📚 Étape 6 : Script RAG Complet (Simple - Sans ChromaDB)

```bash
# Version simple sans base vectorielle
cat > ~/hp-z800-ai-agent/rag/scripts/simple_rag.py << 'EOF'
#!/usr/bin/env python3
"""
RAG Simple : Recherche dans documents et génération de réponse
"""

import requests
import json
import os
from pathlib import Path
from extract_text import extract_text, chunk_text
from generate_embeddings import generate_embedding

OLLAMA_URL = "http://localhost:11434"

def cosine_similarity(vec1, vec2):
    """Calcule la similarité cosinus entre deux vecteurs"""
    import math
    dot_product = sum(a * b for a, b in zip(vec1, vec2))
    magnitude1 = math.sqrt(sum(a * a for a in vec1))
    magnitude2 = math.sqrt(sum(b * b for b in vec2))
    
    if magnitude1 == 0 or magnitude2 == 0:
        return 0
    
    return dot_product / (magnitude1 * magnitude2)

def load_documents(docs_folder: str):
    """Charge et traite tous les documents"""
    documents = []
    docs_path = Path(docs_folder)
    
    for file_path in docs_path.glob('*'):
        if file_path.suffix.lower() in ['.txt', '.pdf', '.docx']:
            print(f"Chargement: {file_path.name}")
            text = extract_text(str(file_path))
            chunks = chunk_text(text, chunk_size=500)
            
            for i, chunk in enumerate(chunks):
                documents.append({
                    'source': file_path.name,
                    'chunk_id': i,
                    'text': chunk,
                    'embedding': None  # Sera généré à la demande
                })
    
    print(f"\n✅ {len(documents)} chunks chargés depuis {len(list(docs_path.glob('*')))} documents")
    return documents

def search_documents(query: str, documents: list, top_k: int = 3):
    """Recherche les chunks les plus pertinents"""
    print(f"\n🔍 Recherche pour: '{query}'")
    
    # Générer embedding de la question
    query_embedding = generate_embedding(query)
    
    # Calculer similarité avec chaque document
    results = []
    for i, doc in enumerate(documents):
        # Générer embedding du document si pas déjà fait
        if doc['embedding'] is None:
            print(f"Génération embedding document {i+1}/{len(documents)}...", end='\r')
            doc['embedding'] = generate_embedding(doc['text'])
        
        similarity = cosine_similarity(query_embedding, doc['embedding'])
        results.append({
            'document': doc,
            'similarity': similarity
        })
    
    # Trier par similarité
    results.sort(key=lambda x: x['similarity'], reverse=True)
    
    print(f"\n✅ Top {top_k} résultats:")
    for i, result in enumerate(results[:top_k], 1):
        print(f"{i}. {result['document']['source']} (similarité: {result['similarity']:.3f})")
    
    return results[:top_k]

def generate_answer(query: str, context_docs: list):
    """Génère une réponse avec Mistral en utilisant le contexte"""
    
    # Construire le contexte
    context = "\n\n".join([
        f"[Source: {doc['document']['source']}]\n{doc['document']['text']}"
        for doc in context_docs
    ])
    
    # Prompt avec contexte
    prompt = f"""Contexte extrait de documents:

{context}

Question: {query}

Réponds à la question en te basant UNIQUEMENT sur le contexte fourni ci-dessus. Si l'information n'est pas dans le contexte, dis-le clairement."""

    # Appel à Mistral
    print("\n💬 Génération de la réponse...")
    response = requests.post(
        f"{OLLAMA_URL}/api/generate",
        json={
            "model": "mistral",
            "prompt": prompt,
            "stream": False
        }
    )
    
    if response.status_code == 200:
        return response.json()['response']
    else:
        return f"Erreur API: {response.status_code}"

def rag_query(query: str, docs_folder: str, top_k: int = 3):
    """Pipeline RAG complet"""
    print("="*60)
    print("RAG Pipeline")
    print("="*60)
    
    # 1. Charger documents
    documents = load_documents(docs_folder)
    
    # 2. Rechercher documents pertinents
    relevant_docs = search_documents(query, documents, top_k)
    
    # 3. Générer réponse
    answer = generate_answer(query, relevant_docs)
    
    print("\n" + "="*60)
    print("RÉPONSE:")
    print("="*60)
    print(answer)
    print("="*60)
    
    return answer

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python simple_rag.py <question>")
        print('Exemple: python simple_rag.py "Quelle est la conclusion du rapport ?"')
        sys.exit(1)
    
    query = " ".join(sys.argv[1:])
    docs_folder = "../documents"
    
    rag_query(query, docs_folder)
EOF

chmod +x ~/hp-z800-ai-agent/rag/scripts/simple_rag.py
```

---

## 🧪 Étape 7 : Test du RAG

### 7.1 Créer un Document de Test

```bash
# Créer un document test
cat > ~/hp-z800-ai-agent/rag/documents/test-doc.txt << 'EOF'
Guide d'Utilisation du HP Z800

Le HP Z800 est une workstation professionnelle lancée en 2009-2010.

Spécifications:
- Processeurs: Dual Intel Xeon E5640 @ 2.67 GHz
- RAM: Jusqu'à 192 GB DDR3 ECC
- GPU: Support des cartes professionnelles NVIDIA Quadro

Configuration Réseau:
Pour configurer le réseau, utilisez la commande ip addr show.
Le pare-feu peut être configuré avec ufw.

Maintenance:
Il est recommandé de nettoyer les ventilateurs tous les 6 mois.
La pâte thermique du CPU doit être changée tous les 3-4 ans.

Conclusion:
Le Z800 reste une excellente machine pour le machine learning et l'IA en 2026.
EOF
```

### 7.2 Tester l'Extraction

```bash
# Activer l'environnement
conda activate rag

# Tester extraction
cd ~/hp-z800-ai-agent/rag/scripts
python extract_text.py ../documents/test-doc.txt
```

### 7.3 Tester les Embeddings

```bash
python generate_embeddings.py
```

### 7.4 Tester le RAG Complet

```bash
# Question sur le document
python simple_rag.py "Quelles sont les spécifications du HP Z800 ?"

# Autre question
python simple_rag.py "À quelle fréquence doit-on changer la pâte thermique ?"
```

**✅ Résultat attendu :**
```
==========================================================
RAG Pipeline
==========================================================
Chargement: test-doc.txt

✅ 2 chunks chargés depuis 1 documents

🔍 Recherche pour: 'Quelles sont les spécifications du HP Z800 ?'
Génération embedding document 1/2...
Génération embedding document 2/2...

✅ Top 3 résultats:
1. test-doc.txt (similarité: 0.856)
2. test-doc.txt (similarité: 0.723)

💬 Génération de la réponse...

==========================================================
RÉPONSE:
==========================================================
Selon le document, les spécifications du HP Z800 sont:
- Processeurs: Dual Intel Xeon E5640 @ 2.67 GHz
- RAM: Jusqu'à 192 GB DDR3 ECC
- GPU: Support des cartes professionnelles NVIDIA Quadro
==========================================================
```

---

## 📊 Étape 8 : Ajouter Vos Propres Documents

```bash
# Copier vos documents dans le dossier
cp /chemin/vers/vos/documents/*.pdf ~/hp-z800-ai-agent/rag/documents/
cp /chemin/vers/vos/documents/*.docx ~/hp-z800-ai-agent/rag/documents/
# Si le PDF est dans Téléchargements :
scp "$env:USERPROFILE\Downloads\Python code for Artificial Intelligence Foundations of Computational Agents.pdf" samir@192.168.1.108:~/hp-z800-ai-agent/rag/documents/
# Lancer le RAG
cd ~/hp-z800-ai-agent/rag/scripts
python simple_rag.py "Votre question ici"
```

---

## ✅ Checklist de Validation

- [ ] nomic-embed-text téléchargé
- [ ] Environnement Conda `rag` créé
- [ ] Dépendances Python installées
- [ ] Structure de dossiers créée
- [ ] Scripts Python créés
- [ ] Test avec document simple réussi
- [ ] RAG fonctionne sur vos documents

---

## 🔧 Scripts Utiles

### Script de Nettoyage

```bash
# Supprimer tous les embeddings en cache
rm -rf ~/hp-z800-ai-agent/rag/vectordb/*
```

### Script de Statistiques

```bash
cat > ~/hp-z800-ai-agent/rag/scripts/stats.py << 'EOF'
#!/usr/bin/env python3
import os
from pathlib import Path

docs_folder = Path("../documents")
files = list(docs_folder.glob("*"))

print(f"Documents: {len(files)}")
for f in files:
    size = f.stat().st_size / 1024
    print(f"  - {f.name}: {size:.1f} KB")
EOF

python ~/hp-z800-ai-agent/rag/scripts/stats.py
```

---

## 📊 Résumé RAG

À la fin de cette étape :

```
✅ Modèle d'embeddings (nomic-embed-text) installé
✅ Environnement Python RAG configuré
✅ Scripts d'extraction de texte
✅ Scripts de génération d'embeddings
✅ Pipeline RAG fonctionnel
✅ Test avec documents réussi

Capacités:
📄 Support PDF, DOCX, TXT
🔍 Recherche sémantique
💬 Réponses basées sur vos documents
⚡ RAG local, pas de cloud
```

---

## ➡️ Prochaine Étape

**[Étape 6 : Setup Fine-tuning](06-finetuning-setup.md)** *(à venir)*

Préparer l'environnement pour fine-tuner Mistral sur vos propres données.

---

## 🆘 Dépannage

### Problème : "nomic-embed-text not found"

```bash
ollama pull nomic-embed-text
ollama list
```

### Problème : Erreur PyPDF2

```bash
conda activate rag
pip install pypdf2
```

### Problème : Embeddings trop lents

- Utilisez un cache (sauvegardez les embeddings)
- Réduisez la taille des chunks
- Utilisez moins de documents

### Problème : Réponses non pertinentes

- Augmentez `top_k` (plus de contexte)
- Améliorez la qualité des documents sources
- Ajustez la taille des chunks

---

**🎉 Vous pouvez maintenant interroger vos propres documents avec Mistral !**
