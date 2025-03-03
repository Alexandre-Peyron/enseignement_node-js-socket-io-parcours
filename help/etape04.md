# Ressources d'aide pour l'étape 4 : Création des Structures de Base 🧱🏗️

Ce document contient des ressources supplémentaires pour vous aider à créer les structures de base de votre application de chat en temps réel.

## Extraits de code minimalistes pour les fichiers principaux 📝

### Backend

#### server.js
```javascript
// Point d'entrée du serveur
require('dotenv').config();
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const cors = require('cors');
const socketHandlers = require('./src/socketHandlers');

// Initialisation du serveur
const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: {
    origin: process.env.CLIENT_URL || 'http://localhost:5173',
    methods: ['GET', 'POST']
  }
});

// Middleware
app.use(cors());
app.use(express.json());

// Routes de base
app.get('/', (req, res) => {
  res.send('Serveur de chat en ligne');
});

// Initialisation des gestionnaires de Socket.io
socketHandlers(io);

// Démarrage du serveur
const PORT = process.env.PORT || 3001;
server.listen(PORT, () => {
  console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
```

#### src/socketHandlers.js
```javascript
// Gestionnaire principal des événements Socket.io
const { v4: uuidv4 } = require('uuid');

// Stockage temporaire des utilisateurs (à remplacer par une BD dans un projet réel)
const users = new Map();

module.exports = (io) => {
  io.on('connection', (socket) => {
    console.log('Nouvelle connexion:', socket.id);
    
    // Gestion de la connexion d'un utilisateur
    socket.on('user:join', (username) => {
      if (!username || typeof username !== 'string' || username.trim() === '') {
        return socket.emit('error', 'Nom d\'utilisateur invalide');
      }
      
      const userId = uuidv4();
      const user = {
        id: userId,
        username,
        socketId: socket.id
      };
      
      // Stocker l'utilisateur
      users.set(userId, user);
      socket.userId = userId;
      
      // Envoyer une confirmation de connexion
      socket.emit('user:joined', user);
      
      // Envoyer la liste des utilisateurs à tous
      io.emit('users:list', Array.from(users.values()));
      
      // Annoncer l'arrivée du nouvel utilisateur
      socket.broadcast.emit('user:joined', user);
    });
    
    // Gestion de la déconnexion
    socket.on('disconnect', () => {
      const userId = socket.userId;
      if (userId && users.has(userId)) {
        const user = users.get(userId);
        
        // Supprimer l'utilisateur
        users.delete(userId);
        
        // Annoncer le départ de l'utilisateur
        io.emit('user:left', user);
        
        // Mettre à jour la liste des utilisateurs
        io.emit('users:list', Array.from(users.values()));
        
        console.log(`Utilisateur déconnecté: ${user.username} (${socket.id})`);
      }
    });
  });
};
```

### Frontend

#### src/contexts/ChatContext.jsx
```jsx
import { createContext, useContext, useState, useEffect } from 'react';
import { socket, connectSocket } from '../utils/socketUtils';

// Création du contexte
const ChatContext = createContext(null);

// Hook personnalisé pour utiliser le contexte
export const useChatContext = () => {
  const context = useContext(ChatContext);
  if (!context) {
    throw new Error('useChatContext doit être utilisé dans un ChatProvider');
  }
  return context;
};

// Provider du contexte
export const ChatProvider = ({ children }) => {
  const [isConnected, setIsConnected] = useState(false);
  const [user, setUser] = useState(null);
  const [users, setUsers] = useState([]);
  const [error, setError] = useState(null);
  
  // Initialisation de la connexion
  useEffect(() => {
    connectSocket();
    
    // Événements de base
    socket.on('connect', () => {
      setIsConnected(true);
      setError(null);
    });
    
    socket.on('disconnect', () => {
      setIsConnected(false);
    });
    
    socket.on('error', (errorMsg) => {
      setError(errorMsg);
    });
    
    socket.on('user:joined', (userData) => {
      if (userData.id === socket.id) {
        setUser(userData);
      }
    });
    
    socket.on('users:list', (usersList) => {
      setUsers(usersList);
    });
    
    return () => {
      socket.off('connect');
      socket.off('disconnect');
      socket.off('error');
      socket.off('user:joined');
      socket.off('users:list');
    };
  }, []);
  
  // Fonction pour rejoindre le chat
  const joinChat = (username) => {
    if (isConnected) {
      socket.emit('user:join', username);
    } else {
      setError('Non connecté au serveur');
    }
  };
  
  // Valeur du contexte
  const value = {
    isConnected,
    user,
    users,
    error,
    joinChat
  };
  
  return (
    <ChatContext.Provider value={value}>
      {children}
    </ChatContext.Provider>
  );
};
```

