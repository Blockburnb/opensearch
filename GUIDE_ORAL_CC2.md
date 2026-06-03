# CC2 - Préparation Orale OpenSearch/Elasticsearch

---

## 📋 Structure de la présentation (7 minutes)

### **1. Introduction et définition (1 min)**

**À dire** :
"Bonjour, je vais vous parler d'Elasticsearch et OpenSearch, deux outils essentiels dans le Big Data.

Imaginez que vous avez des milliards de fichiers textes mélangés dans une énorme pièce. Si je vous demande de trouver tous les fichiers qui contiennent le mot 'erreur', vous allez devoir les lire tous un par un. C'est très lent.

Elasticsearch/OpenSearch fonctionne comme un bibliothécaire très intelligent. Cet outil indexe les données - c'est-à-dire qu'il les organise de manière intelligente - pour qu'on puisse trouver les informations en quelques millisecondes au lieu de secondes ou minutes.

Plus précisément :
- C'est un **moteur de recherche** : il cherche du texte très rapidement
- C'est un outil d'**analyse** : il peut faire des statistiques sur les données
- C'est **distribué** : les données sont réparties sur plusieurs serveurs, ce qui permet de gérer des volumes énormes

Elasticsearch a été créé par Elastic. OpenSearch est un fork (une copie modifiée) créé par Amazon pour rester open-source.

Cet outil est basé sur Lucene, une bibliothèque Java qui fait du full-text search depuis 2000."

### **2. Architecture (1.5 min)**

**À dire** :
"Maintenant, regardons comment Elasticsearch/OpenSearch est organisé. Imaginez une chaîne d'hôtels :

1. **Cluster** : C'est la chaîne d'hôtel complète. Tous les serveurs travaillent ensemble comme une seule entité.

2. **Nœuds** : Ce sont les serveurs physiques. Si j'ai 3 nœuds, j'ai 3 serveurs qui travaillent ensemble. Si un serveur tombe en panne, les autres continuent à fonctionner.

3. **Index** : C'est comme une base de données. Vous pouvez avoir un index 'logs' pour les journaux d'erreurs, un index 'movies' pour les films, etc.

4. **Document** : C'est un enregistrement. Par exemple, un log unique ou un film unique. C'est du JSON.

5. **Shards** : C'est la partie importante pour la scalabilité. Imaginez que vous avez 1 million de films. Vous pouvez les diviser en 5 shards (partitions) :
   - Shard 0 : films 1-200k
   - Shard 1 : films 200k-400k
   - Shard 2 : films 400k-600k
   - Shard 3 : films 600k-800k
   - Shard 4 : films 800k-1M
   
   Au lieu de chercher dans 1 million de films sur 1 serveur, je cherche en parallèle dans 5 groupes de 200k films sur 5 serveurs différents. C'est 5x plus rapide !

6. **Replicas** : C'est une copie de sécurité. Si j'ai 2 replicas, chaque shard est copié 2 fois sur d'autres nœuds. Si un serveur explose, pas de problème, les données existent ailleurs."

**Diagramme à dessiner** :
```
┌─────────────────────── CLUSTER OPENSEARCH ──────────────┐
│                                                           │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐       │
│  │ Node 1   │      │ Node 2   │      │ Node 3   │       │
│  │          │      │          │      │          │       │
│  │ Shard 0  │      │ Shard 1  │      │ Replica 0│       │
│  │ Replica1 │      │ Replica1 │      │ Replica 1│       │
│  └──────────┘      └──────────┘      └──────────┘       │
│                                                           │
└───────────────────────────────────────────────────────────┘
         INDEX: "movies" (5 shards, 2 replicas)
```

### **3. Place dans l'écosystème Big Data (1 min)**

**À dire** :
"Pour comprendre où Elasticsearch se place, je vais vous montrer un pipeline typique d'une grande entreprise qui veut analyser ses logs.

**Étape 1 - Les sources** :
Une application génère 1000 logs par seconde. Votre API, votre base de données, vos serveurs Web... chacun produit des informations.

**Étape 2 - La collecte** :
Vous ne pouvez pas envoyer les millions de logs directement à Elasticsearch. Vous avez besoin d'un collecteur. C'est à ça que servent des outils comme :
- **Logstash** : collecte les logs sur plusieurs serveurs et les envoie à Elasticsearch
- **Fluentd** : même chose, mais plus léger, écrit en C et Ruby
- **Filebeat** : collecte spécialisée pour les fichiers texte

Imaginez que ce sont des agents qui tournent sur chaque serveur et qui disent 'hey, il y a un nouveau log, je l'envoie à Elasticsearch'.

