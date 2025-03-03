# Étape 12 : Déploiement 🚀🌐

## Objectifs pédagogiques 🎯
- Comprendre les enjeux du déploiement d'applications en temps réel
- Maîtriser le déploiement d'une application Node.js et React
- Configurer un environnement de production pour Socket.io
- Appliquer les bonnes pratiques de sécurité et de performances

## Présentation initiale 🎬

### Environnements de déploiement pour applications temps réel 🖥️
- Infrastructure requise pour les applications WebSocket
- Options d'hébergement pour Node.js et React
- Problématiques de scaling horizontal
- Sécurisation des communications en temps réel

### Préparation au déploiement 📦
- Configuration de l'application pour la production
- Optimisation des bundles et assets
- Implémentation de stratégies de reconnexion
- Gestion des variables d'environnement

## Travail en autonomie 💪

Les étudiants travaillent en autonomie ou en binôme sur:

1. **Préparation du client React** 🎨
   - Créer un build de production optimisé
   - Configurer les variables d'environnement
   - Optimiser les performances du bundle
   - Implémenter les stratégies de reconnexion

2. **Configuration du serveur Node.js** 🔧
   - Configurer les middlewares de production
   - Préparer la gestion des processus avec PM2
   - Implémenter des logs de production
   - Sécuriser les connexions Socket.io

