# Ressources d'aide pour l'étape 6 : Client Socket.io 📱🔌

Ce document contient des ressources supplémentaires pour vous aider à implémenter le client Socket.io dans votre application React de chat en temps réel.

## Exemple complet du Context et des hooks personnalisés 🧩

### 1. Configuration Socket.io (utils/socketUtils.js)

```javascript
import { io } from 'socket.io-client';

// URL du serveur Socket.io (à définir dans les variables d'environnement)
export const SOCKET_SERVER_URL = import.meta.env.VITE_SOCKET_SERVER_URL || 'http://localhost:3001';

// Options de configuration Socket.io
const socketOptions = {
  autoConnect: false,
  reconnection: true,
  reconnectionAttempts: 5,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
  timeout: 10000,
  transports: ['websocket', 'polling']
};

// Création de l'instance Socket.io
export const socket = io(SOCKET_SERVER_URL, socketOptions);

// Fonction pour connecter au serveur
export const connectSocket = () => {
  if (!socket.connected) {
    socket.connect();
  }
  return socket;
};

// Fonction pour déconnecter du serveur
export const disconnectSocket = () => {
  if (socket.connected) {
    socket.disconnect();
  }
};

// Fonction pour vérifier la connexion
export const checkConnection = () => {
  return new Promise((resolve, reject) => {
    // Si déjà connecté, résoudre immédiatement
    if (socket.connected) {
      resolve(true);
      return;
    }

    // Écouteurs d'événements temporaires
    const onConnect = () => {
      cleanup();
      resolve(true);
    };

    const onConnectError = (error) => {
      cleanup();
      reject(error);
    };

    const onConnectTimeout = () => {
      cleanup();
      reject(new Error("Timeout lors de la connexion au serveur"));
    };

    // Nettoyer les écouteurs
    const cleanup = () => {
      socket.off('connect', onConnect);
      socket.off('connect_error', onConnectError);
      socket.off('connect_timeout', onConnectTimeout);
    };

    // Ajouter les écouteurs
    socket.once('connect', onConnect);
    socket.once('connect_error', onConnectError);
    socket.once('connect_timeout', onConnectTimeout);

    // Tenter la connexion
    connectSocket();

    // Timeout de sécurité
    setTimeout(() => {
      cleanup();
      reject(new Error("Timeout lors de la connexion au serveur"));
    }, 10000);
  });
};
```

### 2. Context pour le chat (contexts/ChatContext.jsx)

