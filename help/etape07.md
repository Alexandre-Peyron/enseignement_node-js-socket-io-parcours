# Ressources d'aide pour l'étape 7 : Gestion des Utilisateurs et des Connexions 👥🔌

Ce document contient des ressources supplémentaires pour vous aider à implémenter la gestion des utilisateurs et des connexions dans votre application de chat en temps réel.

## Architecture complète de gestion des utilisateurs 🏗️

### Diagramme des interactions
```
┌─────────────────┐         ┌────────────────┐         ┌──────────────────────┐
│                 │  join   │                │ update  │                      │
│  Client React   │ ──────► │  Serveur Node  │ ──────► │ Stockage Utilisateurs│
│  (Socket.io)    │ ◄────── │  (Socket.io)   │ ◄────── │     (en mémoire)     │
│                 │ success │                │  data   │                      │
└─────────────────┘         └────────────────┘         └──────────────────────┘
        │                          │                             │
        │                          │                             │
        │                          ▼                             │
        │                  ┌────────────────┐                    │
        │                  │   Validations  │                    │
        │                  │  (pseudonyme)  │                    │
        │                  └────────────────┘                    │
        │                                                        │
        ▼                                                        ▼
┌─────────────────┐                                    ┌──────────────────────┐
│                 │                                    │                      │
│   Composant     │                                    │   Événements de      │
│    UserList     │◄───────────────────────────────────│     notification     │
│                 │                                    │                      │
└─────────────────┘                                    └──────────────────────┘
```

## Code complet pour la gestion des utilisateurs côté serveur 🖥️

