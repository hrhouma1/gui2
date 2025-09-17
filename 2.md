# Document de présentation – Projets Intégrateurs d’Applications Mobiles

## Introduction

Ce document présente cinq projets intégrateurs conçus et développés dans un cadre académique et professionnel. Chaque projet répond à un besoin concret en matière d’éducation, de productivité, de gestion ou de services communautaires. L’objectif est de démontrer la pertinence des approches retenues, la cohérence technologique et la valeur ajoutée pour les usagers.

Chaque projet est décrit selon trois axes :

1. **Description et objectifs**
2. **Fonctionnalités principales**
3. **Pile technologique recommandée et plan de réalisation**



## 1. Sweatly – Application de mise en forme et suivi de progression

### Description et objectifs

Sweatly est une application mobile de conditionnement physique qui vise à rendre l’entraînement accessible, motivant et durable. Elle accompagne les utilisateurs dans leurs séances grâce à une bibliothèque d’exercices, des programmes personnalisés et un suivi détaillé de la progression.

### Fonctionnalités principales

* Onboarding personnalisé (objectifs, niveau, matériel disponible).
* Bibliothèque d’exercices avec images et démonstrations.
* Workouts prédéfinis et programmes hebdomadaires.
* Suivi des séances (séries, répétitions, charges, temps).
* Statistiques de progression, streaks et badges.
* Notifications de rappel et relances en cas d’inactivité.
* Partage des résultats sous forme de résumé graphique.

### Pile technologique recommandée

* **Mobile** : Flutter (GoRouter, Riverpod, Hive pour cache hors ligne).
* **Backend** : Firebase (Auth, Firestore, Storage, Cloud Functions, FCM).
* **CI/CD** : GitHub Actions ou Codemagic (Android/iOS).
* **Analytics** : Firebase Analytics et Crashlytics.



## 2. Shiftly – Gestion des horaires, du pointage et des fiches de paie

### Description et objectifs

Shiftly est une application de gestion des quarts de travail destinée aux employeurs et employés. Elle facilite la planification, le pointage, la génération de rapports et l’accès sécurisé aux fiches de paie. L’accent est mis sur la transparence, la précision et la protection des données.

### Fonctionnalités principales

* Authentification et profils (employé, gestionnaire).
* Création, assignation et modification des quarts.
* Pointage par GPS, QR code ou photo horodatée.
* Consultation et téléchargement des fiches de paie.
* Exportation des heures travaillées (CSV, PDF).
* Notifications (rappel de quart, changements urgents).
* Gestion du consentement et des données personnelles.

### Pile technologique recommandée

* **Mobile** : Flutter (geolocator, mobile\_scanner, local\_auth).
* **Backend** : Supabase (Postgres + PostGIS, Auth, Edge Functions) comme source de vérité, combiné avec Firestore pour cache temps réel.
* **Storage** : Supabase Storage ou Firebase Storage pour PDF et preuves de pointage.
* **Notifications** : FCM.



## 3. PlaneEx – Agenda intelligent et gestion des conflits

### Description et objectifs

PlaneEx est une application d’agenda enrichi permettant aux utilisateurs d’organiser efficacement leurs activités. Elle propose une gestion intelligente des évènements, détecte les conflits d’horaire et intègre des données contextuelles (météo, jours fériés, trafic).

### Fonctionnalités principales

* Création, édition et suppression d’évènements.
* Vue calendrier (jour, semaine, mois).
* Rappels programmés et notifications automatiques.
* Détection des conflits et proposition d’alternatives.
* Intégration météo, jours fériés et trafic/ETA.
* Export ICS/PDF pour partage d’agenda.
* Confidentialité configurable (public, privé, partagé).

### Pile technologique recommandée

* **Mobile** : Flutter (table\_calendar, Riverpod, GoRouter, http).
* **Backend** : Firebase (Auth, Firestore, Cloud Functions, FCM).
* **APIs externes** : Google Maps (Distance Matrix), OpenWeather, Holiday API.
* **Analytics** : Firebase Analytics et Crashlytics.



## 4. DevLingo – Plateforme éducative pour l’apprentissage des langages informatiques

### Description et objectifs

DevLingo est une application éducative destinée aux étudiants et passionnés de programmation. Elle vise à rendre l’apprentissage des langages informatiques progressif, motivant et interactif grâce à des cours, quiz et systèmes de récompenses.

### Fonctionnalités principales

* Authentification et choix du niveau (débutant, intermédiaire, avancé).
* Tableau de bord avec progression, points et badges.
* Cours interactifs avec exemples de code et illustrations.
* Quiz de validation avec correction immédiate.
* Révisions automatiques des concepts difficiles.
* Défis et classements régionaux.
* Personnalisation des préférences et horaires de rappel.

### Pile technologique recommandée

* **Mobile** : Flutter (GoRouter, Riverpod, Hive pour cache hors ligne).
* **Backend** : Firebase (Auth, Firestore, Storage, Cloud Functions, FCM).
* **Admin console (optionnel)** : Next.js sur Vercel pour gestion du contenu.


## 5. GazGo – Application communautaire pour les prix de l’essence

### Description et objectifs

GazGo est une application collaborative permettant de consulter et de partager les prix de l’essence en temps réel à Montréal. Elle repose sur la contribution des usagers et vise à réduire les coûts de déplacement des automobilistes en facilitant le choix de la station la plus avantageuse.

### Fonctionnalités principales

* Carte interactive Google Maps avec stations-service et prix.
* Consultation en mode invité ou connexion avec compte.
* Ajout et modification des prix par les utilisateurs connectés.
* Validation communautaire des prix (thumbs up / correction).
* Attribution de points et classement de fiabilité des contributeurs.
* Filtres avancés : type de carburant, prix, rayon de recherche.
* Estimation du coût d’un plein selon le véhicule et le carburant choisi.

### Pile technologique recommandée

* **Mobile** : Flutter (google\_maps\_flutter, geolocator, Riverpod).
* **Backend** : Supabase (Postgres + PostGIS pour requêtes géospatiales, Auth, Edge Functions).
* **Storage** : Supabase Storage pour photos de preuves de prix.
* **APIs externes** : Google Maps, Directions et Distance Matrix.



## Conclusion

Ces cinq projets démontrent une approche moderne de développement d’applications mobiles combinant **Flutter** côté client et des solutions **cloud-first** côté serveur (Firebase ou Supabase). Chaque application répond à un besoin précis (santé, travail, organisation, éducation, consommation) et s’appuie sur des fonctionnalités éprouvées (authentification sécurisée, notifications, intégrations API).

La planification Agile en sprints de trois semaines assure une progression incrémentale, la livraison de versions testables et l’ajustement continu selon les besoins. Ce cadre méthodologique et technologique garantit la faisabilité, la robustesse et l’impact potentiel de ces projets dans des contextes académiques, professionnels et citoyens.
