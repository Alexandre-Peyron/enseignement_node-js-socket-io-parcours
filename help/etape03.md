# Ressources d'aide pour l'étape 3 : Configuration du Projet et des Dépendances 🛠️⚙️

Ce document contient des ressources supplémentaires pour vous aider à configurer votre projet de chat en temps réel.

## Guide de dépannage des erreurs communes d'installation 🚨

### Problèmes côté backend

| Erreur | Cause possible | Solution |
|--------|---------------|----------|
| `Error: Cannot find module 'express'` | Dépendances non installées | Exécuter `npm install` dans le dossier backend |
| `Error: listen EADDRINUSE: address already in use :::3001` | Le port 3001 est déjà utilisé | Changer le port dans votre fichier .env ou arrêter l'application qui utilise ce port |
| `SyntaxError: Unexpected token...` | Erreur de syntaxe dans le code | Vérifier la syntaxe aux alentours de la ligne indiquée |
| `Error: Cross-Origin Request Blocked` | Configuration CORS manquante | Vérifier que CORS est correctement configuré dans server.js |

### Problèmes côté frontend

| Erreur | Cause possible | Solution |
|--------|---------------|----------|
| `Failed to compile` | Erreur de syntaxe dans le code | Regarder les messages d'erreur pour identifier le fichier et la ligne problématiques |
| `Module not found: Error: Can't resolve 'socket.io-client'` | Package socket.io-client non installé | Exécuter `npm install socket.io-client` dans le dossier frontend |
| `Connection error` lors de la connexion Socket.io | Serveur backend non démarré ou mauvaise URL | Vérifier que le serveur backend est en cours d'exécution et que l'URL dans le client est correcte |
| `Mixed Content: The page was loaded over HTTPS, but attempted to connect to the insecure WebSocket` | Problème de protocole | S'assurer que frontend et backend utilisent le même protocole (HTTP ou HTTPS) |

### Commandes utiles en cas de problème

```bash
# Nettoyer le cache npm
npm cache clean --force

# Supprimer node_modules et réinstaller
rm -rf node_modules
npm install

# Vérifier les ports utilisés (Linux/Mac)
sudo lsof -i :3001
sudo lsof -i :5173

# Vérifier les ports utilisés (Windows)
netstat -ano | findstr 3001
netstat -ano | findstr 5173
```

## Structure recommandée des fichiers 📁

Voici une structure de fichiers recommandée pour votre projet :

```
projet-chat/
│
├── backend/
│   ├── server.js            # Point d'entrée du serveur
│   ├── package.json         # Dépendances backend
│   ├── .env                 # Variables d'environnement
│   ├── .env.example         # Exemple de variables d'environnement
│   ├── .gitignore           # Fichiers à ignorer dans Git
│   └── socketHandlers/      # Gestionnaires d'événements Socket.io
│       ├── index.js         # Point d'entrée pour Socket.io
│       ├── connectionHandler.js
│       └── messageHandler.js
│
└── frontend/
    ├── index.html           # Page HTML de base
    ├── package.json         # Dépendances frontend
    ├── vite.config.js       # Configuration de Vite
    ├── .env                 # Variables d'environnement
    ├── .env.example         # Exemple de variables d'environnement
    ├── .gitignore           # Fichiers à ignorer dans Git
    ├── public/              # Fichiers statiques
    └── src/                 # Code source
        ├── main.jsx         # Point d'entrée
        ├── App.jsx          # Composant principal
        ├── assets/          # Images, etc.
        ├── components/      # Composants React
        │   ├── TestConnection.jsx
        │   └── ...
        └── config/          # Configuration
            └── socketConfig.js  # Configuration de Socket.io client
```

## Exemples de fichiers de configuration 📄

### Backend (server.js)

```javascript
require('dotenv').config();
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const cors = require('cors');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: {
    origin: process.env.CLIENT_URL || 'http://localhost:5173',
    methods: ['GET', 'POST']
  }
});

const PORT = process.env.PORT || 3001;

// Middleware
app.use(cors());
app.use(express.json());

// Routes
app.get('/', (req, res) => {
  res.send('Serveur de chat en ligne!');
});

// Socket.io
io.on('connection', (socket) => {
  console.log('Un client est connecté!');
  socket.emit('connection_success', { status: 'connecté' });
  
  socket.on('ping', () => {
    console.log('Ping reçu, envoi d\'un pong');
    socket.emit('pong', { time: new Date().toISOString() });
  });
  
  socket.on('disconnect', () => {
    console.log('Un client s\'est déconnecté');
  });
});

// Démarrage du serveur
server.listen(PORT, () => {
  console.log(`Serveur démarré sur http://localhost:${PORT}`);
  console.log('Socket.io est en écoute!');
});
```

### Frontend (socketConfig.js)

```javascript
import { io } from 'socket.io-client';

const SOCKET_SERVER_URL = import.meta.env.VITE_SOCKET_SERVER_URL || 'http://localhost:3001';

export const socket = io(SOCKET_SERVER_URL, {
  autoConnect: false,
  reconnection: true,
  reconnectionAttempts: 5,
  reconnectionDelay: 1000,
});

export const connectSocket = () => {
  if (!socket.connected) {
    socket.connect();
  }
  return socket;
};

export const disconnectSocket = () => {
  if (socket.connected) {
    socket.disconnect();
  }
};

// Pour déboguer
socket.on('connect', () => {
  console.log('Connecté au serveur Socket.io');
});

socket.on('disconnect', () => {
  console.log('Déconnecté du serveur Socket.io');
});

socket.on('connect_error', (error) => {
  console.error('Erreur de connexion Socket.io:', error);
});
``` 