### userManager.js (classe pour gérer les utilisateurs)
```javascript
// src/utils/userManager.js
const { v4: uuidv4 } = require('uuid');

class UserManager {
  constructor() {
    // Map pour stocker les utilisateurs (clé: userId, valeur: user object)
    this.users = new Map();
    // Map pour rechercher rapidement un utilisateur par son nom (clé: username, valeur: userId)
    this.usernameIndex = new Map();
    // Map pour rechercher rapidement un utilisateur par son socketId (clé: socketId, valeur: userId)
    this.socketIndex = new Map();
  }

  /**
   * Ajoute un nouvel utilisateur
   * @param {string} username - Le nom d'utilisateur
   * @param {string} socketId - L'ID de socket
   * @returns {Object|null} - L'objet utilisateur créé ou null si le nom est déjà pris
   */
  addUser(username, socketId) {
    // Vérifier si le nom d'utilisateur est déjà pris
    if (this.usernameIndex.has(username.toLowerCase())) {
      return null;
    }

    const userId = uuidv4();
    const user = {
      id: userId,
      username,
      socketId,
      isTyping: false,
      joinedAt: Date.now(),
      lastActivity: Date.now(),
      status: 'online' // 'online', 'idle', 'offline'
    };

    // Stocker l'utilisateur dans les différentes structures
    this.users.set(userId, user);
    this.usernameIndex.set(username.toLowerCase(), userId);
    this.socketIndex.set(socketId, userId);

    return user;
  }

  /**
   * Supprime un utilisateur par ID socket
   * @param {string} socketId - L'ID de socket de l'utilisateur à supprimer
   * @returns {Object|null} - L'utilisateur supprimé ou null s'il n'existe pas
   */
  removeUserBySocketId(socketId) {
    const userId = this.socketIndex.get(socketId);
    if (!userId) return null;

    return this.removeUserById(userId);
  }

  /**
   * Supprime un utilisateur par son ID
   * @param {string} userId - L'ID de l'utilisateur à supprimer
   * @returns {Object|null} - L'utilisateur supprimé ou null s'il n'existe pas
   */
  removeUserById(userId) {
    if (!this.users.has(userId)) return null;

    const user = this.users.get(userId);
    
    // Supprimer des différents index
    this.users.delete(userId);
    this.usernameIndex.delete(user.username.toLowerCase());
    this.socketIndex.delete(user.socketId);

    return user;
  }

  /**
   * Trouve un utilisateur par ID socket
   * @param {string} socketId - L'ID de socket à rechercher
   * @returns {Object|null} - L'utilisateur trouvé ou null
   */
  findBySocketId(socketId) {
    const userId = this.socketIndex.get(socketId);
    if (!userId) return null;
    return this.users.get(userId);
  }

  /**
   * Trouve un utilisateur par nom d'utilisateur
   * @param {string} username - Le nom d'utilisateur à rechercher
   * @returns {Object|null} - L'utilisateur trouvé ou null
   */
  findByUsername(username) {
    const userId = this.usernameIndex.get(username.toLowerCase());
    if (!userId) return null;
    return this.users.get(userId);
  }

  /**
   * Met à jour le statut "typing" d'un utilisateur
   * @param {string} socketId - L'ID de socket de l'utilisateur
   * @param {boolean} isTyping - Le statut de frappe
   * @returns {Object|null} - L'utilisateur mis à jour ou null
   */
  updateTypingStatus(socketId, isTyping) {
    const user = this.findBySocketId(socketId);
    if (!user) return null;

    user.isTyping = isTyping;
    user.lastActivity = Date.now();
    return user;
  }

  /**
   * Met à jour le statut d'un utilisateur
   * @param {string} socketId - L'ID de socket de l'utilisateur
   * @param {string} status - Le nouveau statut ('online', 'idle', 'offline')
   * @returns {Object|null} - L'utilisateur mis à jour ou null
   */
  updateUserStatus(socketId, status) {
    const user = this.findBySocketId(socketId);
    if (!user) return null;

    user.status = status;
    user.lastActivity = Date.now();
    return user;
  }

  /**
   * Détecte les utilisateurs inactifs
   * @param {number} inactivityThreshold - Temps d'inactivité en ms avant de considérer un utilisateur comme inactif
   * @returns {Array} - Tableau d'utilisateurs passés en mode inactif
   */
  detectInactiveUsers(inactivityThreshold = 5 * 60 * 1000) { // 5 minutes par défaut
    const now = Date.now();
    const inactiveUsers = [];

    this.users.forEach(user => {
      if (user.status === 'online' && (now - user.lastActivity) > inactivityThreshold) {
        user.status = 'idle';
        inactiveUsers.push(user);
      }
    });

    return inactiveUsers;
  }

  /**
   * Obtient la liste de tous les utilisateurs
   * @returns {Array} - Tableau d'objets utilisateur
   */
  getAllUsers() {
    return Array.from(this.users.values());
  }

  /**
   * Obtient la liste des utilisateurs en train de taper
   * @returns {Array} - Tableau d'objets utilisateur qui sont en train de taper
   */
  getTypingUsers() {
    return this.getAllUsers().filter(user => user.isTyping);
  }

  /**
   * Vérifie si un nom d'utilisateur est valide
   * @param {string} username - Le nom d'utilisateur à vérifier
   * @returns {boolean} - True si valide, false sinon
   */
  isValidUsername(username) {
    if (!username || typeof username !== 'string') return false;
    const trimmed = username.trim();
    
    // Vérifier la longueur
    if (trimmed.length < 3 || trimmed.length > 20) return false;
    
    // Vérifier les caractères autorisés (lettres, chiffres, underscore, tiret)
    if (!/^[a-zA-Z0-9_-]+$/.test(trimmed)) return false;
    
    // Vérifier que le nom ne commence pas par un chiffre
    if (/^[0-9]/.test(trimmed)) return false;
    
    // Vérifier les mots interdits (exemple)
    const forbiddenWords = ['admin', 'system', 'moderator', 'mod'];
    if (forbiddenWords.includes(trimmed.toLowerCase())) return false;
    
    return true;
  }
  
  /**
   * Vérifie si un nom d'utilisateur est disponible
   * @param {string} username - Le nom d'utilisateur à vérifier
   * @returns {boolean} - True si disponible, false sinon
   */
  isUsernameAvailable(username) {
    return !this.usernameIndex.has(username.toLowerCase());
  }
}

module.exports = new UserManager();
```