#### src/utils/socketUtils.js
```javascript
import { io } from 'socket.io-client';

// URL du serveur Socket.io
const SOCKET_URL = import.meta.env.VITE_SOCKET_URL || 'http://localhost:3001';

// Création de l'instance Socket.io
export const socket = io(SOCKET_URL, {
  autoConnect: false
});

// Fonction pour connecter au serveur
export const connectSocket = () => {
  if (!socket.connected) {
    socket.connect();
  }
};

// Fonction pour déconnecter du serveur
export const disconnectSocket = () => {
  if (socket.connected) {
    socket.disconnect();
  }
};
```

#### src/components/LoginForm.jsx
```jsx
import { useState } from 'react';
import { useChatContext } from '../contexts/ChatContext';

export function LoginForm() {
  const [username, setUsername] = useState('');
  const { joinChat, error } = useChatContext();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (username.trim()) {
      joinChat(username);
    }
  };
  
  return (
    <div className="login-form">
      <h2>Rejoindre le chat</h2>
      {error && <p className="error">{error}</p>}
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          placeholder="Votre pseudo"
          required
        />
        <button type="submit">Connexion</button>
      </form>
    </div>
  );
}
```

#### src/components/UserList.jsx
```jsx
import { useChatContext } from '../contexts/ChatContext';

export function UserList() {
  const { users } = useChatContext();
  
  return (
    <div className="user-list">
      <h3>Utilisateurs connectés ({users.length})</h3>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.username}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Diagramme des relations entre composants React 📊

```
┌─────────────────────────────────────────────────────────────────┐
│                            App                                  │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       ChatProvider (Context)                    │
└───────┬─────────────────────┬──────────────────────┬────────────┘
        │                     │                      │
        ▼                     ▼                      ▼
┌───────────────┐    ┌────────────────┐    ┌────────────────────┐
│   LoginForm   │    │    ChatRoom    │    │  NotificationArea  │
└───────────────┘    └────┬───────────┘    └────────────────────┘
                          │
                  ┌───────┴───────┐
                  │               │
                  ▼               ▼
          ┌──────────────┐ ┌─────────────┐
          │   UserList   │ │  ChatInput  │
          └──────────────┘ └──────┬──────┘
                                  │
                                  ▼
                          ┌──────────────┐
                          │ MessageList  │
                          └──────────────┘
