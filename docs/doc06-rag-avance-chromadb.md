# Étape 6 : RAG Avancé avec ChromaDB

## 🎯 Objectif

Améliorer le RAG avec ChromaDB pour avoir une base vectorielle persistante et performante.

## ⏱️ Durée Estimée

15-20 minutes

## 📋 Prérequis

- ✅ Étape 5 (RAG simple) complétée
- ✅ Environnement `rag` activé
- ✅ Documents de test fonctionnels

---

## 🚀 Pourquoi ChromaDB ?

### Problèmes du RAG Simple
- ❌ Recalcule les embeddings à chaque requête
- ❌ Lent avec beaucoup de documents
- ❌ Pas de persistance (tout en mémoire)

### Avantages de ChromaDB
- ✅ **Persistance** : Embeddings sauvegardés sur disque
- ✅ **Rapidité** : Index optimisé pour recherche vectorielle
- ✅ **Scalabilité** : Gère facilement 1000+ documents
- ✅ **Métadonnées** : Stocke source, page, date, etc.
- ✅ **Mise à jour** : Ajout/suppression de documents facile

---

## 📦 Étape 1 : Installation de ChromaDB

```bash
# Activer l'environnement
conda activate rag

# Installer ChromaDB
pip install chromadb

# Vérifier l'installation
python -c "import chromadb; print(f'ChromaDB version: {chromadb.__version__}')"
```

---

## 🗂️ Étape 2 : Script d'Indexation avec ChromaDB

```bash
cat > ~/hp-z800-ai-agent/rag/scripts/index_documents.py << 'EOF'
#!/usr/bin/env python3
"""
Indexe tous les documents dans ChromaDB
"""

import chromadb
from pathlib import Path
from extract_text import extract_text, chunk_text
from generate_embeddings import generate_embedding
import hashlib

# Initialiser ChromaDB
client = chromadb.PersistentClient(path="../vectordb")

def get_or_create_collection():
    """Crée ou récupère la collection de documents"""
    try:
        # Essayer de récupérer la collection existante
        collection = client.get_collection(name="documents")
        print(f"📚 Collection existante trouvée: {collection.count()} documents")
    except:
        # Créer une nouvelle collection
        collection = client.create_collection(
            name="documents",
            metadata={"description": "Documents RAG"}
        )
        print("📚 Nouvelle collection créée")
    
    return collection

def document_id(source: str, chunk_id: int) -> str:
    """Génère un ID unique pour un chunk"""
    content = f"{source}_{chunk_id}"
    return hashlib.md5(content.encode()).hexdigest()

def index_document(file_path: Path, collection):
    """Indexe un document dans ChromaDB"""
    print(f"\n📄 Indexation: {file_path.name}")
    
    # Extraire le texte
    text = extract_text(str(file_path))
    
    if "Erreur" in text or "non supporté" in text:
        print(f"  ⚠️  Ignoré: {text}")
        return 0
    
    # Découper en chunks
    chunks = chunk_text(text, chunk_size=500, overlap=50)
    print(f"  📑 {len(chunks)} chunks")
    
    # Indexer chaque chunk
    indexed = 0
    for i, chunk in enumerate(chunks):
        # Générer embedding
        embedding = generate_embedding(chunk)
        
        # Ajouter à ChromaDB
        collection.add(
            ids=[document_id(file_path.name, i)],
            embeddings=[embedding],
            documents=[chunk],
            metadatas=[{
                "source": file_path.name,
                "chunk_id": i,
                "total_chunks": len(chunks),
                "file_type": file_path.suffix
            }]
        )
        
        indexed += 1
        print(f"  ✓ Chunk {i+1}/{len(chunks)}", end='\r')
    
    print(f"  ✅ {indexed} chunks indexés")
    return indexed

def index_all_documents(docs_folder: str):
    """Indexe tous les documents du dossier"""
    print("="*60)
    print("INDEXATION DES DOCUMENTS")
    print("="*60)
    
    collection = get_or_create_collection()
    docs_path = Path(docs_folder)
    
    total_indexed = 0
    total_files = 0
    
    # Parcourir tous les fichiers
    for file_path in docs_path.glob('*'):
        if file_path.suffix.lower() in ['.txt', '.pdf', '.docx']:
            total_files += 1
            indexed = index_document(file_path, collection)
            total_indexed += indexed
    
    print("\n" + "="*60)
    print(f"✅ INDEXATION TERMINÉE")
    print(f"   Fichiers traités: {total_files}")
    print(f"   Chunks indexés: {total_indexed}")
    print(f"   Total en base: {collection.count()}")
    print("="*60)

if __name__ == "__main__":
    import sys
    
    docs_folder = "../documents"
    if len(sys.argv) > 1:
        docs_folder = sys.argv[1]
    
    index_all_documents(docs_folder)
EOF

chmod +x ~/hp-z800-ai-agent/rag/scripts/index_documents.py
```