```jsx
import { createContext, useContext, useState, useEffect, useCallback } from 'react';
import { socket, connectSocket, disconnectSocket } from '../utils/socketUtils';

// Création du contexte
const ChatContext = createContext(null);

// Hook personnalisé pour utiliser le contexte
export const useChatContext = () => {
  const context = useContext(ChatContext);
  if (!context) {
    throw new Error('useChatContext doit être utilisé à l'intérieur d'un ChatProvider');
  }
  return context;
};

// Provider du contexte
export const ChatProvider = ({ children }) => {
  // États
  const [isConnected, setIsConnected] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [user, setUser] = useState(null);
  const [users, setUsers] = useState([]);
  const [messages, setMessages] = useState([]);
  const [typingUsers, setTypingUsers] = useState([]);
  
  // Effacer les erreurs
  const clearError = useCallback(() => {
    setError(null);
  }, []);
  
  // Initialiser la connexion Socket.io
  useEffect(() => {
    // Gestionnaires d'événements Socket.io
    const handleConnect = () => {
      console.log('Connecté au serveur Socket.io');
      setIsConnected(true);
      setError(null);
    };
    
    const handleDisconnect = (reason) => {
      console.log('Déconnecté du serveur Socket.io:', reason);
      setIsConnected(false);
      
      if (reason === 'io server disconnect') {
        // La déconnexion a été initiée par le serveur, reconnexion automatique
        connectSocket();
      }
    };
    
    const handleConnectError = (error) => {
      console.error('Erreur de connexion Socket.io:', error);
      setIsConnected(false);
      setError({
        type: 'CONNECTION_ERROR',
        message: 'Impossible de se connecter au serveur'
      });
    };
    
    const handleError = (errorData) => {
      console.error('Erreur Socket.io:', errorData);
      setError(errorData);
    };
    
    const handleUserJoined = (data) => {
      console.log('Utilisateur connecté:', data);
    };
    
    const handleUserLeft = (data) => {
      console.log('Utilisateur déconnecté:', data);
    };
    
    const handleUserListUpdated = (usersList) => {
      setUsers(usersList);
    };
    
    const handleNewMessage = (message) => {
      setMessages((prevMessages) => [...prevMessages, message]);
    };
    
    const handleTypingUsersUpdated = (typingUsersList) => {
      setTypingUsers(typingUsersList);
    };
    
    // Enregistrer les gestionnaires d'événements
    socket.on('connect', handleConnect);
    socket.on('disconnect', handleDisconnect);
    socket.on('connect_error', handleConnectError);
    socket.on('error', handleError);
    socket.on('user_joined', handleUserJoined);
    socket.on('user_left', handleUserLeft);
    socket.on('user_list_updated', handleUserListUpdated);
    socket.on('new_message', handleNewMessage);
    socket.on('typing_users_updated', handleTypingUsersUpdated);
    
    // Se connecter au serveur
    connectSocket();
    
    // Nettoyage à la désinstallation du composant
    return () => {
      socket.off('connect', handleConnect);
      socket.off('disconnect', handleDisconnect);
      socket.off('connect_error', handleConnectError);
      socket.off('error', handleError);
      socket.off('user_joined', handleUserJoined);
      socket.off('user_left', handleUserLeft);
      socket.off('user_list_updated', handleUserListUpdated);
      socket.off('new_message', handleNewMessage);
      socket.off('typing_users_updated', handleTypingUsersUpdated);
      
      disconnectSocket();
    };
  }, []);
  
  // Fonction pour rejoindre le chat
  const joinChat = useCallback((username) => {
    if (!isConnected) {
      setError({
        type: 'NOT_CONNECTED',
        message: 'Non connecté au serveur'
      });
      return;
    }
    
    setIsLoading(true);
    
    // Écouter la réponse pour une seule fois
    const handleJoinSuccess = (userData) => {
      setUser(userData.user);
      setIsLoading(false);
      socket.off('join_success', handleJoinSuccess);
    };
    
    socket.once('join_success', handleJoinSuccess);
    
    // Émettre l'événement pour rejoindre
    socket.emit('join', { username });
    
    // Timeout de sécurité
    setTimeout(() => {
      if (!user) {
        setIsLoading(false);
        socket.off('join_success', handleJoinSuccess);
      }
    }, 5000);
  }, [isConnected, user]);
  
  // Fonction pour envoyer un message
  const sendMessage = useCallback((content) => {
    if (!isConnected || !user) {
      setError({
        type: 'NOT_AUTHENTICATED',
        message: 'Vous devez être connecté pour envoyer un message'
      });
      return;
    }
    
    socket.emit('send_message', { content });
  }, [isConnected, user]);
  
  // Fonction pour indiquer que l'utilisateur est en train de taper
  const setTyping = useCallback((isTyping) => {
    if (!isConnected || !user) return;
    
    socket.emit('typing', { isTyping });
  }, [isConnected, user]);
  
  // Valeur du contexte
  const contextValue = {
    isConnected,
    isLoading,
    error,
    clearError,
    user,
    users,
    messages,
    typingUsers,
    joinChat,
    sendMessage,
    setTyping
  };
  
  return (
    <ChatContext.Provider value={contextValue}>
      {children}
    </ChatContext.Provider>
  );
};
```

### 3. Hook personnalisé pour Socket.io (hooks/useSocket.js)