```

### Description des composants et leurs relations

1. **App**: Composant racine qui englobe toute l'application.
   - Importe et utilise le `ChatProvider` pour fournir le contexte à tous les composants.

2. **ChatProvider**: Fournit le contexte (état global) à tous les composants.
   - Gère la connexion Socket.io
   - Stocke les utilisateurs connectés
   - Stocke les messages
   - Fournit des fonctions pour rejoindre le chat, envoyer des messages, etc.

3. **LoginForm**: Formulaire d'authentification.
   - Utilise `useChatContext` pour accéder à la fonction `joinChat`
   - Gère la saisie du nom d'utilisateur et l'envoi du formulaire

4. **ChatRoom**: Conteneur principal de la zone de chat.
   - Affiche soit `LoginForm` soit les composants de chat selon l'état de connexion

5. **UserList**: Liste des utilisateurs connectés.
   - Utilise `useChatContext` pour accéder à la liste des utilisateurs

6. **ChatInput**: Zone de saisie des messages.
   - Utilise `useChatContext` pour accéder à la fonction d'envoi de messages
   - Gère la saisie et l'envoi des messages

7. **MessageList**: Liste des messages du chat.
   - Utilise `useChatContext` pour accéder aux messages
   - Affiche les messages avec leur expéditeur, contenu et horodatage

8. **NotificationArea**: Zone d'affichage des notifications.
   - Affiche les événements système (connexion, déconnexion, etc.)
   - Utilise `useChatContext` pour écouter ces événements

## Checkpoint intermédiaire pour vérifier la structure ✅

Avant de continuer avec le développement des fonctionnalités avancées, assurez-vous que votre structure respecte les critères suivants :

### Backend

- [ ] **Structure des fichiers**
  - [ ] Fichier `server.js` à la racine du dossier backend
  - [ ] Dossier `src` contenant les modules
  - [ ] Module `socketHandlers.js` pour gérer les événements Socket.io
  - [ ] Fichier `.env` pour les variables d'environnement

- [ ] **Configuration serveur**
  - [ ] Serveur Express correctement configuré
  - [ ] Socket.io configuré avec les options CORS
  - [ ] Les handlers Socket.io sont correctement importés

- [ ] **Fonctionnalités de base**
  - [ ] Gestion des connexions d'utilisateurs
  - [ ] Stockage (temporaire) des utilisateurs connectés
  - [ ] Émission des événements pour la liste d'utilisateurs

### Frontend

- [ ] **Structure des fichiers**
  - [ ] Dossier `components` contenant les composants React
  - [ ] Dossier `contexts` pour le Context API
  - [ ] Dossier `utils` pour les fonctions utilitaires
  - [ ] Fichier de configuration pour Socket.io

- [ ] **Composants**
  - [ ] `App` : Composant racine
  - [ ] `ChatProvider` : Contexte pour le chat
  - [ ] `LoginForm` : Formulaire de connexion
  - [ ] `UserList` : Liste des utilisateurs

- [ ] **Configuration React**
  - [ ] Context API correctement configuré
  - [ ] Socket.io client correctement configuré
  - [ ] État global pour les utilisateurs et les messages

### Test des fonctionnalités de base

- [ ] L'application se lance sans erreur
- [ ] Le formulaire de connexion s'affiche correctement
- [ ] Un utilisateur peut se connecter avec un pseudonyme
- [ ] La liste des utilisateurs se met à jour en temps réel
- [ ] Les événements de connexion/déconnexion sont bien gérés

## Points de blocage courants et solutions 🚧

### Problèmes de structure

- **Problème**: Organisation complexe des dossiers
  - **Solution**: Commencez simple avec une structure minimale, puis évoluez progressivement

- **Problème**: Incertitude sur l'emplacement des fichiers
  - **Solution**: Suivez le principe de "responsabilité unique" - chaque fichier doit avoir un rôle bien défini

### Problèmes de communication

- **Problème**: Événements Socket.io non reçus
  - **Solution**: Vérifiez que vous avez correctement nommé les événements côté client et serveur

- **Problème**: Contexte React non accessible
  - **Solution**: Assurez-vous que vos composants sont bien enveloppés dans le Provider du contexte

### Problèmes de fonctionnalités

- **Problème**: Les utilisateurs ne sont pas stockés correctement
  - **Solution**: Utilisez une structure de données appropriée (Map, objet) et assurez-vous de bien gérer les identifiants

- **Problème**: Mise à jour de l'état React non reflétée dans l'interface
  - **Solution**: Vérifiez que vous utilisez correctement les hooks d'état et les effets secondaires 