---

## 🔍 Étape 3 : Script de Recherche avec ChromaDB

```bash
cat > ~/hp-z800-ai-agent/rag/scripts/rag_chromadb.py << 'EOF'
#!/usr/bin/env python3
"""
RAG avec ChromaDB - Version optimisée
"""

import chromadb
import requests
from generate_embeddings import generate_embedding

OLLAMA_URL = "http://localhost:11434"

# Initialiser ChromaDB
client = chromadb.PersistentClient(path="../vectordb")

def search_documents(query: str, top_k: int = 3):
    """Recherche dans ChromaDB"""
    print(f"\n🔍 Recherche pour: '{query}'")
    
    # Récupérer la collection
    try:
        collection = client.get_collection(name="documents")
    except:
        print("❌ Aucune collection trouvée. Lancez d'abord: python index_documents.py")
        return []
    
    # Générer embedding de la question
    query_embedding = generate_embedding(query)
    
    # Rechercher dans ChromaDB
    results = collection.query(
        query_embeddings=[query_embedding],
        n_results=top_k,
        include=["documents", "metadatas", "distances"]
    )
    
    # Afficher les résultats
    print(f"\n✅ Top {top_k} résultats:")
    for i, (doc, metadata, distance) in enumerate(zip(
        results['documents'][0],
        results['metadatas'][0],
        results['distances'][0]
    ), 1):
        similarity = 1 - distance  # Convertir distance en similarité
        print(f"{i}. {metadata['source']} (chunk {metadata['chunk_id']+1}/{metadata['total_chunks']}) - similarité: {similarity:.3f}")
        print(f"   Preview: {doc[:100]}...")
    
    return results

def generate_answer(query: str, search_results):
    """Génère une réponse avec Mistral"""
    
    if not search_results or not search_results['documents'][0]:
        return "Aucun document pertinent trouvé."
    
    # Construire le contexte
    context_parts = []
    for doc, metadata in zip(
        search_results['documents'][0],
        search_results['metadatas'][0]
    ):
        context_parts.append(f"[Source: {metadata['source']}, partie {metadata['chunk_id']+1}]\n{doc}")
    
    context = "\n\n".join(context_parts)
    
    # Prompt avec contexte
    prompt = f"""Tu es un assistant qui répond aux questions en te basant sur des documents fournis.

DOCUMENTS:
{context}

QUESTION: {query}

INSTRUCTIONS:
- Réponds UNIQUEMENT en te basant sur les documents fournis ci-dessus
- Si l'information n'est pas dans les documents, dis-le clairement
- Cite les sources quand c'est pertinent
- Sois précis et concis

RÉPONSE:"""

    # Appel à Mistral
    print("\n💬 Génération de la réponse...")
    response = requests.post(
        f"{OLLAMA_URL}/api/generate",
        json={
            "model": "mistral",
            "prompt": prompt,
            "stream": False,
            "options": {
                "temperature": 0.1,  # Plus déterministe
                "top_p": 0.9
            }
        }
    )
    
    if response.status_code == 200:
        return response.json()['response']
    else:
        return f"Erreur API: {response.status_code}"

def rag_query(query: str, top_k: int = 3):
    """Pipeline RAG complet avec ChromaDB"""
    print("="*60)
    print("RAG avec ChromaDB")
    print("="*60)
    
    # 1. Rechercher documents pertinents
    results = search_documents(query, top_k)
    
    if not results:
        print("\n❌ Aucun résultat")
        return
    
    # 2. Générer réponse
    answer = generate_answer(query, results)
    
    print("\n" + "="*60)
    print("RÉPONSE:")
    print("="*60)
    print(answer)
    print("="*60)
    
    return answer

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python rag_chromadb.py <question>")
        print('Exemple: python rag_chromadb.py "Quelle est la conclusion ?"')
        sys.exit(1)
    
    query = " ".join(sys.argv[1:])
    rag_query(query, top_k=3)
EOF

chmod +x ~/hp-z800-ai-agent/rag/scripts/rag_chromadb.py
```