**Étape 3 - L'indexation et le stockage** :
Elasticsearch reçoit les logs et les indexe. C'est-à-dire qu'il les organise de manière à pouvoir les chercher très rapidement.

**Étape 4 - La visualisation** :
C'est un problème : Elasticsearch retourne du JSON, ce n'est pas très visuel. D'où l'existence de :
- **Kibana** : interface web faite par Elastic spécialisée pour visualiser les données Elasticsearch
- **OpenSearch Dashboards** : c'est la version open-source de Kibana

Vous créez des graphiques, des tableaux, des cartes pour voir vos données.

**Étape 5 - Les alertes** :
Si vos serveurs commencent à avoir 100% d'erreurs, vous voulez être alerté ! Elasticsearch peut déclencher des alertes en temps réel.

**Résumé du pipeline** :
```
Serveurs (logs)
    ↓
Collecteur (Logstash/Fluentd)
    ↓
Elasticsearch (Indexation)
    ↓
Kibana/Dashboards (Visualisation)
    ↓
Alertes et rapports
```"

### **4. Positionnement NoSQL (1 min)**

**À dire** :
"Vous vous demandez peut-être : 'Pourquoi ne pas juste utiliser SQL et une base de données relationnelle ?' Bonne question !

**Elasticsearch vs SQL** :
Avec SQL, vous écrivez :
```sql
SELECT * FROM logs WHERE level = 'ERROR'
```
Ça marche, c'est rapide pour les données structurées. MAIS si vous voulez chercher : 'tous les logs qui contiennent le mot connection ET mention une erreur de base de données', c'est compliqué. SQL peut le faire mais c'est pas son point fort.

Elasticsearch est conçu pour ça. C'est du full-text search.

**Elasticsearch vs MongoDB** :
MongoDB est une base de données NoSQL généraliste. Elle gère bien l'insertion et la modification des données. 
Elasticsearch, lui, est optimisé pour la RECHERCHE. Il est un peu plus lent pour insérer, mais ultra-rapide pour chercher.

**Elasticsearch vs Cassandra** :
Cassandra est une base de données distribuée très performante pour les données time-series (métriques, capteurs).
Elasticsearch peut aussi gérer les time-series MAIS son point fort, c'est la recherche et les agrégations.

**Elasticsearch vs Redis** :
Redis stocke les données EN MÉMOIRE. Super rapide mais si vous arrêtez Redis, vous perdez tout (sauf si vous activez la persistance).
Elasticsearch stocke les données sur DISQUE. Beaucoup plus lent mais persistant, et vous pouvez avoir des milliards de documents.

**En résumé** :
| Outil | Cas d'usage | Point fort | Point faible |
|-------|-----------|-----------|-------------|
| SQL (PostgreSQL) | Données structurées, transactions | Cohérence, transactions | Full-text search |
| MongoDB | CRUD généraliste | Flexibilité du schéma | Pas optimisé pour search |
| Cassandra | Métriques, high-cardinality | Scalabilité, performance write | Complexité |
| Redis | Cache, sessions | Super rapide (RAM) | Pas persistant |
| **Elasticsearch** | **Logs, Search, Analytics** | **Full-text search, Agrégations** | **Ressources RAM, pas transactionnel** |"

### **5. Démonstration pratique - Introduction (30 secondes)**

**À dire** :
"Maintenant qu'on a compris la théorie, voyons comment ça marche en pratique.

Je vais vous montrer un système de log : imaginons une application avec plusieurs services (authentification, paiement, base de données). Chaque service génère des logs : '10:05am - Service AUTH - User login successful'.

Je vais :
1. Importer un dataset de logs
2. Faire 2 recherches structurées (avec des filtres précis)
3. Faire 2 recherches full-text (chercher du texte libre)
4. Vous montrer comment ça s'affiche graphiquement

Voyons voir..."

---

## 🚀 Démonstration pratique

### **Étape 1 : Préparer les données de logs**

