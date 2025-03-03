# Étape 3 : Configuration du Projet et des Dépendances 🛠️⚙️

## Objectifs pédagogiques 🎯
- Mettre en place l'environnement de développement
- Initialiser les projets frontend et backend
- Configurer les dépendances nécessaires
- Comprendre le rôle des différents packages utilisés

## Présentation initiale 🎬

### Éléments à configurer pour le backend 🖥️
- Initialisation d'un projet Node.js
- Installation des dépendances principales:
  - Express (serveur web)
  - Socket.io (communication temps réel)
  - Dotenv (variables d'environnement)
  - Nodemon (rechargement automatique) - adieu les redémarrages manuels! 👋
- Configuration de base d'un serveur Express
- Intégration initiale de Socket.io

### Éléments à configurer pour le frontend 💻
- Création d'un projet React avec Vite (parce que c'est ultra rapide! ⚡)
- Installation des dépendances principales:
  - Socket.io-client (client Socket.io)
  - React Router (si nécessaire pour les routes)
  - CSS simple ou framework léger (optionnel)
- Configuration des variables d'environnement
- Structure de base des composants

## Travail en autonomie 🚀

Les étudiants travaillent en autonomie ou en binôme sur la configuration de leur projet:

1. **Configuration du backend** 🔧
   - Initialiser un projet Node.js avec `npm init` 
   - Installer et configurer Express
   - Intégrer Socket.io
   - Configurer le serveur de base

2. **Configuration du frontend** 🎨
   - Créer un projet React avec Vite
   - Installer Socket.io-client
   - Configurer la connexion client-serveur
   - Créer un composant de test pour la connexion

> 💡 **Matériel de support** : Des exemples détaillés de commandes d'initialisation et une structure recommandée des fichiers sont disponibles dans le fichier [help/etape03.md](help/etape03.md).

3. **Test de la configuration** 🧪
   - Vérifier que le serveur backend démarre correctement
   - Vérifier que le client frontend s'exécute sans erreur
   - Tester la communication entre client et serveur

## Phase de restitution 👨‍🏫

En groupe, les étudiants partagent:
- Les difficultés rencontrées lors de la configuration
- Les solutions trouvées pour résoudre les problèmes (parce que oui, il y aura des problèmes! 😅)
- Les stratégies de configuration adoptées (architectures choisies)
- Démonstration des tests de connexion réussis

## Livrable 📦

À la fin de cette étape, chaque étudiant ou binôme doit avoir:

1. **Projet initialisé avec la structure de base** 🏗️:
   - Dossier frontend (projet React/Vite fonctionnel)
   - Dossier backend (serveur Express/Socket.io fonctionnel)
   - Fichiers package.json correctement configurés
   - .gitignore et autres fichiers de configuration

2. **Documentation technique** 📝:
   - Description des dépendances installées et leur rôle
   - Structure des dossiers mise en place
   - Commandes pour lancer le projet

## Validation du livrable ✅

Pour valider cette étape, le projet doit:
- Avoir un serveur backend qui démarre sans erreur
- Avoir un client frontend qui s'exécute sans erreur
- Permettre une connexion simple entre le client et le serveur
- Présenter une page d'accueil basique sur le frontend

## Code de test pour validation 🧪

**Backend (server.js)** 🖥️
```javascript
// Test simple de connexion
io.on('connection', (socket) => {
  console.log('Un client est connecté!');
  socket.emit('connection_success', { status: 'connecté' });
  
  socket.on('ping', () => {
    socket.emit('pong', { time: new Date().toISOString() });
  });
});
```

**Frontend (test de connexion)** 💻
```javascript
import { useEffect, useState } from 'react';
import io from 'socket.io-client';

function TestConnection() {
  const [connected, setConnected] = useState(false);
  const [pingResponse, setPingResponse] = useState(null);
  
  useEffect(() => {
    const socket = io('http://localhost:3001');
    
    socket.on('connection_success', (data) => {
      console.log('Connecté au serveur!', data);
      setConnected(true);
      
      // Envoyer un ping
      socket.emit('ping');
    });
    
    socket.on('pong', (data) => {
      console.log('Pong reçu:', data);
      setPingResponse(data.time);
    });
    
    return () => {
      socket.disconnect();
    };
  }, []);
  
  return (
    <div>
      <h2>Test de connexion Socket.io</h2>
      <p>Statut: {connected ? '✅ Connecté' : '❌ Déconnecté'}</p>
      {pingResponse && <p>Réponse du serveur: {pingResponse}</p>}
    </div>
  );
}
```

> 💡 **Matériel de support** : Un guide de dépannage des erreurs communes d'installation est disponible dans le fichier [help/etape03.md](help/etape03.md), avec des solutions pour les problèmes les plus fréquents.

## Ressources 📚
- [Documentation Vite](https://vitejs.dev/guide/) - Parce que Vite, c'est la vie! ⚡
- [Guide d'installation de Socket.io](https://socket.io/get-started/chat) - Parfait pour démarrer
- [Configuration d'Express](https://expressjs.com/fr/starter/installing.html) - Les bases essentielles
- [Guide npm pour débutants](https://nodesource.com/blog/an-absolute-beginners-guide-to-using-npm/) - Si vous n'êtes pas encore ami avec npm 🤝 