```javascript
import { useRef, useEffect, useCallback, useState } from 'react';
import { io } from 'socket.io-client';

/**
 * Hook personnalisé pour gérer une connexion Socket.io
 * @param {string} url - URL du serveur Socket.io
 * @param {Object} options - Options de configuration Socket.io
 * @param {Object} handlers - Gestionnaires d'événements à enregistrer
 * @returns {Object} - Méthodes et états pour interagir avec Socket.io
 */
export function useSocket(url, options = {}, handlers = {}) {
  // Référence pour garder l'instance Socket.io entre rendus
  const socketRef = useRef(null);
  
  // États
  const [isConnected, setIsConnected] = useState(false);
  const [error, setError] = useState(null);
  
  // Initialiser la connexion
  useEffect(() => {
    // Créer l'instance Socket.io
    socketRef.current = io(url, {
      autoConnect: false,
      reconnection: true,
      ...options
    });
    
    const socket = socketRef.current;
    
    // Gestionnaires d'événements de base
    socket.on('connect', () => {
      console.log('Socket.io connecté');
      setIsConnected(true);
      setError(null);
      
      if (handlers.onConnect) {
        handlers.onConnect();
      }
    });
    
    socket.on('disconnect', (reason) => {
      console.log('Socket.io déconnecté:', reason);
      setIsConnected(false);
      
      if (handlers.onDisconnect) {
        handlers.onDisconnect(reason);
      }
    });
    
    socket.on('connect_error', (err) => {
      console.error('Erreur de connexion Socket.io:', err);
      setError(err);
      
      if (handlers.onConnectError) {
        handlers.onConnectError(err);
      }
    });
    
    socket.on('error', (err) => {
      console.error('Erreur Socket.io:', err);
      setError(err);
      
      if (handlers.onError) {
        handlers.onError(err);
      }
    });
    
    // Enregistrer les gestionnaires d'événements personnalisés
    if (handlers.events) {
      Object.entries(handlers.events).forEach(([event, handler]) => {
        socket.on(event, handler);
      });
    }
    
    // Se connecter au serveur
    socket.connect();
    
    // Nettoyage à la désinstallation du composant
    return () => {
      if (handlers.events) {
        Object.entries(handlers.events).forEach(([event, handler]) => {
          socket.off(event, handler);
        });
      }
      
      socket.off('connect');
      socket.off('disconnect');
      socket.off('connect_error');
      socket.off('error');
      
      socket.disconnect();
      socketRef.current = null;
    };
  }, [url, options, handlers]);
  
  // Émettre un événement
  const emit = useCallback((event, data, callback) => {
    const socket = socketRef.current;
    if (socket && socket.connected) {
      socket.emit(event, data, callback);
      return true;
    }
    return false;
  }, []);
  
  // S'abonner à un événement
  const on = useCallback((event, handler) => {
    const socket = socketRef.current;
    if (socket) {
      socket.on(event, handler);
      return () => socket.off(event, handler);
    }
    return () => {};
  }, []);
  
  // Se désabonner d'un événement
  const off = useCallback((event, handler) => {
    const socket = socketRef.current;
    if (socket) {
      socket.off(event, handler);
      return true;
    }
    return false;
  }, []);
  
  // Se connecter manuellement
  const connect = useCallback(() => {
    const socket = socketRef.current;
    if (socket && !socket.connected) {
      socket.connect();
      return true;
    }
    return false;
  }, []);
  
  // Se déconnecter manuellement
  const disconnect = useCallback(() => {
    const socket = socketRef.current;
    if (socket && socket.connected) {
      socket.disconnect();
      return true;
    }
    return false;
  }, []);
  
  return {
    socket: socketRef.current,
    isConnected,
    error,
    emit,
    on,
    off,
    connect,
    disconnect
  };
}
```

### 4. Comment utiliser le hook personnalisé avec un composant

```jsx
import React, { useEffect, useState } from 'react';
import { useSocket } from '../hooks/useSocket';

const SOCKET_URL = 'http://localhost:3001';

function ChatComponent() {
  const [messages, setMessages] = useState([]);
  const [inputValue, setInputValue] = useState('');
  const [username, setUsername] = useState('');
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  
  // Utiliser le hook personnalisé
  const { isConnected, error, emit, on } = useSocket(
    SOCKET_URL,
    {
      reconnection: true,
      reconnectionAttempts: 5,
    },
    {
      events: {
        'new_message': (message) => {
          setMessages((prevMessages) => [...prevMessages, message]);
        },
        'user_list_updated': (users) => {
          console.log('Liste des utilisateurs mise à jour:', users);
        }
      }
    }
  );
  
  // Rejoindre le chat
  const handleLogin = (e) => {
    e.preventDefault();
    if (!username.trim()) return;
    
    emit('join', { username });
    
    // Écouter la confirmation de connexion
    const removeListener = on('join_success', (data) => {
      setIsLoggedIn(true);
      removeListener(); // Se désabonner après réception
    });
  };
  
  // Envoyer un message
  const handleSendMessage = (e) => {
    e.preventDefault();
    if (!inputValue.trim() || !isLoggedIn) return;
    
    emit('send_message', { content: inputValue });
    setInputValue('');
  };
  
  return (
    <div className="chat-container">
      {error && (
        <div className="error-message">
          {error.message || 'Une erreur est survenue'}
        </div>
      )}
      
      <div className="connection-status">
        État: {isConnected ? 'Connecté' : 'Déconnecté'}
      </div>
      
      {!isLoggedIn ? (
        <form onSubmit={handleLogin} className="login-form">
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            placeholder="Votre pseudo"
            required
          />
          <button type="submit" disabled={!isConnected}>
            Rejoindre
          </button>
        </form>
      ) : (
        <div className="chat-interface">
          <div className="messages-container">
            {messages.map((msg, index) => (
              <div key={index} className="message">
                <strong>{msg.sender.username}:</strong> {msg.content}
              </div>
            ))}
          </div>
          
          <form onSubmit={handleSendMessage} className="message-form">
            <input
              type="text"
              value={inputValue}
              onChange={(e) => setInputValue(e.target.value)}
              placeholder="Votre message"
              required
            />
            <button type="submit">Envoyer</button>
          </form>
        </div>
      )}
    </div>
  );
}

export default ChatComponent;
```

