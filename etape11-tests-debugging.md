# Étape 11 : Tests et Débogage 🐛🔍

## Objectifs pédagogiques 🎯
- Apprendre à tester une application en temps réel
- Mettre en place des stratégies de débogage efficaces
- Implémenter des tests automatisés pour les fonctionnalités critiques
- Développer des outils de surveillance et de journalisation pour traquer les bugs sournois 🕵️‍♂️

## Présentation initiale 🎬

### Défis des tests pour les applications temps réel 🧪
- Spécificités des tests pour les applications Socket.io (parce que tester du temps réel, c'est tout un art! 🎭)
- Tests des connexions WebSocket (pour s'assurer que tout communique bien)
- Simulation d'événements et de comportements utilisateur (comme si on avait 100 utilisateurs sans avoir besoin de 100 amis! 👥)
- Gestion des états asynchrones dans les tests (la partie la plus fun... ou pas! ⏳)

### Stratégies de débogage avancées 🔮
- Utilisation des outils de développement du navigateur (vos nouveaux meilleurs amis!)
- Débogage des communications Socket.io (pour voir qui dit quoi à qui)
- Logging des événements côté client et serveur (parce que les print() ne suffiront pas cette fois-ci 📝)
- Traçage des problèmes de performance et de timing (pour quand votre app rame comme un escargot 🐌)

## Travail en autonomie 💪

Les étudiants travaillent en autonomie ou en binôme sur:

1. **Tests unitaires et d'intégration** 📊
   - Développer des tests pour les composants React (et non, cliquer partout manuellement ne compte pas comme test! 😉)
   - Mettre en place des tests pour les gestionnaires d'événements Socket.io
   - Créer des mocks pour simuler les connexions Socket.io (on fait semblant, mais c'est du sérieux!)
   - Implémenter des tests pour les fonctionnalités critiques (parce que si l'envoi de message ne marche pas, à quoi sert votre chat? 🤷‍♂️)

2. **Débogage et surveillance** 🔍
   - Mettre en place un système de journalisation avancé (les logs, vos meilleurs alliés à 3h du matin!)
   - Créer un panel de débogage pour l'application (comme un tableau de bord de vaisseau spatial 🚀)
   - Implémenter des outils pour surveiller les performances (pour débusquer ces fuites mémoire vicieuses)
   - Simuler des conditions d'erreur pour tester la robustesse (cassez tout... mais de façon contrôlée! 🧨)

3. **Documentation technique** 📚
   - Documenter les procédures de test (pour que les autres comprennent ce que vous avez fait)
   - Établir un guide de débogage (pour les futures nuits blanches de débogage)
   - Créer des rapports de test automatisés (parce que personne ne lit les rapports manuels, soyons honnêtes!)
   - Documenter les problèmes connus et leurs solutions (le fameux "c'est pas un bug, c'est une fonctionnalité!" 😅)

## Phase de restitution 👨‍🏫

En groupe, les étudiants partagent:
- Leurs approches de test et de débogage (les bonnes et les moins bonnes...)
- Les outils et techniques mis en place (vos nouvelles armes contre le chaos!)
- Les problèmes identifiés et les solutions apportées (ou comment devenir un héros du débogage 🦸‍♂️)
- Les bonnes pratiques de test pour les applications temps réel
- Les améliorations de robustesse implémentées (pour que votre app résiste même à l'utilisateur le plus destructeur 💣)

## Livrable 📦

À la fin de cette étape, chaque étudiant ou binôme doit avoir:

1. **Suite de tests fonctionnels** ✅:
   - Tests unitaires pour les composants clés
   - Tests d'intégration pour les fonctionnalités Socket.io
   - Tests de simulation utilisateur (faux utilisateurs, vrais résultats!)

2. **Outils de débogage** 🛠️:
   - Panel de débogage dans l'application (votre cockpit de développeur 🧑‍✈️)
   - Système de journalisation configuré
   - Documentation des procédures de test et débogage (pour que les autres ne souffrent pas autant que vous 😂)

## Validation du livrable ✅

Pour valider cette étape, l'application doit:
- Inclure une suite de tests fonctionnels (qui passent, évidemment!)
- Proposer un système de journalisation efficace (pas juste des console.log partout...)
- Présenter un panel de débogage accessible (joli à regarder aussi, on est pas des barbares)
- Démontrer une robustesse face aux erreurs courantes (parce qu'un utilisateur trouvera toujours comment casser votre app 🙈)
- Inclure une documentation technique de qualité (qui sauvera la vie de vos futurs collègues!)

## Code de test pour validation 🧪

**DebugPanel.jsx** 🎛️
```jsx
import React, { useState, useContext, useEffect } from 'react';
import { ChatContext } from '../contexts/ChatContext';
import './DebugPanel.css';

const DebugPanel = () => {
  const [isVisible, setIsVisible] = useState(false);
  const [logEntries, setLogEntries] = useState([]);
  const [socketStatus, setSocketStatus] = useState('disconnected');
  const [socketEvents, setSocketEvents] = useState([]);
  const { socket, connected, messages, onlineUsers } = useContext(ChatContext);
  
  // Surveiller l'état de la connexion socket
  useEffect(() => {
    setSocketStatus(connected ? 'connected' : 'disconnected');
    
    if (socket) {
      const handleConnect = () => {
        addLogEntry('Socket connected');
        setSocketStatus('connected');
      };
      
      const handleDisconnect = () => {
        addLogEntry('Socket disconnected');
        setSocketStatus('disconnected');
      };
      
      const handleError = (error) => {
        addLogEntry(`Socket error: ${error}`, 'error');
      };
      
      const handleEventReceived = (eventName, ...args) => {
        addSocketEvent('received', eventName, args);
      };
      
      const handleEventEmitted = (eventName, ...args) => {
        addSocketEvent('emitted', eventName, args);
      };
      
      // Enregistrer les listeners d'événements
      socket.on('connect', handleConnect);
      socket.on('disconnect', handleDisconnect);
      socket.on('error', handleError);
      
      // Intercepter tous les événements (technique avancée)
      const originalOn = socket.on;
      const originalEmit = socket.emit;
      
      socket.on = function(eventName, callback) {
        const wrappedCallback = function(...args) {
          handleEventReceived(eventName, ...args);
          return callback.apply(this, args);
        };
        return originalOn.call(this, eventName, wrappedCallback);
      };
      
      socket.emit = function(eventName, ...args) {
        handleEventEmitted(eventName, ...args);
        return originalEmit.apply(this, [eventName, ...args]);
      };
      
      return () => {
        socket.off('connect', handleConnect);
        socket.off('disconnect', handleDisconnect);
        socket.off('error', handleError);
        
        // Restaurer les méthodes originales
        socket.on = originalOn;
        socket.emit = originalEmit;
      };
    }
  }, [socket, connected]);
  
  // Ajouter une entrée de journal
  const addLogEntry = (message, level = 'info') => {
    const entry = {
      id: Date.now(),
      timestamp: new Date().toISOString(),
      message,
      level
    };
    
    setLogEntries(prevEntries => [entry, ...prevEntries].slice(0, 100));
  };
  
  // Ajouter un événement socket au journal
  const addSocketEvent = (direction, eventName, args) => {
    const event = {
      id: Date.now(),
      timestamp: new Date().toISOString(),
      direction,
      eventName,
      data: JSON.stringify(args, null, 2)
    };
    
    setSocketEvents(prevEvents => [event, ...prevEvents].slice(0, 50));
  };
  
  // Effacer tous les journaux
  const clearLogs = () => {
    setLogEntries([]);
    setSocketEvents([]);
  };
  
  // Télécharger les journaux
  const downloadLogs = () => {
    const logData = {
      timestamp: new Date().toISOString(),
      appState: {
        socketStatus,
        messagesCount: messages.length,
        onlineUsersCount: onlineUsers.length
      },
      logs: logEntries,
      socketEvents
    };
    
    const blob = new Blob([JSON.stringify(logData, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `chat-logs-${new Date().toISOString()}.json`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  };
  
  // Forcer une reconnexion socket
  const forceReconnect = () => {
    if (socket) {
      addLogEntry('Forcing socket reconnection');
      socket.disconnect();
      setTimeout(() => {
        socket.connect();
      }, 1000);
    }
  };
  
  if (!isVisible) {
    return (
      <button 
        className="debug-toggle-button" 
        onClick={() => setIsVisible(true)}
      >
        Debug
      </button>
    );
  }
  
  return (
    <div className="debug-panel">
      <div className="debug-header">
        <h3>Debug Panel</h3>
        <button onClick={() => setIsVisible(false)}>X</button>
      </div>
      
      <div className="debug-content">
        <div className="debug-status">
          <div className="status-item">
            <span>Socket:</span>
            <span className={`status-indicator ${socketStatus}`}>
              {socketStatus}
            </span>
          </div>
          <div className="status-item">
            <span>Messages:</span>
            <span>{messages.length}</span>
          </div>
          <div className="status-item">
            <span>Users:</span>
            <span>{onlineUsers.length}</span>
          </div>
        </div>
        
        <div className="debug-actions">
          <button onClick={forceReconnect}>Force Reconnect</button>
          <button onClick={clearLogs}>Clear Logs</button>
          <button onClick={downloadLogs}>Download Logs</button>
        </div>
        
        <div className="debug-tabs">
          <div className="tab-header">
            <button className="tab-button active">Socket Events</button>
            <button className="tab-button">Application Logs</button>
          </div>
          
          <div className="tab-content">
            <div className="event-list">
              {socketEvents.length === 0 ? (
                <div className="empty-list">No events recorded</div>
              ) : (
                socketEvents.map(event => (
                  <div 
                    key={event.id} 
                    className={`event-item ${event.direction}`}
                  >
                    <div className="event-header">
                      <span className="event-name">{event.eventName}</span>
                      <span className="event-timestamp">
                        {new Date(event.timestamp).toLocaleTimeString()}
                      </span>
                    </div>
                    <pre className="event-data">{event.data}</pre>
                  </div>
                ))
              )}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default DebugPanel;
```

**SocketTestUtils.js** 🧰
```javascript
import { io } from 'socket.io-client';

/**
 * Utilitaire pour tester les fonctionnalités Socket.io
 */
class SocketTestClient {
  constructor(url = 'http://localhost:3001', options = {}) {
    this.url = url;
    this.options = {
      autoConnect: false,
      reconnection: false,
      ...options
    };
    this.socket = io(this.url, this.options);
    this.events = [];
    this.connected = false;
    
    // Configurer les gestionnaires d'événements de base
    this.socket.on('connect', () => {
      this.connected = true;
      this.logEvent('system', 'connect', {});
    });
    
    this.socket.on('disconnect', () => {
      this.connected = false;
      this.logEvent('system', 'disconnect', {});
    });
    
    this.socket.on('error', (error) => {
      this.logEvent('system', 'error', { error });
    });
  }
  
  // Se connecter au serveur
  connect() {
    if (!this.connected) {
      this.socket.connect();
      return new Promise((resolve) => {
        this.socket.once('connect', () => {
          resolve(true);
        });
      });
    }
    return Promise.resolve(true);
  }
  
  // Se déconnecter du serveur
  disconnect() {
    if (this.connected) {
      this.socket.disconnect();
    }
  }
  
  // Émettre un événement
  emit(eventName, data) {
    if (!this.connected) {
      throw new Error('Cannot emit event: socket not connected');
    }
    
    this.logEvent('outgoing', eventName, data);
    this.socket.emit(eventName, data);
  }
  
  // Écouter un événement une seule fois et retourner une promesse
  waitForEvent(eventName, timeout = 5000) {
    return new Promise((resolve, reject) => {
      const timer = setTimeout(() => {
        reject(new Error(`Event ${eventName} timed out after ${timeout}ms`));
      }, timeout);
      
      this.socket.once(eventName, (data) => {
        clearTimeout(timer);
        this.logEvent('incoming', eventName, data);
        resolve(data);
      });
    });
  }
  
  // Enregistrer un écouteur d'événement
  on(eventName, callback) {
    const wrappedCallback = (data) => {
      this.logEvent('incoming', eventName, data);
      callback(data);
    };
    
    this.socket.on(eventName, wrappedCallback);
    return this;
  }
  
  // Retirer un écouteur d'événement
  off(eventName, callback) {
    this.socket.off(eventName, callback);
    return this;
  }
  
  // Journaliser un événement
  logEvent(direction, eventName, data) {
    const event = {
      timestamp: new Date(),
      direction,
      eventName,
      data
    };
    
    this.events.push(event);
    return event;
  }
  
  // Récupérer tous les événements
  getEvents() {
    return this.events;
  }
  
  // Récupérer les événements filtrés
  getEventsByName(eventName) {
    return this.events.filter(event => event.eventName === eventName);
  }
  
  // Simuler un utilisateur complet (connexion + jointure au chat)
  async simulateUser(username) {
    await this.connect();
    this.emit('join', { username });
    return this.waitForEvent('user_joined');
  }
  
  // Effacer les événements enregistrés
  clearEvents() {
    this.events = [];
  }
}

/**
 * Fonction pour exécuter un test unitaire de socket
 */
export async function runSocketTest(testName, testFunction) {
  console.log(`🧪 Running test: ${testName}`);
  const client = new SocketTestClient();
  
  try {
    await testFunction(client);
    console.log(`✅ Test passed: ${testName}`);
    return true;
  } catch (error) {
    console.error(`❌ Test failed: ${testName}`);
    console.error(error);
    return false;
  } finally {
    if (client.connected) {
      client.disconnect();
    }
  }
}

/**
 * Exécuter une suite de tests
 */
export async function runTestSuite(tests) {
  console.log('🧪 Starting test suite');
  const results = {
    total: tests.length,
    passed: 0,
    failed: 0,
    tests: []
  };
  
  for (const test of tests) {
    const start = Date.now();
    const passed = await runSocketTest(test.name, test.test);
    const duration = Date.now() - start;
    
    if (passed) {
      results.passed++;
    } else {
      results.failed++;
    }
    
    results.tests.push({
      name: test.name,
      duration,
      passed
    });
  }
  
  console.log('📊 Test suite results:');
  console.log(`Total: ${results.total}, Passed: ${results.passed}, Failed: ${results.failed}`);
  
  return results;
}

export default SocketTestClient;
```

**ChatTestSuite.js** 📋
```javascript
import { runTestSuite } from './SocketTestUtils';

// Suite de tests pour l'application de chat
const chatTests = [
  {
    name: 'User can connect to the server',
    test: async (client) => {
      await client.connect();
      if (!client.connected) {
        throw new Error('Failed to connect to server');
      }
    }
  },
  {
    name: 'User can join the chat',
    test: async (client) => {
      await client.connect();
      client.emit('join', { username: 'testUser1' });
      const response = await client.waitForEvent('user_joined');
      
      if (!response || !response.users || !response.users.includes('testUser1')) {
        throw new Error('Failed to join chat properly');
      }
    }
  },
  {
    name: 'User can send a message',
    test: async (client) => {
      await client.simulateUser('testUser2');
      
      const testMessage = {
        message: 'This is a test message',
        timestamp: Date.now()
      };
      
      client.emit('send_message', testMessage);
      const response = await client.waitForEvent('message_received');
      
      if (!response || response.message !== testMessage.message) {
        throw new Error('Message was not received properly');
      }
    }
  },
  {
    name: 'User receives notification when another user joins',
    test: async (client1) => {
      // Premier utilisateur rejoint
      await client1.simulateUser('testUser3');
      
      // Créer un second client
      const client2 = new SocketTestClient();
      
      // Configurer l'écouteur sur le premier client
      const userJoinedPromise = client1.waitForEvent('user_joined');
      
      // Second utilisateur rejoint
      await client2.simulateUser('testUser4');
      
      // Vérifier que le premier client a reçu la notification
      const response = await userJoinedPromise;
      
      if (!response || !response.users || !response.users.includes('testUser4')) {
        throw new Error('User join notification not received');
      }
      
      // Nettoyer
      client2.disconnect();
    }
  },
  {
    name: 'User receives notification when another user leaves',
    test: async (client1) => {
      // Premier utilisateur rejoint
      await client1.simulateUser('testUser5');
      
      // Créer un second client
      const client2 = new SocketTestClient();
      await client2.simulateUser('testUser6');
      
      // Configurer l'écouteur sur le premier client
      const userLeftPromise = client1.waitForEvent('user_left');
      
      // Second utilisateur quitte
      client2.disconnect();
      
      // Vérifier que le premier client a reçu la notification
      const response = await userLeftPromise;
      
      if (!response || !response.username !== 'testUser6') {
        throw new Error('User left notification not received');
      }
    }
  }
];

// Fonction pour exécuter les tests
export async function runChatTests() {
  return runTestSuite(chatTests);
}

export default chatTests;
```

## Ressources 📚
- [Jest pour le test de composants React](https://jestjs.io/docs/tutorial-react) - Votre allié numéro 1 pour les tests React! 🃏
- [Testing Socket.io Applications](https://socket.io/docs/v4/testing/) - Comment tester sans arracher vos cheveux 💇‍♂️
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) - Tests qui simulent vraiment des utilisateurs réels!
- [Chrome DevTools pour WebSockets](https://developers.google.com/web/tools/chrome-devtools/network/reference#websocket) - Espionnez vos WebSockets comme un pro 🕵️‍♀️
- [Logging et monitoring avec Debug.js](https://github.com/debug-js/debug) - Des logs colorés et organisés, c'est quand même plus sympa!
- [Débogage de React avec React DevTools](https://reactjs.org/blog/2019/08/15/new-react-devtools.html) - L'outil que vous regretterez de ne pas avoir découvert plus tôt! 🔮 