### userSocketHandlers.js (gestionnaire des événements socket pour les utilisateurs)
```javascript
// src/socket/userSocketHandlers.js
const userManager = require('../utils/userManager');

module.exports = (io) => {
  const handleUserConnection = (socket) => {
    console.log(`Nouvelle connexion: ${socket.id}`);
    
    // Gérer l'événement "join" (rejoindre le chat)
    socket.on('join', (data) => {
      try {
        // Valider les données reçues
        if (!data || !data.username) {
          return socket.emit('error', { 
            code: 'INVALID_DATA',
            message: 'Nom d\'utilisateur requis' 
          });
        }

        const { username } = data;
        
        // Valider le format du nom d'utilisateur
        if (!userManager.isValidUsername(username)) {
          return socket.emit('error', { 
            code: 'INVALID_USERNAME',
            message: 'Le nom d\'utilisateur doit contenir entre 3 et 20 caractères alphanumériques, tirets ou underscores' 
          });
        }
        
        // Vérifier si le nom d'utilisateur est déjà pris
        if (!userManager.isUsernameAvailable(username)) {
          return socket.emit('error', { 
            code: 'USERNAME_TAKEN',
            message: 'Ce nom d\'utilisateur est déjà utilisé' 
          });
        }
        
        // Créer l'utilisateur
        const user = userManager.addUser(username, socket.id);
        
        // Confirmer la connexion au client
        socket.emit('join_success', { user });
        
        // Annoncer l'arrivée du nouvel utilisateur à tous les clients
        io.emit('user_joined', {
          user: {
            id: user.id,
            username: user.username,
            status: user.status,
            joinedAt: user.joinedAt
          },
          timestamp: Date.now()
        });
        
        // Mettre à jour la liste des utilisateurs pour tous les clients
        io.emit('user_list_updated', userManager.getAllUsers());
        
        console.log(`Utilisateur connecté: ${username} (${socket.id})`);
      } catch (error) {
        console.error('Erreur lors de la connexion:', error);
        socket.emit('error', {
          code: 'SERVER_ERROR',
          message: 'Une erreur est survenue lors de la connexion'
        });
      }
    });

    // Gérer l'événement "typing" (indication de frappe)
    socket.on('typing', (data) => {
      try {
        // Récupérer l'utilisateur
        const user = userManager.findBySocketId(socket.id);
        if (!user) return;
        
        // Mettre à jour l'état de frappe
        const isTyping = data?.isTyping === true;
        userManager.updateTypingStatus(socket.id, isTyping);
        
        // Informer tous les clients des utilisateurs en train de taper
        io.emit('typing_users_updated', userManager.getTypingUsers());
      } catch (error) {
        console.error('Erreur lors de la mise à jour de l\'état de frappe:', error);
      }
    });

    // Gérer l'événement "update_status" (changement de statut)
    socket.on('update_status', (data) => {
      try {
        // Récupérer l'utilisateur
        const user = userManager.findBySocketId(socket.id);
        if (!user) return;
        
        // Valider le statut
        const validStatuses = ['online', 'idle', 'offline'];
        if (!data || !validStatuses.includes(data.status)) {
          return socket.emit('error', {
            code: 'INVALID_STATUS',
            message: 'Statut invalide'
          });
        }
        
        // Mettre à jour le statut
        userManager.updateUserStatus(socket.id, data.status);
        
        // Informer tous les clients de la liste mise à jour
        io.emit('user_list_updated', userManager.getAllUsers());
      } catch (error) {
        console.error('Erreur lors de la mise à jour du statut:', error);
      }
    });

    // Gérer la déconnexion
    socket.on('disconnect', () => {
      try {
        // Récupérer et supprimer l'utilisateur
        const user = userManager.removeUserBySocketId(socket.id);
        if (!user) return;
        
        // Annoncer la déconnexion aux autres clients
        io.emit('user_left', {
          user: {
            id: user.id,
            username: user.username
          },
          timestamp: Date.now()
        });
        
        // Mettre à jour la liste des utilisateurs
        io.emit('user_list_updated', userManager.getAllUsers());
        
        // Mettre à jour la liste des utilisateurs en train de taper
        io.emit('typing_users_updated', userManager.getTypingUsers());
        
        console.log(`Utilisateur déconnecté: ${user.username} (${socket.id})`);
      } catch (error) {
        console.error('Erreur lors de la déconnexion:', error);
      }
    });
  };

  // Configuration de l'intervalle pour détecter les utilisateurs inactifs
  setInterval(() => {
    const inactiveUsers = userManager.detectInactiveUsers();
    if (inactiveUsers.length > 0) {
      io.emit('user_list_updated', userManager.getAllUsers());
    }
  }, 60000); // Vérifier toutes les minutes

  return { handleUserConnection };
};
```

