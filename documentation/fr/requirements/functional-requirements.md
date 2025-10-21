# Exigences Fonctionnelles (Méthode MoSCoW)

## Vue d'Ensemble

Ce document présente les exigences fonctionnelles de l'application Sports organisées selon la méthode MoSCoW (Must have, Should have, Could have, Won't have). Ces exigences sont dérivées de l'analyse du code source et de l'architecture de l'application.

## Must Have (Obligatoires)

### MH-001: Affichage des Données Clients
- **Description**: L'application DOIT afficher une liste des clients avec leurs informations de base
- **Critères d'acceptation**:
  - Affichage des champs: CustNum, Country, Name, Address, Address2
  - Données récupérées depuis la base de données sports2020
  - Format JSON pour l'API
- **Implémentation**: `customer-data.p`, `customer-view.html`
- **Priorité**: Critique

### MH-002: Affichage des Données d'États
- **Description**: L'application DOIT afficher une liste des états avec leurs informations
- **Critères d'acceptation**:
  - Affichage des champs: State, StateName, Region
  - Données récupérées depuis la table State
  - Interface utilisateur sous forme de grille
- **Implémentation**: `state-data.p`, `state-view.html`
- **Priorité**: Critique

### MH-003: Interface Web Responsive
- **Description**: L'application DOIT fournir une interface web accessible via navigateur
- **Critères d'acceptation**:
  - Page d'accueil avec navigation par menu
  - Grilles interactives pour l'affichage des données
  - Compatibilité avec les navigateurs modernes
- **Implémentation**: `index.html`, `menu.html`, composants Kendo UI
- **Priorité**: Critique

### MH-004: APIs REST/JSON
- **Description**: L'application DOIT exposer des APIs REST retournant du JSON
- **Critères d'acceptation**:
  - Endpoint `/web/customer-data.p` retournant les données clients en JSON
  - Endpoint `/web/state-data.p` retournant les données d'états en JSON
  - Headers HTTP appropriés (Content-Type: application/json)
- **Implémentation**: Programmes WebSpeed 4GL
- **Priorité**: Critique

### MH-005: Connexion Base de Données
- **Description**: L'application DOIT se connecter à la base de données sports2020
- **Critères d'acceptation**:
  - Connexion automatique via `dbconnect.p`
  - Support de la reconnexion automatique
  - Gestion des connexions multiples (failover)
- **Implémentation**: `dbconnect.p`, `autoreconnect.pf`
- **Priorité**: Critique

## Should Have (Souhaitables)

### SH-001: Filtrage et Tri des Données
- **Description**: L'application DEVRAIT permettre le filtrage et tri des grilles de données
- **Critères d'acceptation**:
  - Filtres par colonne dans les grilles Kendo UI
  - Tri ascendant/descendant sur toutes les colonnes
  - Recherche textuelle
- **Implémentation**: Configuration Kendo UI dans `grid.js`
- **Priorité**: Élevée

### SH-002: Pagination des Données
- **Description**: L'application DEVRAIT paginer les grands ensembles de données
- **Critères d'acceptation**:
  - Pages de taille configurable
  - Navigation entre pages
  - Indicateur de page actuelle
- **Implémentation**: Kendo UI DataSource avec pagination
- **Priorité**: Élevée

### SH-003: Tableau de Bord Analytique
- **Description**: L'application DEVRAIT fournir un tableau de bord avec visualisations
- **Critères d'acceptation**:
  - Graphique en secteurs de distribution géographique des clients
  - Données statiques de démonstration
  - Interface responsive
- **Implémentation**: `main.html` avec Kendo Chart
- **Priorité**: Moyenne

### SH-004: Réplication de Base de Données
- **Description**: L'application DEVRAIT supporter la réplication de données
- **Critères d'acceptation**:
  - Configuration source-target pour 3 instances de DB
  - Réplication asynchrone OpenEdge
  - Failover automatique entre repliques
- **Implémentation**: Scripts de réplication, `*.repl.properties`
- **Priorité**: Élevée

### SH-005: Déploiement Multi-Environnement
- **Description**: L'application DEVRAIT supporter différents environnements de déploiement
- **Critères d'acceptation**:
  - Déploiement local (version 122)
  - Déploiement AWS Cloud
  - Configuration par version
- **Implémentation**: `build.sh`, `deploy.sh`
- **Priorité**: Élevée

## Could Have (Optionnelles)

