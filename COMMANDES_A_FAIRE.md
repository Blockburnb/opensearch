# OpenSearch - Guide complet des commandes à exécuter

---

## PARTIE I - Infrastructure et Questions du cluster

### 1. Vérifier que OpenSearch et Dashboards sont lancés

**OpenSearch (port 9200)** :
```bash
curl.exe -X GET http://localhost:9200 -u "admin:admin" --insecure
```

**OpenSearch Dashboards (port 5601)** :
Accéder à : `http://localhost:5601`

---

### 2. Questions sur l'état du cluster (Python avec opensearch-py)

```python
from opensearchpy import OpenSearch

client = OpenSearch(
    hosts=[{"host": "localhost", "port": 9200}],
    http_auth=("admin", "admin"),
    use_ssl=False,
    verify_certs=False,
    ssl_show_warn=False
)

# --- QUESTION 1 : Health du cluster ---
response = client.cluster.health()
print("Health du cluster:", response)

# --- QUESTION 2 : Nombre de nœuds ---
response = client.cluster.state()
print("Nombre de nœuds:", len(response['state']['nodes']))

# --- QUESTION 3 : Adresses IP des nœuds ---
response = client.nodes.info()
for node_id, node_info in response['nodes'].items():
    print(f"Node: {node_id}, IP: {node_info['http']['publish_address']}")

# --- QUESTION 4 : Créer un index avec 2 réplicas ---
index_config = {
    "settings": {
        "number_of_replicas": 2
    }
}
client.indices.create(index="mon_index", body=index_config)

# Vérifier l'index
info = client.indices.get(index="mon_index")
print("Index mon_index créé :", info)
```

---

## TP01 - CRUD

### Via OpenSearch Dev Tools (Dev Tools dans le Dashboard)

#### A. API de base

**Question 1 : Health du cluster**
```
GET _cluster/health
```

**Question 2 : Nombre de nœuds**
```
GET _cluster/state
```

**Question 3 : Statistiques des nœuds**
```
GET _nodes/stats
```

**Question 4 : Adresses IP des nœuds et nombre d'index**
```
GET _cat/nodes?v
GET _cat/indices
```

---

### B. Opérations CRUD

#### CREATE - Insertion

**Créer l'index `myindex`** :
```
PUT /myindex
```

**Insérer un document avec ID (PUT)** :
```
PUT /myindex/_doc/1
{
    "title": "Mon premier document",
    "text":  "C'est un essai...",
    "date":  "2023/06/01"
}
```
*Question : Décrire le contenu de la réponse*

**Insérer un document avec ID auto-généré (POST)** :
```
POST /myindex/_doc
{
    "title": "Mon second document",
    "text":  "C'est un autre essai...",
    "date":  "2023/06/02"
}
```

**Vérifier les documents** :
```
GET /myindex/_search
{
  "query": {
    "match_all": {}
  }
}
```
*Question : Décrire l'ID du second document*

---

#### READ - Lecture

**Récupérer le document ID 1** :
```
GET /myindex/_doc/1
```

---

#### UPDATE - Mise à jour

**Mettre à jour le document ID 1** :
```
POST /myindex/_update/1
{
    "doc": {
      "comment": "ok je mets à jour",
      "description": "je modifie autant que je veux",
      "text" : "meme les champs existants"
    }
}
```
*Question : Décrire ce qu'OpenSearch a fait (version incrémentée, etc.)*

---

#### DELETE - Suppression

**Supprimer le document ID 1** :
```
DELETE /myindex/_doc/1
```
*Question : Décrire la réponse*

---

### C. Bulk API

**Exécuter les opérations bulk** :
```
POST /_bulk
{ "delete": { "_index": "myindex", "_id": "1234" }}
{ "create": { "_index": "myindex", "_id": "123" }}
{ "title":    "Mon premier doc 123" }
{ "index":  { "_index": "myindex","_id": "1234" }}
{ "title":    "Mon second doc 1234" }
{ "update": { "_index": "myindex", "_id": "123"} }
{ "doc" : {"title" : "le titre de mon doc 123"} }
{ "create": { "_index": "myindex", "_id": "1234" }}
{ "title":    "My new first blog post " }
```
*Question : Décrire les succès et échecs*

---

### D. Exercice pratique : Travel

**1. Créer l'index `travel`** :
```
PUT /travel
```

**2. Insérer 2 destinations avec PUT** :
```
PUT /travel/_doc/1
{
    "nom": "Paris",
    "pays": "France"
}

PUT /travel/_doc/2
{
    "nom": "Kyoto",
    "pays": "Japon"
}
```

**3. Lire les documents** :
```
GET /travel/_doc/1
GET /travel/_doc/2
```

**4. Mettre à jour un document (ajouter description et prix)** :
```
POST /travel/_update/1
{
    "doc": {
      "description": "La ville lumière",
      "prix": 1500
    }
}
```

**5. Vérifier la mise à jour** :
```
GET /travel/_doc/1
```

**6. Supprimer un document** :
```
DELETE /travel/_doc/2
```

