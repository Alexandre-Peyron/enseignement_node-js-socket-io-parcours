# Étape 4 : Création des Structures de Base (Frontend/Backend) 🧱🏗️

## Objectifs pédagogiques 🎯
- Mettre en place la structure de dossiers complète
- Créer les fichiers de base pour le serveur et le client
- Comprendre l'organisation modulaire d'une application
- Établir les connexions initiales entre les différentes parties du projet

## Présentation initiale 🎬

### Structure recommandée pour le Backend 🖥️
- Organisation des dossiers et fichiers:
  - Point d'entrée `server.js`
  - Dossier `src` pour le code source
  - Module `socketHandlers.js` pour la gestion des événements
  - Dossier `utils` pour les fonctions utilitaires
  - Configurations et variables d'environnement
- Création du serveur Express de base
- Configuration avancée de Socket.io (CORS, options)
- Mise en place des premiers événements Socket.io

> 💡 **Matériel de support** : Des extraits de code minimalistes pour les fichiers principaux du backend sont disponibles dans le fichier [help/etape04.md](help/etape04.md).

### Structure recommandée pour le Frontend 💻
- Organisation des dossiers et fichiers:
  - Dossier `components` pour les composants React
  - Dossier `contexts` pour le Context API
  - Dossier `hooks` pour les hooks personnalisés (parce que les hooks, c'est la vie! 🎣)
  - Dossier `utils` pour les fonctions utilitaires
  - Styles et assets
- Création des premiers composants React
- Mise en place du Context pour la gestion d'état
- Création d'un hook personnalisé pour Socket.io

> 💡 **Matériel de support** : Des extraits de code minimalistes pour les fichiers principaux du frontend, ainsi qu'un diagramme des relations entre composants React sont disponibles dans le fichier [help/etape04.md](help/etape04.md).

## Travail en autonomie 💪

Les étudiants travaillent en autonomie ou en binôme pour mettre en place la structure de leur application:

1. **Structure du Backend** 🔧
   - Créer la structure de dossiers selon le plan recommandé
   - Développer le fichier principal du serveur
   - Créer le module de gestion des événements Socket.io
   - Mettre en place les premiers événements (connexion/déconnexion)

2. **Structure du Frontend** 🎨
   - Créer la structure de dossiers selon le plan recommandé
   - Développer les composants de base
   - Mettre en place le Context pour la gestion des états
   - Créer un hook personnalisé pour la connexion Socket.io

3. **Test de l'intégration** 🧪
   - Vérifier que les composants se connectent correctement au serveur
   - Tester la communication bidirectionnelle
   - Implémenter un système simple de connexion avec pseudonyme

> 💡 **Matériel de support** : Un checkpoint intermédiaire pour vérifier votre structure avant de continuer est disponible dans le fichier [help/etape04.md](help/etape04.md).

## Phase de restitution 👨‍🏫

En groupe, les étudiants partagent:
- Les structures mises en place et leur organisation
- Les difficultés rencontrées lors de l'implémentation (on sait tous qu'il y en aura! 😉)
- Les solutions trouvées pour les problèmes communs
- Démonstration des fonctionnalités de base implémentées

## Livrable 📦

À la fin de cette étape, chaque étudiant ou binôme doit avoir:

1. **Structure de projet complète** 🏢:
   - Backend: server.js, socketHandlers.js, structure de dossiers
   - Frontend: composants App, UserList, ChatInput (basiques)
   - Context et hook pour Socket.io

2. **Fonctionnalités de base** ⚡:
   - Connexion avec pseudonyme fonctionnelle
   - Affichage des utilisateurs connectés en temps réel
   - Mise à jour de la liste lors d'une connexion/déconnexion

## Validation du livrable ✅

Pour valider cette étape, le projet doit:
- Permettre à un utilisateur de se connecter avec un pseudonyme
- Afficher la liste des utilisateurs connectés
- Mettre à jour la liste en temps réel lorsqu'un utilisateur se connecte/déconnecte
- Suivre la structure de dossiers définie dans le plan d'architecture

> 💡 **Matériel de support** : Une section sur les points de blocage courants et leurs solutions est disponible dans le fichier [help/etape04.md](help/etape04.md).

## Ressources 📚
- [Patterns d'organisation de projets Node.js](https://github.com/goldbergyoni/nodebestpractices) - Pour coder comme un pro!
- [Architecture React recommandée](https://reactjs.org/docs/faq-structure.html) - Comment structurer votre app sans prise de tête
- [Hooks personnalisés React](https://reactjs.org/docs/hooks-custom.html) - Créez vos propres hooks et impressionnez vos amis! 🤩
- [Context API en détail](https://reactjs.org/docs/context.html) - Pour partager vos états sans passer par mille props
- [Gestion d'événements Socket.io](https://socket.io/docs/v4/server-instance/) - Tout ce qu'il faut savoir sur les événements

## Pour la prochaine session 🔮
- Avoir complété la structure de base du projet
- Implémenter la fonctionnalité de connexion avec pseudonyme
- Réfléchir à l'implémentation de la fonctionnalité d'envoi de messages 