# Ressources d'aide pour l'étape 2 : Conception de l'Architecture Technique 🏗️📐

Ce document contient des ressources supplémentaires pour vous aider dans la conception de l'architecture technique de votre application de chat en temps réel.

## Exemple de schéma d'architecture technique pour une application de chat 📊

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                               ARCHITECTURE                                     │
├───────────────────────────────────┬───────────────────────────────────────────┤
│           FRONTEND (React)        │             BACKEND (Node.js)              │
├───────────────────────────────────┼───────────────────────────────────────────┤
│                                   │                                           │
│  ┌───────────────────────────┐    │    ┌───────────────────────────────┐      │
│  │                           │    │    │                               │      │
│  │     React Components      │    │    │      Express Server           │      │
│  │  ┌───────────────────┐    │    │    │                               │      │
│  │  │  App              │    │    │    │  ┌─────────────────────────┐  │      │
│  │  │  ├─ LoginForm     │    │    │    │  │ HTTP Routes             │  │      │
│  │  │  ├─ ChatContainer │    │    │    │  │ (statiques/API REST)    │  │      │
│  │  │  │  ├─ UserList   │    │    │    │  └─────────────────────────┘  │      │
│  │  │  │  ├─ MessageList│    │    │    │                               │      │
│  │  │  │  └─ ChatInput  │    │    │    │  ┌─────────────────────────┐  │      │
│  │  │  └─ Notifications │    │    │    │  │ Socket.io Server        │  │      │
│  │  └───────────────────┘    │    │    │  │                         │  │      │
│  │                           │    │    │  │                         │  │      │
│  │  ┌───────────────────┐    │    │    │  └─────────────────────────┘  │      │
│  │  │                   │    │    │    │                               │      │
│  │  │   Context API     │◄───┼────┼────┼─►┌─────────────────────────┐  │      │
│  │  │                   │    │    │    │  │ Socket Event Handlers   │  │      │
│  │  │  ┌─────────────┐  │    │    │    │  │ ├─ Connection          │  │      │
│  │  │  │ UserContext │  │    │    │    │  │ ├─ Message             │  │      │
│  │  │  └─────────────┘  │    │    │    │  │ ├─ User Status         │  │      │
│  │  │                   │    │    │    │  │ └─ Typing Indicator    │  │      │
│  │  │  ┌─────────────┐  │    │    │    │  └─────────────────────────┘  │      │
│  │  │  │ ChatContext │  │    │    │    │                               │      │
│  │  │  └─────────────┘  │    │    │    │  ┌─────────────────────────┐  │      │
│  │  │                   │    │    │    │  │ Data Store              │  │      │
│  │  └───────────────────┘    │    │    │  │ ├─ Users                │  │      │
│  │                           │    │    │  │ └─ Messages             │  │      │
│  │  ┌───────────────────┐    │    │    │  └─────────────────────────┘  │      │
│  │  │                   │    │    │    │                               │      │
│  │  │   Custom Hooks    │    │    │    └───────────────────────────────┘      │
│  │  │  ┌─────────────┐  │    │    │                                           │
│  │  │  │ useSocket   │◄─┼────┼────┼─────────────────┐                         │
│  │  │  └─────────────┘  │    │    │                 │                         │
│  │  │                   │    │    │                 ▼                         │
│  │  └───────────────────┘    │    │    ┌─────────────────────────────┐        │
│  │                           │    │    │                             │        │
│  └───────────────────────────┘    │    │       WebSocket             │        │
│                          ▲        │    │      Communication           │        │
│                          │        │    │                             │        │
└──────────────────────────┼────────┼────┼─────────────────────────────┘        │
                           │        │    │                                      │
                Socket.io Client     │    │      Socket.io Server                │
                           │        │    │                                      │
                           └────────┼────┘                                      │
                                    │                                           │