**Créer un index pour les logs** (Dev Tools) :
```
PUT /logs-demo
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

**Insérer dei données de logs v1** :
```
POST /logs-demo/_bulk
{ "create": { "_index": "logs-demo", "_id": "1" } }
{ "timestamp": "2024-06-01T10:00:00Z", "level": "ERROR", "service": "api-gateway", "message": "Connection timeout to database", "user": "john_doe" }
{ "create": { "_index": "logs-demo", "_id": "2" } }
{ "timestamp": "2024-06-01T10:05:00Z", "level": "INFO", "service": "auth-service", "message": "User login successful", "user": "jane_smith" }
{ "create": { "_index": "logs-demo", "_id": "3" } }
{ "timestamp": "2024-06-01T10:10:00Z", "level": "ERROR", "service": "payment-service", "message": "Payment processing failed", "user": "bob_jones" }
{ "create": { "_index": "logs-demo", "_id": "4" } }
{ "timestamp": "2024-06-01T10:15:00Z", "level": "WARNING", "service": "api-gateway", "message": "High memory usage detected", "user": "admin" }
{ "create": { "_index": "logs-demo", "_id": "5" } }
{ "timestamp": "2024-06-01T10:20:00Z", "level": "INFO", "service": "database", "message": "Backup completed successfully", "user": "system" }
{ "create": { "_index": "logs-demo", "_id": "6" } }
{ "timestamp": "2024-06-01T10:25:00Z", "level": "ERROR", "service": "auth-service", "message": "Invalid credentials provided", "user": "hacker_attempt" }
{ "create": { "_index": "logs-demo", "_id": "7" } }
{ "timestamp": "2024-06-01T10:30:00Z", "level": "INFO", "service": "api-gateway", "message": "API request processed", "user": "jane_smith" }
{ "create": { "_index": "logs-demo", "_id": "8" } }
{ "timestamp": "2024-06-01T10:35:00Z", "level": "ERROR", "service": "payment-service", "message": "Connection refused to payment gateway", "user": "john_doe" }
```

**Insérer des données de logs v2** :
```
POST demo_logs/_doc
{
  "date": "2025-06-01T10:00:00",
  "niveau": "INFO",
  "service": "WEB",
  "message": "Utilisateur connecté"
}
```

**Vérifier les données** :
```
GET /logs-demo/_search
{
  "query": {
    "match_all": {}
  }
}
```

---

### **Étape 2 : Deux requêtes STRUCTURÉES**

**À dire avant la requête 1** :
"Maintenant je vais faire une recherche STRUCTURÉE. Ça veut dire que je vais chercher des valeurs exactes avec des filtres précis, comme 'SELECT WHERE' en SQL.

Je veux trouver : tous les ERREURS du service API-GATEWAY.

Voici ma requête :"

**Requête Structurée 1 : Tous les ERROR du service api-gateway**
```
GET /logs-demo/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "ERROR" } },
        { "match": { "service": "api-gateway" } }
      ]
    }
  }
}
```

**À dire après la requête 1** :
"Regardez la réponse : j'ai 2 erreurs du service API-GATEWAY. Chaque résultat montre le timestamp, le message, le service. C'est très structuré, très similaire à une requête SQL.

Notez que dans nos logs, il n'y a que 8 enregistrements. Imaginez maintenant 100 millions de logs... en un milliseconde, je trouve les 2 qui correspondent à mes critères. C'est la puissance de Elasticsearch."

---

**À dire avant la requête 2** :
"Maintenant, deuxième requête structurée. Je veux trouver tous les logs entre 10h00 et 10h30 du matin.

En SQL, on dirait : SELECT * FROM logs WHERE timestamp BETWEEN '10:00' AND '10:30'"

**Requête Structurée 2 : Logs des 30 dernières minutes dans une plage horaire**
```
GET /logs-demo/_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "2024-06-01T10:00:00Z",
        "lte": "2024-06-01T10:30:00Z"
      }
    }
  }
}
```

**À dire après la requête 2** :
"Cette fois j'ai utilisé une 'range query' - c'est-à-dire une requête par plage. Vous voyez que j'utilise 'gte' (greater than or equal - plus grand que) et 'lte' (less than or equal - plus petit que).

Résultat : 8 logs trouvés. C'est facile pour une machine, ça aurait pu être 100 millions de logs et ça aurait pris quelques millisecondes quand même."

---

### **Étape 3 : Deux requêtes FULL-TEXT**

**À dire avant la requête 1** :
"Maintenant, les recherches FULL-TEXT. C'est là où Elasticsearch devient vraiment puissant.

Avec SQL et une base de données normale, si je veux chercher tous les logs qui contiennent 'database' OU 'connection', c'est hyper compliqué. Avec Elasticsearch, c'est trivial.

Je fais une recherche : cherche dans la colonne 'message' tous les logs contenant 'database' ou 'connection' :"

**Requête Full-Text 1 : Chercher les logs mentionnant "database" ou "connection"**
```
GET /logs-demo/_search
{
  "query": {
    "match": {
      "message": {
        "query": "database connection",
        "operator": "or"
      }
    }
  }
}
```

**À dire après la requête 1** :
"Regardez les résultats ! J'ai 3 logs trouvés :
- 'Backup completed successfully' - contient 'database'
- 'Connection timeout to database' - contient 'database' et 'connection'
- 'Connection refused to payment gateway' - contient 'connection'

Notez que Elasticsearch est intelligent. Il comprend que 'connection' et 'connections' c'est la même chose (stemming). Et il classe les résultats par score de pertinence - le log qui contient BOTH database ET connection est en premier.

C'est du full-text search : chercher du texte libre, pas des valeurs exactes."

---

**À dire avant la requête 2** :
"Dernière requête. Je veux chercher le mot 'payment' mais dans DEUX champs en même temps : à la fois dans le message ET dans le service.

C'est un 'multi_match' :"

**Requête Full-Text 2 : Multi-match entre le message et le service**
```
GET /logs-demo/_search
{
  "query": {
    "multi_match": {
      "query": "payment",
      "fields": ["message", "service"]
    }
  }
}
```

**À dire après la requête 2** :
"Voilà ! 2 logs trouvés qui contiennent 'payment' :
- Un dans le service 'payment-service'
- Un dans le message 'Payment processing failed'

Avec SQL, pour faire ça, vous auriez besoin de : SELECT * FROM logs WHERE message LIKE '%payment%' OR service LIKE '%payment%'

C'est plus verbose, moins élégant. Et sur 100 millions de rows, ça serait beaucoup plus lent sans les bons indexes."

---

### **Étape 4 : Visualisation graphique**

**À dire** :
"Jusqu'à présent, on a vu du JSON. C'est pas très visuel. C'est pour ça qu'on utilise OpenSearch Dashboards.

Dashboards c'est une interface web qui se connecte à Elasticsearch et qui transforme les données en graphiques, tableaux, cartes...

Voici 2 visualisations qu'on peut créer :"

**Dans OpenSearch Dashboards** :

1. **Aller à : Analytics → Discover**
2. **Sélectionner l'index** : `logs-demo`
3. **Créer des visualisations** :

#### **Visualisation 1 : Graphique timeline des logs par niveau**

**À dire** :
"Cette visualisation montre l'évolution du nombre de logs par niveau (ERROR, INFO, WARNING) dans le temps.
- L'axe horizontal = temps (timestamp)
- L'axe vertical = nombre de logs
- Les couleurs = niveau de sévérité (rouge pour ERROR, bleu pour INFO, jaune pour WARNING)

Intérêt : vous pouvez voir immédiatement si à un moment donné, il y a eu une explosion d'erreurs. Par exemple, si entre 10h et 10h30 il y a soudainement 1000 erreurs au lieu de 2, ça veut dire qu'il y a un problème."

- Aller à **Visualize**
- Créer une nouvelle visualisation
- Type : **Area Chart** ou **Bar Chart**
- X-axis : `timestamp` / Date Histogram
- Y-axis : Count
- Color : `level` (pour différencier ERROR, INFO, WARNING)

#### **Visualisation 2 : Pie chart - Répartition des services**

**À dire** :
"Cette visualisation montre quelle proportion de logs vient de quel service.
- Si le service 'payment-service' produit beaucoup plus de logs que les autres, c'est peut-être qu'il y a un problème
- Ou au contraire, c'est normal s'il gère beaucoup plus de requêtes

On peut immédiatement identifier les services 'bavards' (qui font beaucoup de logs)."

- Type : **Pie Chart**
- Segments : `service` / Terms (pour regrouper par service unique)

**À dire en montrant les visualisations** :
"Voilà ! En quelques secondes, un responsable DevOps peut voir l'état de santé de son système.
- D'où viennent les erreurs ?
- Quel service produit le plus de logs ?
- Y a-t-il eu une augmentation soudaine d'erreurs ?

Sans Elasticsearch et Dashboards, il faudrait lire des millions de logs texte. Impossible."

---

## 📊 Plan de présentation PDF

**Voici la structure pour la présentation PDF (utiliser Google Slides, PowerPoint, Canva, ou LibreOffice)** :

### **Slide 1 : Titre**
- Titre : "Elasticsearch/OpenSearch : Moteur de recherche distribué"
- Sous-titre : "CC2 - Présentation Orale"
- Date, nom, cursus

### **Slide 2 : Définition**

**Point clé : "Elasticsearch c'est Google mais pour vos données"**

- Qu'est-ce qu'Elasticsearch ?
  - C'est un moteur de recherche (comme Google cherche sur Internet, Elasticsearch cherche dans vos données)
  - C'est distribué (les données sont sur plusieurs serveurs)
  - C'est temps réel (les requêtes répondent en millisecondes)
  
- Histoire :
  - Créé en 2010 par Shay Banon
  - Basé sur Apache Lucene (une librairie Java pour faire du full-text search depuis l'an 2000)
  
- Les noms :
  - **Elasticsearch** : créé et maintenu par la société Elastic (devenu propriétaire avec la version 8)
  - **OpenSearch** : fork open-source créé par AWS en 2021 pour rester libre
  - Fonctionnellement très similaires

- Ce qu'il fait :
  - Indexation rapide de millions/milliards de documents
  - Recherche super rapide (en millisecondes)
  - Agrégations et statistiques
  - Temps-réel

### **Slide 3 : Architecture**

**Les composants clés** :

- **Cluster** : Ensemble de serveurs qui travaillent ensemble comme une seule entité
- **Node** : Un serveur individuel dans le cluster
- **Index** : Une base de données (collection de documents) - par exemple "logs" ou "movies"
- **Document** : Un enregistrement (au format JSON)
- **Shard** : Une partition d'un index
  - Permet le parallélisme : chercher dans 5 shards en parallèle au lieu d'1 seul = 5x plus rapide
  - Permet la scalabilité : si 1 serveur peut gérer 100GB, 5 serveurs en shard peuvent en gérer 500GB
  
- **Replica** : Une copie d'un shard
  - Redondance : si un serveur meurt, les données existent sur un autre
  - Amélioration des lectures : les requêtes peuvent être distribuées aux réplicas

**Exemple concret** :
Et que j'ai 1 billion de documents movies et un shard unique, il faudrait les chercher tous un par un.
Avec 5 shards, je divise par 5. Avec 2 replicas, j'ai aussi la redondance.

### **Slide 4 : Écosystème Big Data (ELK/OpenSearch Stack)**

**Pourquoi un stack complet ?**
OpenSearch seul, c'est juste du stockage et de la recherche. C'est pas très utile si vous ne savez pas comment envoyer les données et les visualiser.

**Les composants du stack** :

1. **Sources**
   - Applications, serveurs, bases de données
   - Milliers de logs générés chaque seconde

2. **Collecteurs**
   - **Logstash** : collecte, parse (interprète), enrichit les logs
   - **Fluentd** : plus léger, fait la même chose
   - **Filebeat** : collecte spécialisée pour les fichiers
   - Rôle : transformer "2024-06-01 10:05:00 ERROR timeout" en JSON structuré

3. **Elasticsearch/OpenSearch**
   - Stockage et indexation
   - Full-text search
   - Agrégations

4. **Kibana/OpenSearch Dashboards**
   - Interface web de visualisation
   - Créer des graphiques, dashboards
   - Créer des alertes

5. **Autres outils possibles**
   - **Beats** (Metricbeat, Filebeat, etc.) : collecteurs spécialisés
   - **X-Pack / Security** : gestion des droits d'accès
   - **Watcher/Alerting** : déclencher des actions quand quelque chose se passe

**Comparaisons avec alternatives** :
- **Splunk** : LE leader du marché mais très cher (10x plus cher que ELK)
- **Datadog** : SaaS cloud, plus cher mais plus facile
- **Sumo Logic** : alternative SaaS
- **New Relic** : APM (monitoring applicatif)

### **Slide 5 : Cas d'usage**

**1. Logs et troubleshooting**
   - "Mon application a crashé, pourquoi ?"
   - 100 Go de logs en 1 jour
   - Chercher "OutOfMemory" en 200 microsecondes

**2. Observabilité et monitoring**
   - APM (Application Performance Monitoring)
   - CPU, mémoire, disque en temps réel
   - "À quelle heure mon CPU a dépassé 90% ?"

**3. Search analytics**
   - Analyser les recherches que font les utilisateurs
   - Comprendre ce qu'ils cherchent
   - Google Analytics sur vos données

**4. Security et SOC**
   - Détecter les intrusions
   - "Quelqu'un a tenté 10000 connexions en 1 minute, alerte !"

**5. E-commerce et product search**
   - Chercher des produits (comme sur Amazon)
   - Suggestions "Avez-vous pensé à..." grâce à Elasticsearch

**6. Infrastructure as Code**
   - Logs Kubernetes
   - Logs AWS CloudTrail
   - Auditer qui a fait quoi

### **Slide 6 : Position NoSQL**

**Qu'est-ce que NoSQL ?**
Une base de données qui n'est pas relationnelle (pas de tablets, lignes, colonnes SQL classiques).
Plutôt des documents JSON, ou key-value pairs, ou graphes, etc.

**Comparaison détaillée** :

| Type | Exemple | Cas d'usage | vs Elasticsearch |
|------|---------|------------|-----------------|
| **Document** | MongoDB | CRUD généraliste | MongoDB : bon pour insérer/modifier. ES : bon pour chercher. |
| **Key-Value** | Redis | Cache ultra-rapide | Redis : en mémoire (RAM), très rapide mais pas persistant. ES : sur disque, persistant. |
| **Time-Series** | InfluxDB, Prometheus | Métriques de capteurs | InfluxDB : spécialisé pour 1 point par seconde. ES : aussi bon mais moins spécialisé. |
| **Graph** | Neo4j | Relations complexes | Neo4j : pour les graphs. ES : pas conçu pour les relations. |
| **Column-Store** | ClickHouse | Agrégations énormes | ClickHouse : meilleures agrégations. ES : meilleures recherches. |

**En résumé** :
Elasticsearch n'est PAS le mieux pour tout. Mais il est le meilleur pour le full-text search et très bon pour les logs + analytics.

### **Slide 7 : Démo - Import de logs**

**À montrer** :
- Commande d'import : `POST /_bulk` avec les 8 documents de logs
- Le JSON structuré d'un log (timestamp, level, service, message, user)
- Le résultat : "Inserted 8 documents"

**À dire** :
"On a 8 logs. En production, c'est 100 millions de logs par jour. Elasticsearch les indexe tous et les rend searchable en quelques secondes."

### **Slide 8 : Démo - Requête structurée 1**

**Question** : "Tous les ERREURS du service API-GATEWAY"

**À dire** :
"Je combine 2 filtres avec AND :
- field 'level' = 'ERROR'
- AND field 'service' = 'api-gateway'

Résultat : 2 documents trouvés de type {timestamp, level, service, message}

C'est rapide et exact. C'est du filtering structuré."

### **Slide 9 : Démo - Requête structurée 2**

**Question** : "Logs entre 10:00 et 10:30 le 01/06/2024"

**À dire** :
"Je fais une requête par plage (range query) :
- timestamp >= 2024-06-01T10:00:00Z
- AND timestamp <= 2024-06-01T10:30:00Z

Résultat : 8 documents trouvés (tous les logs de cette tranche horaire)

En SQL ça serait : WHERE timestamp BETWEEN '10:00' AND '10:30'"

### **Slide 10 : Démo - Requête full-text 1**

**Question** : "Logs mentionnant 'database' OU 'connection' dans le message"

**À dire** :
"C'est du full-text search. Je cherche du texte libre, pas des valeurs exactes.

Le moteur cherche tous les documents où le field 'message' contient une de ces deux mots.

Résultat : 3 documents trouvés
- 'Backup completed successfully' (contient 'database')
- 'Connection timeout to database' (contient les deux)
- 'Connection refused to payment gateway' (contient 'connection')

Note importante : Elasticsearch est intelligent. Il comprend que :
- 'connect', 'connection', 'connecting' c'est la même notion (stemming)
- Il classe par pertinence (le log avec les 2 mots est en premier)"

### **Slide 11 : Démo - Requête full-text 2**

**Question** : "Logs contenant 'payment' dans le message OU dans le service"

**Type** : Multi-match query

**À dire** :
"Au lieu de chercher dans un seul field, je cherche dans PLUSIEURS fields en même temps.

C'est super utile quand on ne sait pas où sera l'information.

Résultat : 2 documents trouvés
- Logs du service 'payment-service'
- Logs avec 'Payment processing failed' dans le message

En SQL normal ça serait beaucoup plus compliqué. Avec Elasticsearch, c'est une ligne de code JSON."

### **Slide 12 : Visualisations graphiques**

**Graphique 1 : Timeline des logs par niveau**
```
Y = Nombre de logs
X = Temps
Couleurs = Niveau (ERROR, INFO, WARNING)
```
**Utilité** : Détecter immédiatement les pics d'erreurs
- "À 10h15 il y a eu 1000 erreurs" = problème !
- Pas besoin de lire 1000 logs texte

**Graphique 2 : Pie chart - Quels services génèrent les logs ?**
```
Slice = Service
Size = Nombre de logs
```
**Utilité** : 
- Identifier les services "bavards"
- Peut indiquer un problème (service qui log trop)
- Ou juste qu'il gère beaucoup plus de trafic

**Avantage clé** :
Sans visualisations, impossible de comprendre les patterns.
Avec visualisations, un DevOps peut monitorer 1000 services en glissant les yeux sur 2-3 graphiques.

### **Slide 13 : Avantages**

**Performance**
- ✅ Full-text search : trouvez "erreur de connexion" parmi 100 milliards de logs en <100ms
- ✅ Parallélisme : shards permettent de chercher en parallèle sur 5-10 serveurs

**Scalabilité**
- ✅ Horizontale : besoin de plus de capacité ? Ajoutez un serveur, les données se rééquilibrent
- ✅ Verticale : un serveur peut gérer des terabytes de données

**Flexibilité**
- ✅ Schéma dynamique : pas besoin de déclarer les colonnes comme en SQL
- ✅ Requêtes complexes : agrégations, filtres multiples, fuzzy matching

**Temps réel**
- ✅ Indexation : les données sont searchable en <1 seconde
- ✅ Resultats : les réponses arrivent en millisecondes

**Écosystème**
- ✅ Kibana/Dashboards : visualisation intégrée
- ✅ Alerting : déclencher des actions (SMS, email, webhook)

### **Slide 14 : Limitations**

**RAM intensif**
- ⚠️ Elasticsearch aime la RAM
- Pour 1 terabyte de données : il y a ~100-200GB en mémoire (cache, indexes, etc.)
- Coûteux en infrastructure

**Pas transactionnel**
- ⚠️ SQL : INSERT A, INSERT B, COMMIT = tout ou rien
- Elasticsearch : INSERT A réussit, INSERT B échoue = données incohérentes
- Solution : gérer manuellement la cohérence ou accepter le "finally consistent"

**Pas conçu pour les relations**
- ⚠️ SQL : JOIN sur 3 tables = facile
- Elasticsearch : fait pas les JOINs
- Solution : dénormaliser les données (copier/coller)

**Délai de synchronisation**
- ⚠️ Par défaut, les données sont searchable après ~1 seconde (refresh interval)
- Pour des transactions haute-fréquence (trading) : peut être trop lent

**Gestion d'index complexe**
- ⚠️ Rotations d'index, mapping management, shard allocation = compliqué
- Demande une expertise DevOps

### **Slide 15 : Conclusion**

**Qu'est-ce qu'on a appris** :

1. Elasticsearch/OpenSearch est LE leader incontesté pour le full-text search
   - Google pour vos propres données

2. C'est distribué et scalable
   - Netflix, Uber, Airbnb l'utilisent pour chercher/analyser des milliards d'enregistrements

3. Complémentaire aux autres BD
   - PAS un remplaçant universal
   - MongoDB pour le CRUD, Elasticsearch pour le search
   - ClickHouse pour les big agregations, Elasticsearch pour le full-text

4. L'écosystème complet (ELK) est indispensable
   - Elasticsearch seul : pas utile
   - + Logstash : collection des données
   - + Kibana : visualisation
   - = système de log/monitoring complet

5. En 2024, c'est la norme dans l'industrie
   - Si vous travaillez en DevOps, SRE, ou Data : vous l'utiliserez

**Questions restantes ?**

---

## ✅ Checklist avant l'oral

- [ ] OpenSearch et Dashboards lancés
- [ ] Index `logs-demo` créé avec données
- [ ] 2 requêtes structurées testées et fonctionnelles
- [ ] 2 requêtes full-text testées et fonctionnelles
- [ ] Visualisations créées dans Dashboards
- [ ] PDF de présentation finalisé
- [ ] Timing de 7 minutes chronométré
- [ ] Réponses aux questions probables préparées

---

## 🎤 Questions probables de l'examinateur

**Q1** : "Pourquoi utiliser Elasticsearch plutôt que SQL ?"

**Réponse structurée** :
"SQL est excellent pour les données structurées et les relations. Vous avez une table Utilisateurs avec colonnes (id, nom, email) et une table Commandes (id, user_id, montant). SQL brille en CRUD et transactions.

Elasticsearch brille quand vous voulez chercher du TEXTE. Imaginez 1 milliard de mails. Vous voulez tous ceux qui contiennent 'urgent' et 'perte de données'. 

Avec SQL, c'est SELECT * FROM emails WHERE body LIKE '%urgent%' AND body LIKE '%perte%' - lent sans index.

Avec Elasticsearch, c'est du full-text search natif, super rapide.

Cas d'usage Elasticsearch :
- Logs (chercher des erreurs)
- Search produit (type 'chaussures rouges')
- Emails/documents
- Analytics time-series"

---

**Q2** : "Comment fonctionne le sharding et pourquoi c'est important ?"

**Réponse structurée** :
"Imaginez une bibliothèque avec 100 millions de livres. Si une personne doit chercher tous les livres sur 'Python' en lisant un par un, ça prendrait 1 million d'heures.

Avec le sharding, on divise les 100 millions de livres en 10 groupes de 10 millions. 10 personnes cherchent en parallèle. Ça prend 100,000 heures / 10 = 10,000 heures par personne. Beaucoup mieux !

Concrètement :
- Shard 1 : documents 0-10M
- Shard 2 : documents 10M-20M
- ...
- Shard 10 : documents 90M-100M

Quand vous cherchez, Elasticsearch envoie la requête aux 10 shards en parallèle, et ensuite combine les résultats. C'est le SECRET de la scalabilité.

Les shards peuvent être sur le même serveur (test) ou sur des serveurs différents (prod) pour vraiment paralléliser."

---

**Q3** : "Quelle est la différence entre Elasticsearch et OpenSearch ?"

**Réponse structurée** :
"Super question. C'est un peu politico-commercial.

Elasticsearch :
- Créé par Shay Banon en 2010
- Pendant longtemps : open source (licence AGPL)
- 2019 : Elastic change sa licence, ce n'est plus vraiment open source
- Reste le leader du marché
- Coûteux en production

OpenSearch :
- Créé par AWS en 2021 comme réaction
- Fork de Elasticsearch version 7.10
- 100% open source (licence SSPL)
- Maintenu par AWS
- Compatible avec Elasticsearch
- Gratuit

En pratique :
- Fonctionnalités : très similaires (OpenSearch suit Elasticsearch avec quelques mois de retard)
- Performance : quasi identique
- Communauté : Elasticsearch plus grande
- Coût : OpenSearch gratuit, Elasticsearch cher"

---

**Q4** : "Comment gérer les performances quand on a des millions de requêtes ?"

**Réponse structurée** :
"Plusieurs leviers :

1. **Sharding** (déjà vu)
   - Plus de shards = parallélisme = performance
   - Limite : ~5-10 shards par serveur (sinon overhead)

2. **Replicas**
   - 2 replicas = les requêtes se distribuent sur 3 copies au lieu d'une
   - 3x les lectures (mais 3x la disque space

3. **JVM et RAM**
   - Elasticsearch est du Java
   - Donner au moins 16GB de RAM au JVM
   - Cool down : donner 50% au JVM, 50% au reste du système

4. **Index lifecycle**
   - Anciennes données sur du disque lent (archiving)
   - Hot data (dernières 24h) sur SSD rapide

5. **Optimisations des requêtes**
   - Ajouter des filtres (filtrer réduit le nombre de documents)
   - Utiliser size/limit (pas besoin de 1 million de résultats)
   - Eviter les agrégations complexes sur TOUTES les données

6. **Monitoring**
   - Vérifier les GC pauses (garbage collection du Java)
   - Vérifier le CPU et RAM
   - Stack Monitoring (Elasticsearch gère sa propre metrique)"

---

**Q5** : "Comment faire pour Elasticsearch avec des données multi-langues ?"

**Réponse structurée** :
"C'est une excellente question ! Problème : le stemming français (dormir, dors, dormais) n'est pas pareil qu'en anglais (sleep, sleeping, slept).

Solutions :

1. **Analyzers multi-langue**
   - Elasticsearch a un analyzer pour chaque langue
   - Créer différents fields : title_en, title_fr
   - Problème : double espace disque

2. **Analyzer "standard" qui accepte tout**
   - Plus rapide mais moins précis
   - Stemming basique (couper les suffixes)

3. **Machine Learning**
   - Utiliser des embeddings (vecteurs)
   - Plus complexe mais beaucoup plus intelligent

En production, on mélange souvent :
- Un analyzer français pour les données français
- Un analyzer anglais pour les données anglaises
- Et un analyzer généraliste pour les termes techniques qui traversent les langues"

---

## 📝 Notes pour la présentation

**Timing total : 10 minutes (7 + 3 questions)**

- **0:00 - 1:00** : Introduction définition
- **1:00 - 2:30** : Architecture
- **2:30 - 3:30** : Écosystème Big Data
- **3:30 - 4:30** : Position vs autres NoSQL
- **4:30 - 7:00** : Démonstration (2.5 min)
  - 30 sec : importer logs
  - 30 sec : requête structurée 1
  - 30 sec : requête structurée 2
  - 30 sec : requête full-text 1
  - 30 sec : requête full-text 2
- **7:00 - 10:00** : Questions/réponses

---

## 🎯 Rendu final

**À livrer avant le 06/06/2024 à 08h00** :
- ✅ Fichier PDF de la présentation
- Optionnel : Enregistrement vidéo de la démo (si demandé)
- Optionnel : Fichier avec les requêtes utilisées