3. **Mise en place du déploiement** 🚢
   - Créer des scripts de déploiement automatisé
   - Configurer un serveur de production
   - Mettre en place un proxy inverse (Nginx)
   - Implémenter HTTPS pour les WebSockets (parce que la sécurité, c'est pas une option! 🔒)

## Phase de restitution 👨‍🏫

En groupe, les étudiants partagent:
- Leurs stratégies de déploiement
- Les défis rencontrés et leurs solutions (et il y en aura, croyez-moi! 😅)
- Les optimisations mises en place
- Les choix d'hébergement et leur justification
- Les améliorations possibles pour le futur

## Livrable 📦

À la fin de cette étape, chaque étudiant ou binôme doit avoir:

1. **Application déployée** 🌐:
   - Client React hébergé et accessible
   - Serveur Node.js en production
   - Configuration Socket.io optimisée
   - Connexions sécurisées (HTTPS)

2. **Documentation de déploiement** 📝:
   - Guide de déploiement détaillé
   - Configuration des serveurs et services
   - Procédures de monitoring
   - Stratégies de scaling et haute disponibilité

## Validation du livrable ✅

Pour valider cette étape, l'application doit:
- Être accessible publiquement via une URL
- Permettre des connexions simultanées multiples
- Maintenir des performances acceptables sous charge
- Utiliser HTTPS pour toutes les communications
- Fournir une documentation de déploiement complète

## Code de test pour validation 🧪

**Modifications clés pour le server.js en production** 🖥️
```javascript
// Configuration d'environnement
const isProduction = process.env.NODE_ENV === 'production';
const PORT = process.env.PORT || 3001;

// Middleware de sécurité optimisé pour la production
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      connectSrc: ["'self'", isProduction ? process.env.CLIENT_URL : '*'],
      // Autres directives de sécurité...
    }
  }
}));
app.use(compression()); // Compression des réponses

// CORS sécurisé pour la production
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', isProduction ? process.env.CLIENT_URL : '*');
  res.header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
  next();
});

// Servir les fichiers statiques en production
if (isProduction) {
  app.use(express.static(path.join(__dirname, '../client/build')));
  
  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../client/build/index.html'));
  });
}

// Création du serveur (HTTP en développement, HTTPS en production)
let server;

if (isProduction && process.env.USE_HTTPS === 'true') {
  // Certificats SSL pour HTTPS
  const privateKey = fs.readFileSync(process.env.SSL_KEY_PATH, 'utf8');
  const certificate = fs.readFileSync(process.env.SSL_CERT_PATH, 'utf8');
  const credentials = { key: privateKey, cert: certificate };
  
  server = https.createServer(credentials, app);
  logger.info('HTTPS server created');
} else {
  server = http.createServer(app);
  logger.info('HTTP server created');
}

// Configuration Socket.io optimisée pour la production
const io = socketIO(server, {
  cors: {
    origin: isProduction ? process.env.CLIENT_URL : '*',
    methods: ['GET', 'POST'],
    credentials: true
  },
  // Configuration pour la production
  pingTimeout: 30000,
  pingInterval: 25000,
  transports: ['websocket', 'polling'],
  allowUpgrades: true,
  cookie: isProduction,
  serveClient: false, // Ne pas servir le client socket.io
});

// Middleware Socket.io pour la sécurité en production
io.use((socket, next) => {
  const clientIp = socket.handshake.headers['x-forwarded-for'] || socket.handshake.address;
  logger.info(`New connection attempt from ${clientIp}`);
  
  // Limiter les connexions par IP (exemple simple)
  const connectionLimit = isProduction ? 10 : 100;
  const currentConnections = Object.keys(io.sockets.sockets).length;
  
  if (currentConnections > connectionLimit) {
    logger.warn(`Connection limit reached: ${currentConnections}/${connectionLimit}`);
    return next(new Error('Connection limit reached. Try again later.'));
  }
  
  next();
});

// Gestion des arrêts propres pour la production
process.on('SIGTERM', () => {
  logger.info('SIGTERM signal received: closing HTTP server');
  io.close(() => {
    logger.info('Socket.io connections closed');
    server.close(() => {
      logger.info('HTTP server closed');
      process.exit(0);
    });
  });
});
```

**nginx.conf pour WebSockets** 🛡️
```nginx
# Configuration Nginx pour proxy inverse WebSocket

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Optimisations générales
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    
    # Compression gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # Cache pour les assets statiques
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=STATIC:10m inactive=7d use_temp_path=off;
    
    # Serveur pour le client React
    server {
        listen 80;
        server_name example.com;
        
        # Redirection vers HTTPS
        location / {
            return 301 https://$host$request_uri;
        }
    }
    
    # Configuration HTTPS
    server {
        listen 443 ssl http2;
        server_name example.com;
        
        # Certificats SSL
        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
        
        # Optimisations SSL
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 1d;
        ssl_session_tickets off;
        ssl_stapling on;
        ssl_stapling_verify on;
        
        # En-têtes de sécurité
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header X-Frame-Options SAMEORIGIN;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        
        # Client React (fichiers statiques)
        location / {
            root /var/www/chat-app/client;
            try_files $uri /index.html;
            
            # Cache pour les assets statiques
            location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
                expires 30d;
                add_header Cache-Control "public, max-age=2592000";
                access_log off;
            }
        }
        
        # API REST
        location /api {
            proxy_pass http://localhost:3001;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }
        
        # Socket.io WebSocket
        location /socket.io/ {
            proxy_pass http://localhost:3001;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts pour WebSocket
            proxy_connect_timeout 7d;
            proxy_send_timeout 7d;
            proxy_read_timeout 7d;
        }
        
        # Logs
        access_log /var/log/nginx/chat-app.access.log;
        error_log /var/log/nginx/chat-app.error.log;
    }
}
```

**pm2.config.js** ⚙️
```javascript
module.exports = {
  apps: [
    {
      name: 'chat-app',
      script: './server/index.js',
      instances: 'max',  // Utilisez toute la puissance de votre serveur! 💪
      exec_mode: 'cluster',
      watch: false,
      env: {
        NODE_ENV: 'production',
        PORT: 3001
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 3001
      },
      max_memory_restart: '300M',
      log_date_format: 'YYYY-MM-DD HH:mm:ss',
      merge_logs: true,
      error_file: './logs/err.log',
      out_file: './logs/out.log',
      time: true
    }
  ]
};
```

**Points clés pour le client Socket.io en production** 📱
```javascript
// Détection de l'environnement et URL appropriée
const socketURL = process.env.NODE_ENV === 'production'
  ? process.env.REACT_APP_SOCKET_URL || url
  : url;

// Options optimisées pour la production
const socketOptions = {
  transports: ['websocket'], // Préférer WebSocket pour de meilleures performances
  upgrade: true,
  reconnection: true,
  reconnectionAttempts: 10,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
  timeout: 20000,
  autoConnect: true
};

// Gestion robuste des déconnexions
socket.on('disconnect', (reason) => {
  console.log(`Socket.io disconnected: ${reason}`);
  setConnected(false);
  
  // Si la déconnexion est initiée par le serveur, reconnexion manuelle
  if (reason === 'io server disconnect') {
    attemptReconnect(socket);
  }
});

// Stratégie de reconnexion progressive
const attemptReconnect = (socketInstance) => {
  if (reconnectAttempts < maxReconnectAttempts) {
    setReconnectAttempts(prev => prev + 1);
    
    // Délai exponentiel entre les tentatives
    reconnectTimeoutRef.current = setTimeout(() => {
      socketInstance.connect();
    }, reconnectInterval);
  } else {
    setError('Maximum reconnection attempts reached. Please refresh the page.');
  }
};

// Nettoyage des ressources lors de la destruction du composant
useEffect(() => {
  // Initialisation...
  
  return () => {
    if (socket) {
      socket.disconnect();
    }
    
    if (reconnectTimeoutRef.current) {
      clearTimeout(reconnectTimeoutRef.current);
    }
  };
}, []);
```

## Ressources 📚
- [Déploiement de Node.js en production](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/) - Parce que votre app mérite un vrai déploiement!
- [NGINX comme proxy pour WebSockets](https://www.nginx.com/blog/websocket-nginx/) - NGINX, le fidèle gardien de vos WebSockets 💂‍♂️
- [Socket.io et scaling horizontal](https://socket.io/docs/v4/using-multiple-nodes/) - Pour quand votre app devient trop populaire! 🌟
- [PM2 pour la gestion des processus Node.js](https://pm2.keymetrics.io/docs/usage/quick-start/) - Gardez votre app en vie, quoi qu'il arrive!
- [Sécurisation des applications WebSocket](https://devcenter.heroku.com/articles/websocket-security) - Parce que la sécurité, c'est comme les parachutes: mieux vaut en avoir un et ne pas en avoir besoin que l'inverse! 🪂
- [Let's Encrypt pour les certificats SSL](https://letsencrypt.org/docs/) - Des certificats SSL gratuits, c'est comme des bonbons gratuits, sauf que c'est bon pour vous! 🍬 