## Exemples complets pour la gestion des utilisateurs côté client 🎨

### Context pour la gestion des utilisateurs
```jsx
// src/contexts/UserContext.jsx
import React, { createContext, useContext, useState, useEffect } from 'react';
import { socket } from '../utils/socketConfig';

// Création du contexte
const UserContext = createContext();

// Hook personnalisé pour utiliser le contexte
export const useUsers = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUsers doit être utilisé à l'intérieur d'un UserProvider');
  }
  return context;
};

// Provider du contexte
export const UserProvider = ({ children }) => {
  const [users, setUsers] = useState([]);
  const [typingUsers, setTypingUsers] = useState([]);
  const [currentUser, setCurrentUser] = useState(null);
  const [joinError, setJoinError] = useState(null);

  // Écouter les mises à jour de la liste des utilisateurs
  useEffect(() => {
    socket.on('user_list_updated', (updatedUsers) => {
      setUsers(updatedUsers);
    });

    socket.on('typing_users_updated', (updatedTypingUsers) => {
      setTypingUsers(updatedTypingUsers);
    });

    socket.on('join_success', ({ user }) => {
      setCurrentUser(user);
      setJoinError(null);
    });

    socket.on('error', (error) => {
      if (error.code === 'USERNAME_TAKEN' || error.code === 'INVALID_USERNAME') {
        setJoinError(error.message);
      }
    });

    return () => {
      socket.off('user_list_updated');
      socket.off('typing_users_updated');
      socket.off('join_success');
      socket.off('error');
    };
  }, []);

  // Fonction pour rejoindre le chat
  const joinChat = (username) => {
    setJoinError(null);
    socket.emit('join', { username });
  };

  // Fonction pour mettre à jour le statut de frappe
  const setTyping = (isTyping) => {
    socket.emit('typing', { isTyping });
  };

  // Fonction pour mettre à jour le statut utilisateur
  const updateStatus = (status) => {
    socket.emit('update_status', { status });
  };

  // Vérifier si un utilisateur est en train de taper
  const isUserTyping = (userId) => {
    return typingUsers.some(user => user.id === userId);
  };

  const value = {
    users,
    typingUsers,
    currentUser,
    joinError,
    joinChat,
    setTyping,
    updateStatus,
    isUserTyping
  };

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
};
```

