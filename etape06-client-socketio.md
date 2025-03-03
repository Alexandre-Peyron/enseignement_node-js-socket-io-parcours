# Étape 6 : Connexion Client Socket.io et Tests de Communication 📱🔌

## Objectifs pédagogiques 🎯
- Implémenter un client Socket.io avec React
- Créer un hook personnalisé pour la gestion des connexions
- Mettre en place un Context pour partager l'état de la connexion
- Tester la communication bidirectionnelle entre client et serveur

## Présentation initiale 🎬

### Architecture du client Socket.io avec React 🏗️
- Intégration de Socket.io dans une application React
- Patterns de gestion d'état pour les connexions en temps réel
- Hooks personnalisés vs Context API pour Socket.io (fight! 🥊)
- Cycle de vie des connexions Socket.io dans React

> 💡 **Matériel de support** : Un exemple complet du Context et des hooks personnalisés pour Socket.io est disponible dans le fichier [help/etape06.md](help/etape06.md).

### Composants clés à implémenter 🧩
- Création du Context pour le chat
- Implémentation du hook personnalisé `useSocket`
- Gestion des événements Socket.io côté client
- Gestion des erreurs et de la reconnexion

## Travail en autonomie 💪

Les étudiants travaillent en autonomie ou en binôme pour développer le client Socket.io:

1. **Implémentation du Context** 🌍
   - Développer le fichier `ChatContext.jsx`
   - Mettre en place les états pour la connexion, les utilisateurs et les messages
   - Configurer la connexion Socket.io
   - Implémenter les fonctions pour interagir avec le serveur

2. **Création du hook personnalisé (optionnel)** 🎣
   - Développer un hook `useSocket.js` pour encapsuler la logique de connexion
   - Implémenter les méthodes emit et on
   - Gérer le cycle de vie de la connexion Socket.io

3. **Test de la connexion** 🧪
   - Créer un composant de test pour vérifier l'état de la connexion
   - Tester la réception des messages du serveur
   - Vérifier la mise à jour des listes d'utilisateurs
   - Tester l'envoi et la réception de messages (ça marche? ça marche pas? Le suspense! 😅)

> 💡 **Matériel de support** : Un guide complet de débogage des problèmes de connexion client/serveur est disponible dans le fichier [help/etape06.md](help/etape06.md).

## Phase de restitution 👨‍🏫

En groupe, les étudiants partagent:
- Leurs implémentations du Context et/ou des hooks
- Les stratégies de gestion d'état choisies
- Les difficultés rencontrées et leurs solutions
- Démonstration des fonctionnalités client avec le serveur

## Livrable 📦

À la fin de cette étape, chaque étudiant ou binôme doit avoir:

1. **Fichier `ChatContext.jsx` complet** 📄 incluant:
   - Initialisation de la connexion Socket.io
   - Gestion des états (connexion, utilisateurs, messages)
   - Méthodes pour interagir avec le serveur (join, sendMessage)
   - Gestion des erreurs

2. **Composant de test fonctionnel** 🧪 montrant:
   - État de la connexion
   - Liste des utilisateurs connectés
   - Formulaire simple pour rejoindre le chat
   - Interface basique pour envoyer un message

## Validation du livrable ✅

Pour valider cette étape, le client doit pouvoir:
- Se connecter au serveur Socket.io
- Permettre à l'utilisateur de rejoindre le chat avec un pseudonyme
- Afficher les erreurs (ex: pseudonyme déjà utilisé - "Désolé, ce nom est déjà pris! 🙈")
- Afficher la liste des utilisateurs connectés
- Envoyer et recevoir des messages simples

> 💡 **Matériel de support** : Consultez la section "Meilleures pratiques" dans le fichier [help/etape06.md](help/etape06.md) pour des conseils sur l'implémentation optimale du client Socket.io.

## Ressources 📚
- [Documentation Socket.io Client](https://socket.io/docs/v4/client-api/) - La bible du client Socket.io!
- [Hooks React](https://reactjs.org/docs/hooks-overview.html) - Tout comprendre sur les hooks
- [React Context API](https://reactjs.org/docs/context.html) - Pour éviter le "props drilling" 👷‍♂️
- [Gestion des effets dans React](https://reactjs.org/docs/hooks-effect.html) - Maîtrisez useEffect comme un pro!
- [Patterns de conception React](https://kentcdodds.com/blog/application-state-management-with-react) - Des astuces d'experts pour votre app 