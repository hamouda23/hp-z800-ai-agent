# Dépannage RAG

## Erreur 500 lors de l'Indexation

### Symptôme
```
Exception: Erreur API: 500
```

### Cause
Chunks trop gros pour nomic-embed-text (limite ~512 tokens)

### Solution
Réduire chunk_size à 180 mots avec overlap de 60:

\```python
# Dans extract_text.py
chunk_size=180, overlap=60
\```

Ajouter sécurité dans generate_embeddings.py:
\```python
if len(words) > 250:
    text = " ".join(words[:250])
\```

### Résultat
✅ Indexation stable même avec gros PDFs (500+ pages)
