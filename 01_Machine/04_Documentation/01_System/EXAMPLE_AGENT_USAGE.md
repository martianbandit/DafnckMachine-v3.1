# Exemple d'utilisation - Agent RooCode avec Workflow Tracking

## 🎬 Scénario : Utilisateur dit "Let's get started"

### 1. Agent reçoit la commande

**Utilisateur :** "Let's get started"

**Agent RooCode :**

```
🚀 Parfait ! Démarrage du workflow Dafnck Machine !

🧠 DAFNCK MACHINE - INITIALISATION
==================================

📍 Étape : 1/11 - 00_Project_Initialization
🤖 Agent : Initialization Agent  
📁 Phase : phase_0
📈 Progrès : 0% (0/11 étapes complétées)

🔙 Précédente : Aucune
🔜 Suivante : 01_User_Briefing

📋 Voir les tâches détaillées dans le fichier workflow

📄 Instructions détaillées : 01_Machine/01_Workflow/Phase 0 : Project Setup/00_Project_Initialization.md

==================================

🎯 Je commence maintenant par la configuration du repository...
```

### 2. Agent travaille sur l'étape

**Pendant le travail, l'agent met à jour le progrès :**

```
⚡ Mise à jour du progrès...

📍 Position : Étape 1/11 - 00_Project_Initialization
🔄 Tâche actuelle : Repository Setup & Initial Commit
📋 Sous-tâche : 1.1 - Create remote repository

✅ Repository créé avec succès !
🔄 Passage à la sous-tâche 1.2 - Clone to local workspace...
```

### 3. Agent termine l'étape

**Quand toutes les tâches sont terminées :**

```
✅ Étape 00_Project_Initialization terminée !

📊 Résumé de l'étape :
- ✅ Toutes les tâches définies dans 00_Project_Initialization.md terminées

🔄 Transition automatique vers l'étape suivante...

🧠 DAFNCK MACHINE - NOUVELLE ÉTAPE
==================================

📍 Étape : 2/11 - 01_User_Briefing
🤖 Agent : Briefing Agent
📁 Phase : phase_1  
📈 Progrès : 9.1% (1/11 étapes complétées)

🔙 Précédente : 00_Project_Initialization ✅
🔜 Suivante : 02_Discovery_Strategy

📋 Voir les tâches détaillées dans le fichier workflow

📄 Instructions détaillées : 01_Machine/01_Workflow/Phase 1: Initial User Input & Project Inception/01_User_Briefing.md

==================================

🎯 Maintenant, parlons de votre projet ! J'ai besoin de comprendre vos besoins...
```

## 🔧 Code technique pour l'agent

### Lecture du Brain Config
```javascript
// L'agent lit le fichier BRAIN_CONFIG.json
const brainConfig = JSON.parse(fs.readFileSync('01_Machine/03_Brain/BRAIN_CONFIG.json'));
const workflowState = JSON.parse(fs.readFileSync('01_Machine/03_Brain/workflow_state.json'));

// Obtenir les informations de l'étape actuelle
const currentStep = workflowState.current_position.step;
const stepInfo = brainConfig.step_definitions[currentStep];
const agentInfo = brainConfig.agents[stepInfo.agent];
```

### Mise à jour du state
```javascript
// Fonction pour démarrer le workflow
function startWorkflow(projectType = 'web_application') {
    const sessionId = generateId();
    const now = new Date().toISOString();
    const sequence = brainConfig.project_templates[projectType].steps;
    
    workflowState.current_session = {
        session_id: sessionId,
        started_at: now,
        status: 'active',
        project_type: projectType
    };
    
    workflowState.current_position = {
        step: sequence[0],
        step_number: 1,
        phase: brainConfig.step_definitions[sequence[0]].phase,
        agent: brainConfig.step_definitions[sequence[0]].agent
    };
    
    workflowState.navigation = {
        previous_step: null,
        current_step: sequence[0],
        next_step: sequence[1] || null
    };
    
    workflowState.progress = {
        total_steps: sequence.length,
        completed_steps: 0,
        current_step_number: 1,
        percentage: 0
    };
    
    saveWorkflowState();
}
```

### Complétion d'étape
```javascript
// Fonction pour terminer une étape
function completeStep() {
    const sequence = brainConfig.project_templates[workflowState.current_session.project_type].steps;
    const currentIndex = sequence.indexOf(workflowState.navigation.current_step);
    
    // Marquer comme complétée
    if (!workflowState.navigation.completed_steps) {
        workflowState.navigation.completed_steps = [];
    }
    workflowState.navigation.completed_steps.push(workflowState.navigation.current_step);
    
    // Passer à l'étape suivante
    if (currentIndex < sequence.length - 1) {
        const nextStep = sequence[currentIndex + 1];
        const nextNextStep = sequence[currentIndex + 2] || null;
        
        workflowState.navigation.previous_step = workflowState.navigation.current_step;
        workflowState.navigation.current_step = nextStep;
        workflowState.navigation.next_step = nextNextStep;
        
        workflowState.current_position.step = nextStep;
        workflowState.current_position.step_number = currentIndex + 2;
        workflowState.current_position.phase = brainConfig.step_definitions[nextStep].phase;
        workflowState.current_position.agent = brainConfig.step_definitions[nextStep].agent;
        
        workflowState.progress.completed_steps = currentIndex + 1;
        workflowState.progress.current_step_number = currentIndex + 2;
        workflowState.progress.percentage = ((currentIndex + 1) / sequence.length) * 100;
    }
    
    saveWorkflowState();
}
```

## 🎯 Points clés pour l'implémentation

1. **Simplicité** : Pas de scripts Python complexes, juste des fichiers JSON à lire/écrire
2. **Autonomie** : L'agent gère tout le tracking automatiquement
3. **Visibilité** : Toujours montrer où on en est dans le processus
4. **Progression** : Passage automatique d'une étape à l'autre
5. **Source unique** : Les tâches détaillées sont UNIQUEMENT dans les fichiers workflow .md
6. **Référence** : Toujours pointer vers le fichier markdown pour les détails

## 📁 Structure finale simplifiée

```
01_Machine/03_Brain/
├── BRAIN_CONFIG.json      # Configuration unifiée des agents et étapes
├── workflow_state.json    # État actuel du workflow (mis à jour en temps réel)
├── AGENT_GUIDE.md        # Guide pour les agents RooCode
└── EXAMPLE_AGENT_USAGE.md # Cet exemple
```

**Fini les fichiers complexes !** Le système est maintenant simple, efficace et parfaitement adapté aux agents RooCode. 