---

## 🧪 Étape 4 : Test de ChromaDB

### 4.1 Indexer les Documents

```bash
cd ~/hp-z800-ai-agent/rag/scripts

# Indexer tous les documents
python index_documents.py
```

**✅ Résultat attendu :**
```
============================================================
INDEXATION DES DOCUMENTS
============================================================
📚 Nouvelle collection créée

📄 Indexation: test-doc.txt
  📑 2 chunks
  ✅ 2 chunks indexés

============================================================
✅ INDEXATION TERMINÉE
   Fichiers traités: 1
   Chunks indexés: 2
   Total en base: 2
============================================================
```

### 4.2 Tester la Recherche

```bash
# Première question
python rag_chromadb.py "Quelles sont les spécifications du HP Z800 ?"

# Deuxième question
python rag_chromadb.py "Comment nettoyer le Z800 ?"

# Troisième question
python rag_chromadb.py "Quelle est la configuration réseau recommandée ?"
```

**✅ Cette fois, c'est BEAUCOUP plus rapide !** (pas de recalcul des embeddings)

---

## 📊 Étape 5 : Script de Gestion de la Base

```bash
cat > ~/hp-z800-ai-agent/rag/scripts/manage_db.py << 'EOF'
#!/usr/bin/env python3
"""
Gestion de la base ChromaDB
"""

import chromadb
import sys

client = chromadb.PersistentClient(path="../vectordb")

def stats():
    """Affiche les statistiques de la base"""
    try:
        collection = client.get_collection(name="documents")
        count = collection.count()
        
        print("="*60)
        print("STATISTIQUES DE LA BASE VECTORIELLE")
        print("="*60)
        print(f"Total de chunks: {count}")
        
        # Lister les sources
        if count > 0:
            all_data = collection.get(include=["metadatas"])
            sources = set(m['source'] for m in all_data['metadatas'])
            
            print(f"\nDocuments indexés: {len(sources)}")
            for source in sorted(sources):
                chunks = sum(1 for m in all_data['metadatas'] if m['source'] == source)
                print(f"  - {source}: {chunks} chunks")
        
        print("="*60)
    except:
        print("❌ Aucune collection trouvée")

def reset():
    """Réinitialise la base"""
    try:
        client.delete_collection(name="documents")
        print("✅ Base réinitialisée")
    except:
        print("⚠️  Aucune collection à supprimer")

def search_by_source(source: str):
    """Recherche tous les chunks d'une source"""
    try:
        collection = client.get_collection(name="documents")
        results = collection.get(
            where={"source": source},
            include=["documents", "metadatas"]
        )
        
        print(f"\n📄 Chunks de '{source}':")
        for i, (doc, meta) in enumerate(zip(results['documents'], results['metadatas']), 1):
            print(f"\n--- Chunk {i} ---")
            print(doc[:200] + ("..." if len(doc) > 200 else ""))
        
        print(f"\n✅ Total: {len(results['documents'])} chunks")
    except Exception as e:
        print(f"❌ Erreur: {e}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage:")
        print("  python manage_db.py stats              # Statistiques")
        print("  python manage_db.py reset              # Réinitialiser")
        print("  python manage_db.py search <filename>  # Voir chunks d'un fichier")
        sys.exit(1)
    
    cmd = sys.argv[1]
    
    if cmd == "stats":
        stats()
    elif cmd == "reset":
        confirm = input("⚠️  Êtes-vous sûr de vouloir réinitialiser la base ? (oui/non): ")
        if confirm.lower() == "oui":
            reset()
    elif cmd == "search" and len(sys.argv) > 2:
        search_by_source(sys.argv[2])
    else:
        print("❌ Commande inconnue")
EOF

chmod +x ~/hp-z800-ai-agent/rag/scripts/manage_db.py
```