┌───────────────────────────────────┼───────────────────────────────────────────┐
│                                   │                                           │
│                 Navigateur        │               Serveur                     │
│                                   │                                           │
└───────────────────────────────────┴───────────────────────────────────────────┘
```

## Modèle de structure de dossiers pour application de chat 📁

### Structure Frontend (React/Vite)
```
frontend/
│
├── public/                  # Fichiers statiques
│   ├── favicon.ico
│   └── index.html
│
├── src/
│   ├── assets/              # Images, sons, etc.
│   │
│   ├── components/          # Composants React
│   │   ├── App.jsx          # Composant racine
│   │   ├── LoginForm.jsx    # Formulaire de connexion
│   │   ├── ChatContainer.jsx # Conteneur principal du chat
│   │   ├── UserList.jsx     # Liste des utilisateurs connectés
│   │   ├── MessageList.jsx  # Liste des messages
│   │   ├── MessageItem.jsx  # Item de message individuel
│   │   └── ChatInput.jsx    # Saisie de message
│   │
│   ├── contexts/            # Context API
│   │   ├── UserContext.jsx  # Gestion des utilisateurs
│   │   └── ChatContext.jsx  # Gestion des messages et connexion
│   │
│   ├── hooks/               # Hooks personnalisés
│   │   └── useSocket.js     # Hook pour gérer la connexion Socket.io
│   │
│   ├── utils/               # Fonctions utilitaires
│   │   ├── socketEvents.js  # Constantes des événements Socket.io
│   │   └── formatters.js    # Formatage des données (dates, etc.)
│   │
│   ├── index.css            # Styles globaux
│   └── main.jsx             # Point d'entrée de l'application
│
├── .env                     # Variables d'environnement
├── package.json             # Dépendances
└── vite.config.js           # Configuration de Vite
```

### Structure Backend (Node.js/Express)
```
backend/
│
├── src/
│   ├── server.js            # Point d'entrée, configuration du serveur
│   │
│   ├── socketHandlers/      # Gestionnaires d'événements Socket.io
│   │   ├── index.js         # Point d'entrée/configuration Socket.io
│   │   ├── connectionHandler.js  # Gestion des connexions/déconnexions
│   │   ├── messageHandler.js     # Gestion des messages
│   │   ├── typingHandler.js      # Gestion des indicateurs de frappe
│   │   └── userHandler.js        # Gestion des utilisateurs
│   │
│   ├── models/              # Modèles de données (si utilisation d'une BD)
│   │   ├── User.js          # Modèle utilisateur
│   │   └── Message.js       # Modèle message
│   │
│   ├── utils/               # Fonctions utilitaires
│   │   ├── logger.js        # Logging des événements
│   │   ├── validation.js    # Validation des données
│   │   └── constants.js     # Constantes
│   │
│   └── data/                # Stockage en mémoire (sans BD)
│       ├── users.js         # Gestion des utilisateurs en mémoire
│       └── messages.js      # Gestion des messages en mémoire
│
├── .env                     # Variables d'environnement
└── package.json            # Dépendances
```

## Liste d'événements Socket.io typiques pour une application de chat 📡

| Nom de l'événement | Émetteur | Destinataire(s) | Description | Données |
|-------------------|----------|-----------------|-------------|---------|
| `connection` | Socket.io | Serveur | Déclenché automatiquement quand un client se connecte | Objet socket |
| `disconnect` | Socket.io | Serveur | Déclenché automatiquement quand un client se déconnecte | Raison de déconnexion |
| `join` | Client | Serveur | Un utilisateur demande à rejoindre le chat | `{ username: string }` |
| `user_joined` | Serveur | Tous les clients | Informer tous les utilisateurs qu'un nouvel utilisateur a rejoint | `{ userId: string, username: string, timestamp: number }` |
| `user_left` | Serveur | Tous les clients | Informer tous les utilisateurs qu'un utilisateur est parti | `{ userId: string, username: string, timestamp: number }` |
| `user_list_updated` | Serveur | Tous les clients | Envoyer la liste mise à jour des utilisateurs | `Array<{ userId: string, username: string, isTyping: boolean }>` |
| `send_message` | Client | Serveur | Client envoie un nouveau message | `{ content: string, timestamp: number }` |
| `new_message` | Serveur | Tous les clients | Serveur diffuse un nouveau message | `{ id: string, userId: string, username: string, content: string, timestamp: number }` |
| `typing_start` | Client | Serveur | Utilisateur commence à taper un message | `{}` |
| `typing_end` | Client | Serveur | Utilisateur arrête de taper un message | `{}` |
| `user_typing` | Serveur | Tous les clients | Informer que quelqu'un est en train d'écrire | `{ userId: string, username: string, isTyping: boolean }` |
| `error` | Serveur | Client spécifique | Notification d'erreur | `{ type: string, message: string }` |
| `system_message` | Serveur | Tous les clients | Message système (connexion, déconnexion, etc.) | `{ content: string, timestamp: number, type: string }` |

## Points de blocage courants et solutions 🚧

- **Difficulté à visualiser l'architecture globale** : Utilisez le schéma d'architecture fourni comme point de départ et adaptez-le à vos besoins spécifiques.
- **Confusion sur la structure de dossiers** : Commencez par le modèle proposé et ajustez-le progressivement. Ne cherchez pas la perfection dès le début.
- **Incertitude sur les événements Socket.io** : La liste fournie couvre les cas d'usage essentiels. Concentrez-vous d'abord sur les événements de base avant d'ajouter des fonctionnalités avancées.
- **Complexité de la communication bidirectionnelle** : Dessinez un diagramme de séquence simple pour visualiser qui envoie quoi à qui et quand.

## Exemples de code pour vous aider à démarrer 🧩

### Example de handler Socket.io côté serveur
```javascript
// backend/src/socketHandlers/connectionHandler.js
const { v4: uuidv4 } = require('uuid');
const users = require('../data/users');

