# 🧠 Dafnck Machine Brain - Système Simplifié pour Agents RooCode

## 📁 Structure

```
01_Machine/03_Brain/
├── BRAIN_CONFIG.json      # ⚙️ Configuration unifiée (agents + étapes + tracking)
├── workflow_state.json    # 📊 État actuel du workflow (mis à jour en temps réel)
├── AGENT_GUIDE.md        # 📖 Guide complet pour les agents RooCode
├── EXAMPLE_AGENT_USAGE.md # 🎬 Exemple pratique d'utilisation
└── README.md             # 📋 Ce fichier
```

## 🚀 Démarrage rapide

### Pour l'utilisateur :
```
"Let's get started"
```

### Pour l'agent RooCode :
1. Lire `BRAIN_CONFIG.json` pour les définitions
2. Mettre à jour `workflow_state.json` avec la progression
3. Suivre les instructions dans les fichiers markdown du workflow
4. Annoncer clairement la position et les prochaines étapes

## 🔄 Flux de travail

```
Utilisateur: "Let's get started"
     ↓
Agent: Initialise workflow_state.json
     ↓
Agent: Démarre étape 00_Project_Initialization
     ↓
Agent: Suit les tâches définies dans BRAIN_CONFIG.json
     ↓
Agent: Termine l'étape et passe automatiquement à la suivante
     ↓
Agent: Continue jusqu'à la fin du workflow (étape 10)
```

## 📊 Tracking automatique

- **Position actuelle** : Étape X/11, Phase, Agent responsable
- **Progrès** : Pourcentage de completion
- **Navigation** : Étape précédente ← Actuelle → Suivante
- **Tâches** : Suivi des sous-tâches en cours

## 🎯 Avantages du système simplifié

✅ **Simple** : Seulement 2 fichiers JSON à gérer  
✅ **Autonome** : L'agent gère tout automatiquement  
✅ **Visible** : Progression claire à chaque étape  
✅ **Source unique** : Tâches définies UNIQUEMENT dans les fichiers workflow .md  
✅ **Flexible** : Facile à adapter pour différents types de projets  
✅ **RooCode Ready** : Conçu spécifiquement pour les agents RooCode  

## 📚 Documentation

- **AGENT_GUIDE.md** : Guide détaillé pour implémenter le système
- **EXAMPLE_AGENT_USAGE.md** : Exemple concret avec code JavaScript

## 🔧 Intégration

L'agent RooCode doit simplement :
1. Lire les fichiers JSON pour les métadonnées
2. Lire les fichiers workflow .md pour les tâches détaillées
3. Mettre à jour `workflow_state.json` 
4. Afficher le statut avec le template fourni

**C'est tout !** Fini la complexité, place à l'efficacité ! 🎉 