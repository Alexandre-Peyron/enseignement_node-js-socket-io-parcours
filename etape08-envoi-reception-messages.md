# Étape 8 : Envoi et Réception de Messages 💬📩

## Objectifs pédagogiques 🎯
- Développer les fonctionnalités d'envoi et de réception de messages
- Implémenter différents types de messages (utilisateur, système)
- Créer un composant d'affichage des messages
- Mettre en place le système d'indicateur de frappe

## Présentation initiale 🎬

### Système de messagerie avec Socket.io 📨
- Principes de l'envoi et de la réception de messages
- Structure optimale pour les objets message
- Diffusion des messages à tous les utilisateurs
- Gestion des messages système et utilisateur

### Composants de messagerie 🏗️
- Structure du composant ChatList pour afficher les messages
- Conception du composant ChatInput pour l'envoi
- Styles et mise en page pour les messages
- Gestion des types de messages et des métadonnées

## Travail en autonomie 💪

Les étudiants travaillent en autonomie ou en binôme sur:

1. **Implémentation des événements de messagerie** 📡
   - Compléter les gestionnaires d'événements `send_message` côté serveur
   - Gérer la réception des messages côté client
   - Implémenter les indicateurs de frappe (pour voir qui tape frénétiquement ⌨️)
   - Ajouter des validations pour les messages (pas de messages vides, on n'est pas là pour rigoler... enfin si, mais pas comme ça! 😄)

2. **Création des composants de messagerie** 🎨
   - Développer le composant ChatList pour afficher les messages
   - Créer le composant ChatInput pour l'envoi
   - Styler les différents types de messages
   - Implémenter l'autoscroll vers les nouveaux messages (parce que scroller manuellement, c'est tellement 2010! 🙄)

3. **Tests et optimisations** 🧪
   - Tester l'envoi et la réception de messages
   - Vérifier les indicateurs de frappe
   - Tester avec plusieurs utilisateurs simultanés
   - Optimiser la performance pour un grand nombre de messages (pas de lag, on n'est pas en 1995! ⚡)

## Phase de restitution 👨‍🏫

En groupe, les étudiants partagent:
- Leurs implémentations des composants de messagerie
- Les différentes approches pour les styles de messages
- Les stratégies d'optimisation utilisées
- Les difficultés rencontrées et leurs solutions

## Livrable 📦

À la fin de cette étape, chaque étudiant ou binôme doit avoir:

1. **Composants de messagerie complets** 📱:
   - ChatList.jsx pour l'affichage des messages
   - ChatInput.jsx pour l'envoi de messages
   - Styles CSS pour tous les types de messages

2. **Fonctionnalités de messagerie opérationnelles** 🔥:
   - Envoi et réception de messages en temps réel
   - Affichage des messages système
   - Indicateurs de frappe
   - Scroll automatique vers les nouveaux messages

## Validation du livrable ✅

Pour valider cette étape, l'application doit:
- Permettre l'envoi et la réception de messages en temps réel
- Afficher correctement les différents types de messages
- Mettre à jour l'interface utilisateur en temps réel
- Afficher les indicateurs de frappe
- Présenter une interface utilisateur soignée pour les messages

## Ressources 📚
- [Emission d'événements Socket.io](https://socket.io/docs/v4/emit-cheatsheet/) - Pour devenir un maître des événements 🧙‍♂️
- [Gestion des événements React](https://reactjs.org/docs/handling-events.html) - Domptez ces events React!
- [CSS Flexbox pour la mise en page](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) - Flexbox est votre ami fidèle
- [Référence au DOM avec useRef](https://reactjs.org/docs/hooks-reference.html#useref) - Pour cibler ces éléments comme un pro 🎯
- [Optimisation des performances React](https://reactjs.org/docs/optimizing-performance.html) - Parce qu'une app rapide, c'est une app qu'on aime! 