**7. Lister tous les documents** :
```
GET /travel/_search
{
  "query": {
    "match_all": {}
  }
}
```

**8. Bonus - Refaire tout avec Bulk** :
```
POST /_bulk
{ "create": { "_index": "travel", "_id": "1" }}
{ "nom": "Paris", "pays": "France", "description": "La ville lumière", "prix": 1500 }
{ "create": { "_index": "travel", "_id": "2" }}
{ "nom": "Kyoto", "pays": "Japon", "description": "Temple historique", "prix": 2000 }
```

---

## TP02 - Structured Search

### Préparation : Charger les données movies

**Sur Windows PowerShell** :
```powershell
cd e:\iut\opensearch
curl.exe -H "Content-Type: application/x-ndjson" -X PUT "https://localhost:9200/_bulk" -ku admin:admin --data-binary "@data/movies.json"
```

**Vérifier le chargement** :
```
GET /movies/_doc/1
```

---

### Questions - Recherches structurées

**1. Rechercher le film avec le titre `Spider-Man`** :
```
GET /movies/_search
{
  "query": {
    "match": {
      "fields.title": "Spider-Man"
    }
  }
}
```

**2. Rechercher les films réalisés entre le 01/05/1977 et le 31/05/1977** :
```
GET /movies/_search
{
  "query": {
    "range": {
      "fields.release_date": {
        "gte": "1977-05-01",
        "lte": "1977-05-31"
      }
    }
  }
}
```

**3. Rechercher les acteurs et directeurs du film `Spider-Man` (afficher uniquement ces champs)** :
```
GET /movies/_search
{
  "_source": ["fields.actors", "fields.directors"],
  "query": {
    "match": {
      "fields.title": "Spider-Man"
    }
  }
}
```

**4. Trier par ordre décroissant de la clé `release_date`** :
```
GET /movies/_search
{
  "sort": [
    { "fields.release_date": "desc" }
  ],
  "query": {
    "match_all": {}
  }
}
```

**5. Rechercher les films sans la clé `rating` et afficher uniquement le titre** :
```
GET /movies/_search
{
  "_source": ["fields.title"],
  "query": {
    "bool": {
      "must_not": [
        { "exists": { "field": "fields.rating" } }
      ]
    }
  }
}
```

**6. Top 10 des films avec un `rank` > 8 (trié par rank asc, puis year desc)** :
```
GET /movies/_search
{
  "sort": [
    { "fields.rank": "asc" },
    { "fields.year": "desc" }
  ],
  "query": {
    "bool": {
      "must": [
        { "range": { "fields.rank": { "lte": 10 } } },
        { "range": { "fields.rank": { "gt": 8 } } }
      ]
    }
  }
}
```

---

## TP03 - Full Text Search

### Préparation : Vérifier les données movies

```
GET /movies/_doc/1
```

---

### Questions - Recherches textuelles

**1. Exécuter et commenter (recherche simple Star Wars)** :
```
GET /movies/_search
{
  "_source": "fields.title",
  "query": {
    "match": {
      "fields.title": "Star Wars"
    }
  }
}
```
*Commentaire : Observe les résultats trouvés*

**2. Exécuter et commenter (avec operator AND)** :
```
GET /movies/_search
{
  "_source": "fields.title",
  "query": {
    "match": {
      "fields.title": {
        "query": "Star Wars",
        "operator": "and"
      }
    }
  }
}
```
*Commentaire : Compare avec la requête précédente*

**3. Rechercher les films "Star Wars" avec George Lucas comme réalisateur (match)** :
```
GET /movies/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "fields.title": "Star Wars" } },
        { "match": { "fields.directors": "George Lucas" } }
      ]
    }
  }
}
```

**4. Rechercher les films avec Harrison Ford contenant "Jones" dans le plot** :
```
GET /movies/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "fields.actors": "Harrison Ford" } },
        { "match": { "fields.plot": "Jones" } }
      ]
    }
  }
}
```

**5. Rechercher "Jersey" dans le titre ou le résumé (multi_match)** :
```
GET /movies/_search
{
  "query": {
    "multi_match": {
      "query": "Jersey",
      "fields": ["fields.title", "fields.plot"]
    }
  }
}
```

**6. Rechercher les films avec Harrison Ford et invasion extra-terrestre** :
```
GET /movies/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "fields.actors": "Harrison Ford" } },
        { "match": { "fields.plot": "invasion extra-terrestre" } }
      ]
    }
  }
}
```

---

## Résumé des sections à compléter

| TP | Question | Statut |
|----|----------|--------|
| Partie I | Health du cluster | À faire |
| Partie I | Nombre de nœuds | À faire |
| Partie I | IP des nœuds | À faire |
| Partie I | Créer mon_index avec 2 réplicas | À faire |
| TP01 | Questions cluster (dev tools) | À faire |
| TP01 | CRUD complet | À faire |
| TP01 | Exercice travel | À faire |
| TP02 | Requêtes structurées (6 requêtes) | À faire |
| TP03 | Requêtes full text (6 requêtes) | À faire |