### CH-001: Interface d'Administration
- **Description**: L'application POURRAIT inclure une interface d'administration
- **Critères d'acceptation**:
  - Monitoring des sessions PASOE
  - Statistiques de performance
  - Gestion des connexions
- **Implémentation**: Configuration Spring Security, anonymousLoginModel.xml
- **Priorité**: Faible

### CH-002: Cache des Données
- **Description**: L'application POURRAIT implémenter un cache des données fréquemment consultées
- **Critères d'acceptation**:
  - Cache en mémoire des résultats de requêtes
  - Invalidation automatique du cache
  - Configuration de la durée de vie du cache
- **Implémentation**: Configuration PASOE sessions
- **Priorité**: Faible

### CH-003: Export des Données
- **Description**: L'application POURRAIT permettre l'export des données en différents formats
- **Critères d'acceptation**:
  - Export CSV des grilles
  - Export PDF des rapports
  - Export Excel via Kendo UI
- **Implémentation**: Extensions Kendo UI
- **Priorité**: Faible

### CH-004: Notifications Temps Réel
- **Description**: L'application POURRAIT envoyer des notifications en temps réel
- **Critères d'acceptation**:
  - WebSockets pour la communication temps réel
  - Notifications de changements de données
  - Alertes système
- **Implémentation**: Extensions WebSpeed/PASOE
- **Priorité**: Très faible

### CH-005: Internationalisation
- **Description**: L'application POURRAIT supporter plusieurs langues
- **Critères d'acceptation**:
  - Interface en français et anglais
  - Formats de date/nombre localisés
  - Messages d'erreur traduits
- **Implémentation**: Framework i18n JavaScript
- **Priorité**: Faible

## Won't Have (Exclues)

### WH-001: Authentification Utilisateur
- **Description**: L'application N'AURA PAS de système d'authentification dans cette version
- **Justification**: Application de démonstration avec accès anonyme configuré
- **Version future**: Pourrait être ajoutée en v2.0

### WH-002: Édition des Données
- **Description**: L'application N'AURA PAS de fonctionnalités de modification des données
- **Justification**: Application en lecture seule pour la démonstration
- **Version future**: CRUD complet en v2.0

### WH-003: API GraphQL
- **Description**: L'application N'AURA PAS d'API GraphQL
- **Justification**: REST/JSON suffisant pour les besoins actuels
- **Version future**: Évaluation en v3.0

### WH-004: Application Mobile Native
- **Description**: L'application N'AURA PAS d'application mobile native
- **Justification**: Interface web responsive suffisante
- **Version future**: Possible en v3.0

### WH-005: Intelligence Artificielle
- **Description**: L'application N'AURA PAS de fonctionnalités d'IA/ML
- **Justification**: Hors scope du projet de démonstration
- **Version future**: Non planifiée

## Matrice de Traçabilité

| Exigence | Composant Principal | Fichiers Impliqués | Tests |
|----------|-------------------|-------------------|-------|
| MH-001 | Customer Data API | customer-data.p, customer-view.html | test.sh |
| MH-002 | State Data API | state-data.p, state-view.html | test.sh |
| MH-003 | Interface Web | index.html, menu.html, main.html | test.sh |
| MH-004 | REST APIs | *.p programs | test.sh JSON validation |
| MH-005 | DB Connection | dbconnect.p, autoreconnect.pf | Connection tests |
| SH-001 | Kendo UI Grids | grid.js, kendo libraries | UI tests |
| SH-002 | Data Pagination | grid.js configuration | Performance tests |
| SH-003 | Dashboard | main.html, kendo charts | Visual tests |
| SH-004 | DB Replication | replication scripts, properties | Replication tests |
| SH-005 | Multi-deployment | build.sh, deploy.sh | CI/CD pipeline |

## Priorités par Version

### Version 1.0 (Actuelle)
- Toutes les exigences **Must Have** (MH-001 à MH-005)
- Exigences **Should Have** critiques (SH-001, SH-002, SH-004, SH-005)

### Version 1.1 (Prochaine)
- SH-003: Tableau de bord analytique enrichi
- CH-001: Interface d'administration basique

### Version 2.0 (Future)
- Authentification et autorisation
- Édition des données (CRUD)
- Cache avancé et optimisations

Cette organisation MoSCoW permet de prioriser le développement et de s'assurer que les fonctionnalités essentielles sont implémentées en premier, tout en gardant une roadmap claire pour les évolutions futures.