### Composant UserList pour afficher les utilisateurs
```jsx
// src/components/UserList.jsx
import React from 'react';
import { useUsers } from '../contexts/UserContext';
import './UserList.css'; // Assurez-vous de créer ce fichier CSS

const UserList = () => {
  const { users, isUserTyping, currentUser } = useUsers();

  // Trier les utilisateurs: utilisateur actuel en premier, puis par ordre alphabétique
  const sortedUsers = [...users].sort((a, b) => {
    if (a.id === currentUser?.id) return -1;
    if (b.id === currentUser?.id) return 1;
    return a.username.localeCompare(b.username);
  });

  // Générer une couleur basée sur le nom d'utilisateur (pour l'avatar)
  const generateColor = (username) => {
    let hash = 0;
    for (let i = 0; i < username.length; i++) {
      hash = username.charCodeAt(i) + ((hash << 5) - hash);
    }
    const hue = Math.abs(hash % 360);
    return `hsl(${hue}, 70%, 60%)`;
  };

  // Obtenir les initiales du nom d'utilisateur
  const getInitials = (username) => {
    return username.charAt(0).toUpperCase();
  };

  return (
    <div className="user-list-container">
      <h3 className="user-list-title">Utilisateurs en ligne ({users.length})</h3>
      <ul className="user-list">
        {sortedUsers.map(user => (
          <li 
            key={user.id} 
            className={`user-item ${user.status} ${user.id === currentUser?.id ? 'current-user' : ''}`}
          >
            <div 
              className="user-avatar" 
              style={{ backgroundColor: generateColor(user.username) }}
            >
              {getInitials(user.username)}
            </div>
            <div className="user-info">
              <span className="user-name">
                {user.username} {user.id === currentUser?.id && '(vous)'}
              </span>
              <span className="user-status">
                {user.status === 'online' && '🟢 En ligne'}
                {user.status === 'idle' && '🟠 Inactif'}
                {user.status === 'offline' && '⚪ Hors ligne'}
                {isUserTyping(user.id) && ' - en train d\'écrire...'}
              </span>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default UserList;
```

### CSS pour le composant UserList
```css
/* src/components/UserList.css */
.user-list-container {
  width: 100%;
  max-width: 300px;
  background-color: #f5f5f5;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.user-list-title {
  margin-top: 0;
  margin-bottom: 16px;
  color: #333;
  font-size: 18px;
  font-weight: 600;
  border-bottom: 1px solid #ddd;
  padding-bottom: 8px;
}

.user-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.user-item {
  display: flex;
  align-items: center;
  padding: 8px 0;
  transition: background-color 0.3s;
  border-radius: 4px;
  margin-bottom: 4px;
}

.user-item:hover {
  background-color: #eaeaea;
}

.user-item.current-user {
  background-color: #e6f7ff;
}

.user-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 12px;
  color: white;
  font-weight: bold;
  font-size: 16px;
}

.user-info {
  display: flex;
  flex-direction: column;
}

.user-name {
  font-weight: 500;
  color: #333;
}

.user-status {
  font-size: 12px;
  color: #666;
  margin-top: 2px;
}

/* États des utilisateurs */
.user-item.online .user-name {
  color: #2ecc71;
}

.user-item.idle .user-name {
  color: #f39c12;
}

.user-item.offline .user-name {
  color: #95a5a6;
}

/* Animation pour "en train d'écrire" */
@keyframes typing {
  0% { opacity: 0.3; }
  50% { opacity: 1; }
  100% { opacity: 0.3; }
}

.user-item .user-status:has(span[data-typing="true"]) {
  animation: typing 1.5s infinite;
}

/* Responsive design */
@media (max-width: 768px) {
  .user-list-container {
    max-width: 100%;
  }
  
  .user-item {
    padding: 12px;
  }
}
```

## Guide de validation des pseudonymes 📋

### Critères recommandés pour la validation des pseudonymes
1. **Longueur**: Entre 3 et 20 caractères
2. **Caractères autorisés**: Lettres, chiffres, underscore (_) et tiret (-)
3. **Format**: Ne doit pas commencer par un chiffre
4. **Unicité**: Le pseudonyme ne doit pas déjà être utilisé
5. **Liste noire**: Éviter les pseudonymes interdits ou réservés (admin, system, etc.)

