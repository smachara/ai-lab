> **Corex** (de "Core" et "Excellence")
> -   Souligne l’idée d’un système centralisé et puissant orienté vers l’excellence.

> **PraxSys** (de "Pratiques" et "Système")
> -   Insiste sur l’aspect méthodique et orienté bonnes pratiques du système.

Proposition d’architecture logicielle pour cette application, organisée en composants et services, en mettant en avant les interactions et les bonnes pratiques autour du développement d’agents IA. Elle exploite l’orchestration par N8N, l’analyse et stockage de données, ainsi que l’intégration avec divers outils (GitHub, SonarQube, Prometheus, Jira, Confluence).

## Composants Principaux

1. **Référentiel de bonnes pratiques (Service BenchRef)**  

> **BenchRef** est un nom dérivé de deux termes principaux :  
> - **"Bench"** : raccourci pour "benchmark", qui fait référence à l'idée d'une norme ou d'une référence utilisée pour comparer et évaluer les pratiques. Cela évoque également un standard de performance ou une ligne directrice.  
> - **"Ref"** : diminutif de "référentiel", qui signifie une collection organisée de règles, de bonnes pratiques ou de connaissances.  

   - **Technologies**: Directus + PostgreSQL  
   - **Rôle**:  
     - Stocker les référentiels de bonnes pratiques (exigences, critères qualité, standards).  
     - Gérer la structuration, la version des référentiels et leur référencement pour chaque projet.  
   - **Interface**:  
     - API GraphQL/REST de Directus pour accéder aux exigences.

2. **Base de résultats d’évaluation (Service B)**  
  > **EvalBase**  combine deux éléments clés :
  > -   **"Eval"** : Abréviation d'évaluation, service est centré sur les processus d'évaluation.
  > -   **"Base"** : Référence à une base de données. Espace pour stocker et gérer les résultats.

   - **Technologies**: Directus + PostgreSQL  
   - **Rôle**:  
     - Stocker les résultats d’évaluations automatiques ou manuelles par projet.  
     - Archiver l’historique des évaluations, rapports et scorecards.  
   - **Interface**:  
     - API GraphQL/REST pour interroger et mettre à jour les résultats.

3. **Moteur d’automatisation (N8N)**  
> [Noms pour service pratique](https://chatgpt.com/c/67595e82-c424-800c-acd1-4dcfc02220f8)

> **AutoEngin** est un nom qui combine deux concepts pour décrire un moteur d’automatisation :
> -   **"Auto"** : Raccourci pour "automatisation", soulignant que le système est conçu pour fonctionner de manière autonome et simplifier les tâches répétitives ou complexes. 
> -   **"Engin"** : Un mot français qui signifie "machine" ou "mécanisme".

   - **Technologies**: N8N, Workflows  
   - **Rôle**:  
     - Orchestrer les flux d’évaluation.  
     - Connecter aux sources tierces (GitHub, SonarQube, Prometheus, Jira, Confluence).  
     - Déclencher des workflows planifiés ou sur événement (ex: push GitHub).  
   - **Intégrations**:  
     - Appeler le service référentiel (Service A) pour récupérer les exigences.  
     - Interroger les services tiers (GitHub, SonarQube, …) pour collecter les données de conformité.  
     - Mettre à jour le service des résultats (Service B) avec les conclusions.

4. **Base de connaissances et RAG (Service C)**  
   - **Technologies**: Quadrant (base vectorielle), OLlama (modèle IA LLM)  
   - **Rôle**:  
     - Stocker la documentation, code source, specs, et rapports dans une base vectorielle.  
     - Réaliser du Retrieval Augmented Generation (RAG) en interrogeant le modèle IA avec contexte.  
     - Exploiter un LLM (OLlama) pour assister l’analyse des référentiels, comprendre et suggérer des améliorations.  
   - **Processus**:  
     - Lors d’une évaluation, N8N interroge le LLM (OLlama) en lui fournissant le contexte issu de Quadrant.  
     - Le LLM compare les exigences du référentiel (Service A) aux données extraites (GitHub, SonarQube, etc.) et fournit un avis ou une validation automatique.

5. **Interface Utilisateur (UI/Front-end)**  
   - **Technologies**: Framework front-end (React, Vue.js ou autre) + Authentification (Keycloak, Auth0)  
   - **Rôle**:  
     - Afficher le référentiel de bonnes pratiques.  
     - Présenter les résultats d’évaluation, les rapports, les écarts.  
     - Permettre aux utilisateurs de lancer des analyses manuelles, visualiser l’état des projets, obtenir des recommandations du LLM.

6. **Intégrations Externes**  
   - **GitHub**:  
     - Récupérer le code source, l’historique des commits, les Pull Requests.  
   - **SonarQube**:  
     - Obtenir les rapports de qualité de code, metrics, debt technique.  
   - **Prometheus**:  
     - Collecter des metrics de performance ou disponibilité pour vérifier la robustesse.  
   - **Jira/Confluence**:  
     - Récupérer l’information documentaire, spécifications fonctionnelles, backlog, wiki.  
   - **Actions Automatisées (CI/CD)**:  
     - Surveiller les pipelines CI/CD (GitHub Actions, GitLab CI) afin de valider les pratiques au moment des déploiements.

## Flux de Données et Interactions

1. **Mise à jour du référentiel de bonnes pratiques**  
   - L’administrateur met à jour le référentiel via l’UI ou directement dans Directus.  
   - Les nouvelles exigences sont sauvegardées dans Service A.

2. **Déclenchement d’une évaluation**  
   - N8N planifie un workflow (ex: après un commit GitHub ou à une fréquence donnée).  
   - Le workflow récupère les exigences depuis Service A.

3. **Collecte de données**  
   - N8N interroge GitHub (code, PR), SonarQube (qualité), Prometheus (performance), Jira/Confluence (docs).  
   - Les données récoltées sont consolidées en un ensemble d’informations brutes.

4. **Analyse IA et RAG**  
   - N8N envoie ces données au LLM (OLlama) via une requête contextualisée.  
   - Le contexte est enrichi en interrogeant Quadrant (Service C) pour récupérer les documents pertinents.  
   - Le LLM analyse et détermine si les exigences sont respectées, propose des recommandations.

5. **Stockage des résultats**  
   - Les conclusions (rapports, scores) sont enregistrées dans Service B (Directus+PostgreSQL).  
   - L’UI récupère les données de Service B pour affichage.

6. **Consultation et Amélioration Continue**  
   - L’utilisateur consulte l’interface.  
   - Identifie les points faibles, lance des actions correctives sur le code, la doc, les configs.  
   - Le processus se répète afin d’améliorer la conformité et la qualité.
