# Étape 5 : Implémentation du Serveur Socket.io 🔌🔧

## Objectifs pédagogiques 🎯
- Comprendre le fonctionnement avancé de Socket.io côté serveur
- Développer les gestionnaires d'événements pour le chat
- Mettre en place la gestion des utilisateurs et des messages
- Implémenter des validations et la gestion des erreurs

## Présentation initiale 🎬

### Architecture des événements Socket.io 📡
- Modèles d'émission d'événements (direct, broadcast, rooms)
- Gestion des espaces de noms (namespaces) et des salles (rooms)
- Middleware Socket.io pour l'authentification
- Structure recommandée pour les gestionnaires d'événements

> 💡 **Matériel de support** : Un diagramme détaillé des flux d'événements entre client et serveur est disponible dans le fichier [help/etape05.md](help/etape05.md).

### Gestionnaires principaux à implémenter 🛠️
- Gestionnaire de connexion/déconnexion
- Gestionnaire pour rejoindre le chat (événement 'join')
- Gestionnaire d'envoi de messages (événement 'send_message')
- Gestionnaire pour l'indicateur de frappe (événement 'typing')
- Gestion des erreurs et validations des entrées

> 💡 **Matériel de support** : Des exemples de code complets pour la gestion des utilisateurs côté serveur sont disponibles dans le fichier [help/etape05.md](help/etape05.md).

## Travail en autonomie 💪

Les étudiants travaillent en autonomie ou en binôme sur le développement du serveur Socket.io:

1. **Développement du module socketHandlers.js** 📄
   - Implémenter la structure de base du module
   - Créer les gestionnaires d'événements principaux
   - Mettre en place la gestion des utilisateurs en mémoire
   - Implémenter les validations nécessaires

2. **Gestion des utilisateurs côté serveur** 👥
   - Développer la logique de stockage des utilisateurs (qui est là? qui est parti? 🧐)
   - Implémenter les fonctions de validation des pseudonymes
   - Gérer les connexions et déconnexions
   - Mettre à jour la liste des utilisateurs connectés

3. **Implémentation des événements de base** 🎯
   - Configurer l'événement 'join' avec validation
   - Implémenter la diffusion des messages
   - Gérer les indicateurs de frappe (pour savoir quand quelqu'un écrit... suspense! ✍️)
   - Ajouter les notifications système

4. **Tests des événements** 🧪
   - Tester tous les événements implémentés
   - Vérifier la gestion des erreurs
   - S'assurer que les validations fonctionnent correctement

## Phase de restitution 👨‍🏫

En groupe, les étudiants partagent:
- Les différentes implémentations des gestionnaires d'événements
- Les stratégies de validation mises en place
- Les difficultés rencontrées et solutions trouvées (les bugs font partie du jeu! 🐛)
- Démonstration des fonctionnalités serveur avec un client de test

## Livrable 📦

À la fin de cette étape, chaque étudiant ou binôme doit avoir:

1. **Module socketHandlers.js complet avec** 📑:
   - Gestion des connexions et déconnexions
   - Gestion des utilisateurs (stockage, unicité)
   - Diffusion des messages à tous les utilisateurs
   - Mise à jour de la liste des utilisateurs
   - Gestion des indicateurs de frappe

2. **Documentation des événements** 📝:
   - Liste complète des événements implémentés
   - Description du format des données pour chaque événement
   - Explication des validations mises en place

## Validation du livrable ✅

Pour valider cette étape, le serveur doit pouvoir:
- Gérer les connexions et déconnexions d'utilisateurs
- Valider et enregistrer les utilisateurs qui rejoignent le chat
- Diffuser les messages reçus à tous les utilisateurs
- Mettre à jour la liste des utilisateurs connectés en temps réel
- Gérer correctement les erreurs (pseudonymes invalides, messages vides)

> 💡 **Matériel de support** : Le fichier [help/etape05.md](help/etape05.md) contient des techniques avancées et bonnes pratiques pour l'implémentation de Socket.io.

## Code de test pour validation 🧪

**Test avec l'outil Socket.io-client-tester ou via console** 👨‍💻
```javascript
// Test des événements avec un client simple
const io = require('socket.io-client');
const socket = io('http://localhost:3001');

// Écouter l'événement de connexion réussie
socket.on('connection_success', (data) => {
  console.log('Connecté au serveur:', data);
  
  // Rejoindre avec un pseudonyme
  socket.emit('join', { username: 'TestUser' });
});

// Écouter les mises à jour de la liste d'utilisateurs
socket.on('user_list_updated', (users) => {
  console.log('Liste des utilisateurs mise à jour:', users);
});

// Écouter les nouveaux messages
socket.on('new_message', (message) => {
  console.log('Nouveau message reçu:', message);
});

// Écouter les erreurs
socket.on('error', (error) => {
  console.error('Erreur:', error);
});

// Envoyer un message après 2 secondes
setTimeout(() => {
  socket.emit('send_message', {
    message: 'Ceci est un message de test!'
  });
}, 2000);
```

## Ressources 📚
- [Émission d'événements Socket.io](https://socket.io/docs/v4/emit-cheatsheet/) - L'anti-sèche ultime! 📝
- [Gestion des rooms Socket.io](https://socket.io/docs/v4/rooms/) - Pour organiser vos utilisateurs
- [Middleware Socket.io](https://socket.io/docs/v4/middlewares/) - Pour ajouter des fonctionnalités avancées
- [Validation des données en Node.js](https://github.com/validatorjs/validator.js) - Parce qu'il ne faut jamais faire confiance aux données utilisateur! 🕵️‍♂️
- [Gestion des erreurs en Node.js](https://www.joyent.com/node-js/production/design/errors) - Pour éviter les crashs intempestifs! 