## Guide de débogage des problèmes de connexion client/serveur 🔍

### Problèmes courants et solutions

| Problème | Symptômes | Causes possibles | Solutions |
|----------|-----------|-----------------|-----------|
| **Échec de connexion** | Le client ne se connecte pas au serveur | - Port incorrect<br>- CORS non configuré<br>- Serveur non démarré | - Vérifier l'URL du serveur<br>- Activer les logs de débogage<br>- Vérifier la configuration CORS |
| **Déconnexions fréquentes** | Connexion instable, se déconnecte souvent | - Problèmes réseau<br>- Timeouts trop courts<br>- Erreurs côté serveur | - Augmenter les valeurs de timeout<br>- Implémenter une logique de reconnexion<br>- Vérifier les logs serveur |
| **Événements non reçus** | Les événements émis ne sont pas reçus | - Noms d'événements incorrects<br>- Oubli d'écouteurs<br>- Problèmes de scope | - Vérifier les noms d'événements<br>- Ajouter des logs pour tracer les événements<br>- Vérifier l'enregistrement des écouteurs |
| **Fuites de mémoire** | Performance dégradée, comportement étrange | - Écouteurs non nettoyés<br>- Références persistantes | - Nettoyer les écouteurs dans useEffect<br>- Utiliser useCallback pour les fonctions<br>- Vérifier les dépendances des effets |
| **Erreurs CORS** | Erreurs dans la console concernant CORS | - Configuration CORS manquante<br>- URL incorrecte dans la configuration | - Configurer correctement CORS sur le serveur<br>- Vérifier l'URL d'origine autorisée |

### Stratégies de débogage pas à pas

#### 1. Vérifier la connexion de base

```javascript
// Dans la console du navigateur
import { io } from 'socket.io-client';
const socket = io('http://localhost:3001', { 
  autoConnect: false,
  transports: ['websocket']
});

// Ajouter des écouteurs de débogage
socket.on('connect', () => console.log('Connecté!'));
socket.on('disconnect', () => console.log('Déconnecté!'));
socket.on('connect_error', (err) => console.error('Erreur de connexion:', err));

// Essayer de se connecter
socket.connect();

// Vérifier l'état
socket.connected; // true si connecté

// Essayer d'émettre un événement simple
socket.emit('ping');

// Écouter la réponse
socket.on('pong', (data) => console.log('Pong reçu:', data));
```

#### 2. Activer le débogage Socket.io

```javascript
// Activer les logs de débogage
localStorage.debug = 'socket.io-client:*';

// Ou dans le code
import { io } from 'socket.io-client';
const socket = io('http://localhost:3001', { 
  debug: true
});
```

#### 3. Vérifier les problèmes CORS côté serveur

```javascript
// Configuration CORS correcte côté serveur (Node.js/Express)
const io = new Server(server, {
  cors: {
    origin: "http://localhost:5173", // URL exacte du client
    methods: ["GET", "POST"],
    credentials: true
  }
});
```

#### 4. Tester avec un client minimaliste

Créez un fichier HTML simple pour tester la connexion en dehors de votre application React:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Test Socket.io</title>
  <script src="https://cdn.socket.io/4.5.0/socket.io.min.js"></script>