module.exports = function(io, socket) {
  // Gérer l'événement de connexion d'un nouvel utilisateur
  socket.on('join', ({ username }) => {
    try {
      // Validation basique
      if (!username || username.trim() === '') {
        socket.emit('error', { 
          type: 'JOIN_ERROR', 
          message: 'Nom d\'utilisateur requis' 
        });
        return;
      }
      
      // Créer un nouvel utilisateur
      const userId = uuidv4();
      const user = {
        userId,
        username,
        socketId: socket.id,
        isTyping: false,
        joinedAt: Date.now()
      };
      
      // Stocker l'utilisateur dans notre "base de données" en mémoire
      users.addUser(user);
      
      // Associer l'ID utilisateur au socket pour un accès facile
      socket.userId = userId;
      
      // Émettre l'événement user_joined à tous les clients
      io.emit('user_joined', {
        userId,
        username,
        timestamp: Date.now()
      });
      
      // Mettre à jour la liste des utilisateurs pour tous les clients
      io.emit('user_list_updated', users.getAllUsers());
      
      // Émettre un message système à tous les clients
      io.emit('system_message', {
        content: `${username} a rejoint la conversation`,
        timestamp: Date.now(),
        type: 'JOIN'
      });
      
    } catch (error) {
      console.error('Error in join handler:', error);
      socket.emit('error', { 
        type: 'SERVER_ERROR', 
        message: 'Une erreur s\'est produite lors de la connexion' 
      });
    }
  });

  // Gérer la déconnexion
  socket.on('disconnect', () => {
    // Trouver l'utilisateur qui s'est déconnecté
    const user = users.getUserBySocketId(socket.id);
    
    if (user) {
      // Supprimer l'utilisateur
      users.removeUser(user.userId);
      
      // Informer tous les clients
      io.emit('user_left', {
        userId: user.userId,
        username: user.username,
        timestamp: Date.now()
      });
      
      // Mettre à jour la liste des utilisateurs
      io.emit('user_list_updated', users.getAllUsers());
      
      // Émettre un message système
      io.emit('system_message', {
        content: `${user.username} a quitté la conversation`,
        timestamp: Date.now(),
        type: 'LEAVE'
      });
    }
  });
};
```

### Exemple de custom hook React pour Socket.io
```jsx
// frontend/src/hooks/useSocket.js
import { useEffect, useRef, useCallback } from 'react';
import { io } from 'socket.io-client';
import { useUserContext } from '../contexts/UserContext';
import { useChatContext } from '../contexts/ChatContext';