**Utilisation :**
```bash
# Voir les stats
python manage_db.py stats

# Voir les chunks d'un document
python manage_db.py search test-doc.txt

# Réinitialiser la base
python manage_db.py reset
```

---

## 🚀 Étape 6 : Workflow Complet

### 1️⃣ Ajouter de Nouveaux Documents

```bash
# Copier vos documents
cp ~/Documents/rapport.pdf ~/hp-z800-ai-agent/rag/documents/
cp ~/Documents/notes.docx ~/hp-z800-ai-agent/rag/documents/

# Réindexer tout
cd ~/hp-z800-ai-agent/rag/scripts
python index_documents.py
```

### 2️⃣ Interroger la Base

```bash
python rag_chromadb.py "Votre question"
```

### 3️⃣ Vérifier la Base

```bash
python manage_db.py stats
```

---

## 📈 Comparaison : Simple vs ChromaDB

| Critère | RAG Simple | RAG ChromaDB |
|---------|-----------|--------------|
| **Première requête** | ~15 secondes | ~20 secondes (indexation) |
| **Requêtes suivantes** | ~15 secondes | **~3 secondes** ⚡ |
| **10 documents** | ~2 minutes | ~5 secondes ⚡ |
| **100 documents** | ~20 minutes | ~10 secondes ⚡ |
| **Persistance** | ❌ Non | ✅ Oui |
| **Scalabilité** | ❌ Limité | ✅ Excellente |

---

## ✅ Checklist de Validation

- [ ] ChromaDB installé
- [ ] Documents indexés avec succès
- [ ] Base vectorielle créée (`~/hp-z800-ai-agent/rag/vectordb/`)
- [ ] Requêtes plus rapides qu'avant
- [ ] Stats de la base fonctionnelles
- [ ] Ajout de nouveaux documents fonctionne

---

## 🔧 Scripts Utiles

### Réindexer Automatiquement

```bash
cat > ~/hp-z800-ai-agent/rag/scripts/auto_index.sh << 'EOF'
#!/bin/bash
# Surveille le dossier documents et réindexe automatiquement

DOCS_DIR="../documents"

while true; do
    echo "🔄 Vérification de nouveaux documents..."
    python index_documents.py
    echo "✅ Prochaine vérification dans 5 minutes"
    sleep 300
done
EOF

chmod +x ~/hp-z800-ai-agent/rag/scripts/auto_index.sh
```

---

## 📊 Résumé ChromaDB

```
✅ Base vectorielle persistante (ChromaDB)
✅ Indexation automatique des documents
✅ Recherche ultra-rapide (5-10x plus rapide)
✅ Métadonnées enrichies (source, chunk, type)
✅ Gestion facile (stats, reset, search)
✅ Scalable (1000+ documents)

Performance:
- Simple RAG: 15s par requête
- ChromaDB: 3s par requête ⚡
- Économie: ~80% de temps
```

---

## ➡️ Prochaine Étape

**[Étape 7 : Fine-tuning Setup](07-finetuning-setup.md)** *(optionnel)*

Préparer l'environnement pour fine-tuner Mistral sur vos propres données.

---

## 🆘 Dépannage

### Problème : "No module named 'chromadb'"

```bash
conda activate rag
pip install chromadb
```

### Problème : Base corrompue

```bash
python manage_db.py reset
rm -rf ~/hp-z800-ai-agent/rag/vectordb/*
python index_documents.py
```

### Problème : Indexation lente

- Réduisez la taille des chunks
- Indexez par lots
- Vérifiez que la GPU est utilisée

---

**🎉 Votre RAG est maintenant professionnel et performant !**