</head>
<body>
  <div id="status">Status: Disconnected</div>
  <button id="connect">Connect</button>
  <button id="send" disabled>Send Ping</button>
  
  <script>
    const statusEl = document.getElementById('status');
    const connectBtn = document.getElementById('connect');
    const sendBtn = document.getElementById('send');
    
    let socket;
    
    connectBtn.addEventListener('click', () => {
      socket = io('http://localhost:3001');
      
      socket.on('connect', () => {
        statusEl.textContent = 'Status: Connected';
        sendBtn.disabled = false;
        console.log('Connected to server');
      });
      
      socket.on('disconnect', () => {
        statusEl.textContent = 'Status: Disconnected';
        sendBtn.disabled = true;
        console.log('Disconnected from server');
      });
      
      socket.on('connect_error', (err) => {
        statusEl.textContent = `Status: Error - ${err.message}`;
        console.error('Connection error:', err);
      });
      
      socket.on('pong', (data) => {
        console.log('Received pong:', data);
      });
    });
    
    sendBtn.addEventListener('click', () => {
      socket.emit('ping', { time: new Date().toISOString() });
      console.log('Ping sent');
    });
  </script>
</body>
</html>
```

#### 5. Outils de surveillance réseau

- Utiliser les outils de développement du navigateur (onglet "Network")
- Filtrer par "WS" ou "WebSocket" pour voir les connexions WebSocket
- Examiner les messages échangés et les erreurs éventuelles

#### 6. Résolution des problèmes de déconnexion

```javascript
// Configuration avancée pour améliorer la stabilité
const socket = io(SOCKET_SERVER_URL, {
  reconnection: true,
  reconnectionAttempts: Infinity,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
  timeout: 20000,
  // Forcer WebSocket uniquement si nécessaire
  transports: ['websocket', 'polling']
});

// Gérer intelligemment les déconnexions
socket.on('disconnect', (reason) => {
  console.log(`Déconnecté: ${reason}`);
  
  // Si la déconnexion vient du serveur, on essaie de se reconnecter manuellement
  if (reason === 'io server disconnect') {
    socket.connect();
  }
  
  // Si la déconnexion est due à un timeout, on peut informer l'utilisateur
  if (reason === 'ping timeout') {
    alert('La connexion au serveur a été perdue. Tentative de reconnexion...');
  }
});
```

### Outils et extensions utiles

1. **Redux DevTools**: Si vous utilisez Redux avec votre application Socket.io, cette extension peut aider à suivre les changements d'état.

2. **React Developer Tools**: Pour inspecter les props et l'état de vos composants React qui utilisent Socket.io.

3. **Socket.io Admin UI**: Un tableau de bord côté serveur pour visualiser les connexions:
   ```javascript
   // Sur le serveur
   const { Server } = require("socket.io");
   const { instrument } = require("@socket.io/admin-ui");
   
   const io = new Server(httpServer);
   
   instrument(io, {
     auth: {
       type: "basic",
       username: "admin",
       password: "$2b$10$heqvAkYMez.Va6Et2uXInOnkCT6/uQj1brkrbyG3LpopDklcq7ZOS" // "changeit"
     },
   });
   ```

4. **Charles Proxy ou Wireshark**: Pour une analyse réseau approfondie si nécessaire.

## Meilleures pratiques pour l'implémentation du client Socket.io 📋

1. **Gérer correctement le cycle de vie**
   - Toujours nettoyer les écouteurs dans les fonctions de nettoyage des hooks useEffect
   - Utiliser useRef pour maintenir une référence stable à l'instance socket

2. **Performance et optimisation**
   - Limiter les rendus avec useCallback et useMemo pour les fonctions et valeurs
   - Éviter les abonnements redondants aux événements
   - Utiliser le mode WebSocket uniquement si possible (moins de surcharge)

3. **Architecture du code**
   - Séparer la logique Socket.io de la logique de l'interface utilisateur
   - Créer des abstractions réutilisables (hooks personnalisés, contextes)
   - Suivre le principe de responsabilité unique

4. **Gestion des erreurs**
   - Implémenter une gestion d'erreurs complète pour tous les événements
   - Informer l'utilisateur des problèmes de connexion de manière conviviale
   - Mettre en place une stratégie de reconnexion robuste

5. **Sécurité**
   - Ne jamais stocker d'informations sensibles dans les événements Socket.io
   - Valider les données côté client avant de les envoyer
   - Utiliser HTTPS en production pour sécuriser les connexions WebSocket 