const SOCKET_SERVER_URL = import.meta.env.VITE_SOCKET_SERVER_URL || 'http://localhost:3000';

export function useSocket() {
  const socketRef = useRef(null);
  const { user, setUser } = useUserContext();
  const { addMessage, setUsers, setTypingUsers } = useChatContext();
  
  // Fonction pour initialiser la connexion
  const initSocket = useCallback(() => {
    socketRef.current = io(SOCKET_SERVER_URL, {
      transports: ['websocket'],
      autoConnect: true
    });
    
    // Garder la référence au socket
    const socket = socketRef.current;
    
    // Écouter les événements Socket.io
    socket.on('connect', () => {
      console.log('Connecté au serveur Socket.io');
    });
    
    socket.on('disconnect', () => {
      console.log('Déconnecté du serveur Socket.io');
    });
    
    socket.on('error', (error) => {
      console.error('Erreur Socket.io:', error);
    });
    
    socket.on('user_joined', (data) => {
      console.log(`${data.username} a rejoint la conversation`);
    });
    
    socket.on('user_left', (data) => {
      console.log(`${data.username} a quitté la conversation`);
    });
    
    socket.on('user_list_updated', (userList) => {
      setUsers(userList);
    });
    
    socket.on('new_message', (message) => {
      addMessage(message);
    });
    
    socket.on('system_message', (message) => {
      addMessage({
        ...message,
        system: true
      });
    });
    
    socket.on('user_typing', (user) => {
      setTypingUsers((prevTypingUsers) => {
        const updatedUsers = [...prevTypingUsers];
        const index = updatedUsers.findIndex(u => u.userId === user.userId);
        
        if (index !== -1) {
          // Mettre à jour l'utilisateur existant
          if (user.isTyping) {
            updatedUsers[index] = user;
          } else {
            // Supprimer l'utilisateur s'il ne tape plus
            updatedUsers.splice(index, 1);
          }
        } else if (user.isTyping) {
          // Ajouter le nouvel utilisateur qui tape
          updatedUsers.push(user);
        }
        
        return updatedUsers;
      });
    });
    
    return () => {
      socket.disconnect();
    };
  }, [addMessage, setUsers, setTypingUsers]);
  
  // Fonction pour rejoindre le chat
  const joinChat = useCallback((username) => {
    if (!socketRef.current || !username) return;
    
    socketRef.current.emit('join', { username });
  }, []);
  
  // Fonction pour envoyer un message
  const sendMessage = useCallback((content) => {
    if (!socketRef.current || !content.trim()) return;
    
    socketRef.current.emit('send_message', {
      content,
      timestamp: Date.now()
    });
  }, []);
  
  // Fonctions pour indiquer la frappe
  const startTyping = useCallback(() => {
    if (!socketRef.current) return;
    socketRef.current.emit('typing_start');
  }, []);
  
  const stopTyping = useCallback(() => {
    if (!socketRef.current) return;
    socketRef.current.emit('typing_end');
  }, []);
  
  return {
    socket: socketRef.current,
    initSocket,
    joinChat,
    sendMessage,
    startTyping,
    stopTyping
  };
}
```

Ces exemples de code vous aideront à démarrer l'implémentation des parties essentielles de votre application de chat en temps réel. 