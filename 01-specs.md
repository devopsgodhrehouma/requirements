### **📜 Cahier des Charges : Plateforme de Vente de Cours et PDFs**  
📆 **Date :** [À compléter]  
📍 **Responsable :** [Ton nom]  
👨‍💻 **Développeur Backend :** [Nom du dev Spring Boot]  
🌍 **Technologies utilisées :**  
- **Frontend :** Docusaurus  
- **Backend :** Spring Boot  
- **Base de données :** MySQL/PostgreSQL  
- **Paiements :** Stripe  
- **Stockage des PDFs :** AWS S3  

---

## **📌 1. Objectif du Projet**  
Le projet consiste à développer une **plateforme de vente de cours et de PDFs** où :  
✅ **Docusaurus** gère l'affichage du contenu et la documentation.  
✅ **Spring Boot** gère l'authentification, les paiements et la protection des contenus.  
✅ **Stripe** permet le paiement par abonnement ou par achat individuel.  

L'objectif est de **sécuriser l'accès aux cours** selon l’abonnement ou les achats de l’utilisateur.  

---

## **📌 2. Fonctionnalités Générales**  
| **Fonctionnalité** | **Détails** |
|-----------------|-----------|
| 🔐 **Authentification** | Connexion, inscription, JWT, gestion des rôles (`STANDARD_USER`, `MONTHLY_USER`, `PREMIUM_USER`). |
| 🛒 **Gestion des achats** | L’utilisateur peut acheter un cours à l’unité ou prendre un abonnement mensuel/annuel. |
| 📂 **Accès aux cours** | L’utilisateur peut accéder aux cours qu’il a achetés ou à tous s’il a un abonnement annuel. |
| 💳 **Paiement en ligne** | Intégration avec **Stripe** (achat unique ou abonnement). |
| 🔄 **Vérification des accès** | Un utilisateur qui tente d'accéder à un cours qu’il n’a pas payé est redirigé vers `/subscribe`. |
| 📥 **Téléchargement de PDFs** | Les PDFs sont stockés sur **AWS S3** et accessibles après paiement. |
| 🎛 **Espace utilisateur** | L’utilisateur peut voir son historique d’achats et gérer son abonnement. |

---

## **📌 3. Gestion des Accès et Abonnements**  
### **🔑 1. Règles d’Accès aux Cours**  
| **Type d’utilisateur** | **Accès** |
|-----------------|------------|
| **`STANDARD_USER`** | Peut accéder uniquement aux cours qu’il a achetés. |
| **`MONTHLY_USER`** | Accès aux cours du mois en cours. |
| **`PREMIUM_USER`** | Accès illimité à tous les cours. |

### **🔗 2. Cas d’Utilisation**
| **Cas** | **Action** |
|--------|----------|
| Un utilisateur non connecté essaie d’accéder à un cours. | Redirigé vers `/login`. |
| Un utilisateur tente d’accéder à un cours qu’il n’a pas acheté. | Redirigé vers `/subscribe`. |
| Un utilisateur avec abonnement annuel accède à n’importe quel cours. | Accès autorisé. |
| Un utilisateur ayant acheté un cours accède à une leçon de ce cours. | Accès autorisé. |
| Un utilisateur avec un abonnement mensuel tente d’accéder à un cours non inclus ce mois-ci. | Redirigé vers `/subscribe`. |

---

## **📌 4. Routes API Backend**
🔹 **Le backend (Spring Boot) expose des endpoints pour gérer l’authentification, les achats et l’accès aux cours.**  

### **🔑 1. Routes d’Authentification**
| **Méthode** | **Route** | **Description** | **Protection** |
|------------|---------|---------------|--------------|
| `POST` | `/api/auth/login` | Connexion utilisateur | **Public** |
| `POST` | `/api/auth/register` | Inscription utilisateur | **Public** |
| `GET` | `/api/auth/me` | Récupérer les infos de l’utilisateur connecté | **JWT obligatoire** |

---

### **📚 2. Routes des Cours et PDFs**
| **Méthode** | **Route** | **Description** | **Protection** |
|------------|---------|---------------|--------------|
| `GET` | `/api/courses/list` | Liste des cours disponibles | **Public** |
| `GET` | `/api/courses/my-courses` | Liste des cours achetés par l’utilisateur | **JWT obligatoire** |
| `GET` | `/api/courses/{courseId}` | Détails d’un cours | **JWT obligatoire + Vérification d’achat** |
| `GET` | `/api/pdf/download/{pdfId}` | Télécharger un PDF | **JWT obligatoire + Vérification d’achat** |

---

### **💳 3. Routes des Paiements**
| **Méthode** | **Route** | **Description** | **Protection** |
|------------|---------|---------------|--------------|
| `POST` | `/api/payment/checkout/{courseId}` | Lancer un paiement | **JWT obligatoire** |
| `GET` | `/api/payment/verify/{transactionId}` | Vérifier un paiement | **JWT obligatoire** |

---

### **🔐 5. Gestion des Droits dans Docusaurus**
Docusaurus doit appeler l’API `/api/auth/me` au chargement du site pour savoir **si l’utilisateur est connecté et quel est son abonnement**.  

Si un utilisateur tente d’accéder à une route `/docs/ansible/*`, Docusaurus doit :  
1️⃣ Vérifier si l’utilisateur est connecté.  
2️⃣ Vérifier `/api/courses/my-courses` pour voir s’il a accès à ce cours.  
3️⃣ Sinon, **le rediriger vers `/subscribe`**.  

---

## **📌 5. Plan de Déploiement**
| **Composant** | **Technologie** | **Hébergement** |
|--------------|---------------|----------------|
| **Frontend** | Docusaurus | AWS Amplify |
| **Backend** | Spring Boot | AWS EC2 / Railway |
| **Base de Données** | MySQL / PostgreSQL | AWS RDS |
| **Stockage PDFs** | AWS S3 | AWS S3 |
| **Paiements** | Stripe | Stripe API |

---

## **📌 6. Tâches à Répartir**
| **Tâche** | **Responsable** | **Statut** |
|-----------|--------------|---------|
| Intégration JWT et Spring Security | **Dev Backend** | ⏳ En cours |
| Configuration des endpoints d’authentification | **Dev Backend** | ✅ Terminé |
| Protection des cours selon l’abonnement | **Dev Backend** | 🚀 À faire |
| Intégration API dans Docusaurus | **Dev Frontend** | ⏳ En cours |
| Gestion des paiements Stripe | **Dev Backend** | 🚀 À faire |
| Déploiement Docusaurus sur AWS Amplify | **Dev Frontend** | ✅ Terminé |
| Hébergement du backend sur AWS EC2 | **Dev Backend** | 🔄 En test |

---

## **📌 7. Tests et Validation**
| **Cas de test** | **Attendu** | **Résultat** |
|---------------|------------|-------------|
| Un utilisateur non connecté tente d’accéder à `/docs/ansible/*`. | Redirigé vers `/login`. | ✅ OK |
| Un utilisateur ayant acheté un cours accède à sa documentation. | Accès autorisé. | ✅ OK |
| Un utilisateur non premium essaie d’accéder à un cours non acheté. | Redirigé vers `/subscribe`. | ✅ OK |
| Un utilisateur premium essaie d’accéder à n’importe quel cours. | Accès autorisé. | ✅ OK |

---

## **📌 8. Conclusion et Étapes Suivantes**
🔹 **Le frontend (Docusaurus) appelle le backend pour gérer l'accès aux cours.**  
🔹 **Le backend protège les endpoints et vérifie les abonnements.**  
🔹 **Docusaurus redirige vers `/subscribe` si l’accès est refusé.**  