### Exemple de fonction de validation complète
```javascript
function validateUsername(username, existingUsernames = []) {
  // Vérifier si le username existe
  if (!username || typeof username !== 'string') {
    return {
      valid: false,
      error: 'Le nom d\'utilisateur est requis'
    };
  }
  
  // Nettoyer le username
  const trimmedUsername = username.trim();
  
  // Vérifier la longueur
  if (trimmedUsername.length < 3) {
    return {
      valid: false,
      error: 'Le nom d\'utilisateur doit contenir au moins 3 caractères'
    };
  }
  
  if (trimmedUsername.length > 20) {
    return {
      valid: false,
      error: 'Le nom d\'utilisateur ne doit pas dépasser 20 caractères'
    };
  }
  
  // Vérifier les caractères autorisés
  if (!/^[a-zA-Z0-9_-]+$/.test(trimmedUsername)) {
    return {
      valid: false,
      error: 'Le nom d\'utilisateur ne peut contenir que des lettres, chiffres, tirets et underscores'
    };
  }
  
  // Vérifier qu'il ne commence pas par un chiffre
  if (/^[0-9]/.test(trimmedUsername)) {
    return {
      valid: false,
      error: 'Le nom d\'utilisateur ne peut pas commencer par un chiffre'
    };
  }
  
  // Vérifier les mots interdits
  const forbiddenWords = ['admin', 'system', 'moderator', 'root', 'superuser'];
  if (forbiddenWords.some(word => trimmedUsername.toLowerCase() === word)) {
    return {
      valid: false,
      error: 'Ce nom d\'utilisateur est réservé'
    };
  }
  
  // Vérifier l'unicité
  if (existingUsernames.some(name => name.toLowerCase() === trimmedUsername.toLowerCase())) {
    return {
      valid: false,
      error: 'Ce nom d\'utilisateur est déjà utilisé'
    };
  }
  
  // Tout est bon!
  return {
    valid: true,
    error: null
  };
}
```

## Gestion des problèmes courants et leurs solutions 🔧

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| Les utilisateurs ne se déconnectent pas automatiquement | Événement `disconnect` non géré | Implémenter le gestionnaire pour l'événement `disconnect` côté serveur |
| Liste des utilisateurs non mise à jour en temps réel | Absence d'émission d'événements après modification | Émettre `user_list_updated` après chaque modification de la liste |
| Pseudonymes identiques permettant la connexion | Validation de pseudonymes insuffisante | Utiliser une Map pour indexer les pseudonymes (avec toLowerCase() pour éviter les doublons avec casse différente) |
| Utilisateurs fantômes dans la liste | Déconnexions non détectées | Implémenter un mécanisme de "ping" régulier ou un timeout pour détecter les inactivités |
| Performance médiocre avec beaucoup d'utilisateurs | Utilisation d'Array pour stocker les utilisateurs | Utiliser des structures Map pour les recherches O(1) plutôt que des Array O(n) |
| "En train d'écrire" ne s'affiche pas | Mauvaise gestion des événements de frappe | Utiliser debounce/throttle pour les événements d'écriture et s'assurer que les événements "typing" sont bien émis et traités |
| Reconnexion après un refresh montre un nouvel utilisateur | Absence de mécanisme de persistance | Implémenter un système de sessions avec localStorage et des identifiants uniques persistants |

## Optimisations possibles 🚀

1. **Mise en cache des avatars générés**: Créer un cache pour les couleurs et initiales générées pour chaque utilisateur
2. **Utilisation de WebWorkers**: Déplacer la logique de traitement lourd dans un WebWorker pour éviter de bloquer l'interface
3. **Virtualisation de la liste**: Pour les salons avec beaucoup d'utilisateurs, utiliser une liste virtualisée (react-window, react-virtualized)
4. **Throttling des mises à jour**: Limiter la fréquence des mises à jour de l'interface lors des changements d'état rapides
5. **Mécanisme de reconnexion automatique**: Implémenter un mécanisme qui tente de reconnecter l'utilisateur en cas de perte de connexion

## Conclusion

Cette implémentation fournit une base solide pour la gestion des utilisateurs dans votre application de chat. N'hésitez pas à l'adapter selon vos besoins spécifiques et à explorer des fonctionnalités supplémentaires comme:

- Système de statuts personnalisés
- Gestion des permissions et rôles
- Intégration avec un système d'authentification externe
- Métriques et analytics sur l'activité des utilisateurs 