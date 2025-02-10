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

### **📌 8. Exemple Gestion de l’Accès aux Cours Achetables à Vie**  

✅ Lorsqu’un utilisateur **achète un cours à vie**, il doit voir **tous les cours qu’il a achetés** sur **sa page personnelle**.  
✅ L’API backend doit exposer un **endpoint** qui retourne la **liste des cours achetés par l’utilisateur**.  
✅ Docusaurus doit appeler cet endpoint pour afficher **uniquement les cours disponibles pour cet utilisateur**.  

---

## **🔑 Endpoints Concernés**
| **Méthode** | **Route** | **Description** | **Protection** |
|------------|---------|----------------------|--------------|
| `GET` | `/api/auth/me` | Retourne les infos utilisateur | **JWT obligatoire** |
| `GET` | `/api/user/courses` | Retourne la liste des cours achetés par l’utilisateur | **JWT obligatoire** |

---

## **📌 8.1. Explication du Endpoint `/api/user/courses`**
### ✅ **Objectif**
Ce endpoint retourne **uniquement les cours que l’utilisateur a achetés à vie**.  

### ✅ **Données retournées (Exemple de Réponse)**
Lorsque **Docusaurus** appelle `/api/user/courses`, le backend retourne une **liste des cours achetés** :
```json
{
  "courses": [
    {
      "id": 1,
      "title": "Ansible pour les Débutants",
      "category": "Ansible",
      "access": "lifetime"
    },
    {
      "id": 2,
      "title": "Docker et Kubernetes",
      "category": "Docker",
      "access": "lifetime"
    }
  ]
}
```
👉 **Seuls les cours achetés apparaissent ici.**  
👉 L’utilisateur **ne verra pas** les cours qu’il n’a pas achetés.  

---

## **📌 8.2. Intégration Frontend Docusaurus**
Docusaurus doit appeler `/api/user/courses` et afficher **seulement les cours achetés** dans **sa page personnelle**.  

Dans **`src/pages/my-courses.js`** :
```jsx
import React, { useEffect, useState } from 'react';

export default function MyCourses() {
    const [courses, setCourses] = useState([]);

    useEffect(() => {
        fetch('/api/user/courses', {
            headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
        })
        .then(res => res.json())
        .then(data => setCourses(data.courses));
    }, []);

    return (
        <div>
            <h1>Mes Cours</h1>
            {courses.length === 0 ? (
                <p>Vous n'avez pas encore acheté de cours.</p>
            ) : (
                <ul>
                    {courses.map((course) => (
                        <li key={course.id}>
                            <a href={`/docs/category/${course.category}`}>{course.title}</a>
                        </li>
                    ))}
                </ul>
            )}
        </div>
    );
}
```
👉 **Chaque cours acheté est affiché avec un lien direct vers sa documentation.**  
👉 **Si l’utilisateur n’a aucun cours, un message est affiché.**  

---

# 10 - IMPORTANT À LIRE (POUR MOHSEN)

### 📜 **Sécurisation des Cours dans Docusaurus avec TypeScript et Spring Boot**  

Dans ce document, nous allons détailler **comment protéger l’accès aux cours** dans **Docusaurus (TypeScript)** en utilisant un **backend Spring Boot** pour gérer l’authentification et les autorisations.

---

## **📌 1. Objectif**
Nous devons implémenter un **système de protection des cours** pour :  
1️⃣ Vérifier si l’utilisateur **a acheté un cours** avant d’y accéder.  
2️⃣ **Rediriger l’utilisateur** vers la page d’abonnement (`/subscribe`) s’il n’a pas accès.  
3️⃣ **Afficher dynamiquement** les cours achetés sur la page **"Mes Cours"**.  

---

## **📌 2. Fonctionnement Général**
| **Fonctionnalité** | **Docusaurus (Frontend - TypeScript)** | **Spring Boot (Backend)** |
|----------------|----------------|----------------|
| **Afficher les cours achetés sur la page utilisateur** | ✅ Appelle `/api/user/courses` et affiche les résultats | ✅ Retourne la liste des cours achetés |
| **Bloquer l’accès à un cours non acheté** | ✅ Vérifie l’accès et redirige vers `/subscribe` | ✅ Vérifie l’achat dans la base de données |
| **Gérer l’authentification (JWT)** | ✅ Stocke le token JWT et l’envoie avec chaque requête | ✅ Vérifie le JWT et renvoie les données utilisateur |
| **Gérer les paiements Stripe** | ✅ Redirige l’utilisateur vers Stripe via `/api/payment/checkout/{courseId}` | ✅ Gère la validation et active l’accès après paiement |

---

## **📌 3. Intégration Frontend dans Docusaurus (TypeScript)**
### ✅ **1. Création d’un Composant de Protection des Routes (`ProtectedRoutes.tsx`)**
Ce composant **vérifie l’accès** et redirige si nécessaire.

📂 **Fichier :** `src/components/ProtectedRoutes.tsx`
```tsx
import React, { useEffect, useState } from 'react';
import { useHistory } from '@docusaurus/router';

interface ProtectedRoutesProps {
  course: string;
}

const ProtectedRoutes: React.FC<ProtectedRoutesProps> = ({ course }) => {
  const history = useHistory();
  const [authorized, setAuthorized] = useState<boolean | null>(null);

  useEffect(() => {
    const fetchCourses = async () => {
      const token = localStorage.getItem('token');
      if (!token) {
        history.push('/login');
        return;
      }

      try {
        const response = await fetch('/api/user/courses', {
          headers: { 'Authorization': `Bearer ${token}` },
        });

        if (!response.ok) {
          history.push('/subscribe');
          return;
        }

        const data = await response.json();
        const hasAccess = data.courses.some((c: { category: string }) => c.category === course);
        setAuthorized(hasAccess);

        if (!hasAccess) {
          history.push('/subscribe');
        }
      } catch (error) {
        console.error("Erreur lors de la récupération des cours", error);
        history.push('/subscribe');
      }
    };

    fetchCourses();
  }, [course, history]);

  return authorized === null ? <p>Chargement...</p> : null;
};

export default ProtectedRoutes;
```
👉 **Ce composant vérifie les droits d’accès et redirige vers `/subscribe` si l’utilisateur n’a pas acheté le cours.**  

---

### ✅ **2. Utilisation de `ProtectedRoutes` dans une Page de Cours**
Pour **protéger l’accès aux cours**, nous devons **intégrer** ce composant dans les pages de documentation.

📂 **Fichier :** `src/pages/docs/category/ansible.tsx`
```tsx
import React from 'react';
import ProtectedRoutes from '@site/src/components/ProtectedRoutes';

const AnsibleDocs: React.FC = () => {
  return (
    <div>
      <ProtectedRoutes course="ansible" />
      <h1>Bienvenue dans le cours Ansible</h1>
      <p>Contenu du cours ici...</p>
    </div>
  );
};

export default AnsibleDocs;
```
👉 **Si l’utilisateur n’a pas acheté le cours "Ansible", il sera redirigé vers `/subscribe`.**  

---

### ✅ **3. Affichage des Cours Achetés dans la Page "Mes Cours"**
Nous devons afficher dynamiquement **les cours achetés** sur la page `/my-courses`.

📂 **Fichier :** `src/pages/my-courses.tsx`
```tsx
import React, { useEffect, useState } from 'react';

interface Course {
  id: number;
  title: string;
  category: string;
  access: string;
}

const MyCourses: React.FC = () => {
  const [courses, setCourses] = useState<Course[]>([]);

  useEffect(() => {
    const fetchCourses = async () => {
      const token = localStorage.getItem('token');
      if (!token) return;

      const response = await fetch('/api/user/courses', {
        headers: { 'Authorization': `Bearer ${token}` },
      });

      if (response.ok) {
        const data = await response.json();
        setCourses(data.courses);
      }
    };

    fetchCourses();
  }, []);

  return (
    <div>
      <h1>Mes Cours</h1>
      {courses.length === 0 ? (
        <p>Vous n'avez pas encore acheté de cours.</p>
      ) : (
        <ul>
          {courses.map((course) => (
            <li key={course.id}>
              <a href={`/docs/category/${course.category}`}>{course.title}</a>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};

export default MyCourses;
```
👉 **Cette page affiche uniquement les cours achetés.**

---

## **📌 4. Implémentation du Backend Spring Boot**
Le backend doit fournir **une API REST sécurisée** qui retourne les cours achetés.

📂 **Fichier :** `UserCourseController.java`
```java
@RestController
@RequestMapping("/api/user")
public class UserCourseController {

    @Autowired
    private UserService userService;

    @GetMapping("/courses")
    public ResponseEntity<?> getUserCourses(@RequestHeader("Authorization") String token) {
        User user = userService.getUserFromToken(token);
        List<Course> courses = userService.getPurchasedCourses(user.getId());

        return ResponseEntity.ok(Map.of("courses", courses));
    }
}
```
👉 **Le backend vérifie l’utilisateur via son token et retourne ses cours achetés.**  

---

## **📌 5. Résumé et Étapes Suivantes**
✅ **Docusaurus (TypeScript) protège l’accès aux cours avec `ProtectedRoutes`.**  
✅ **Spring Boot fournit `/api/user/courses` pour récupérer les cours achetés.**  
✅ **Si un utilisateur non connecté ou non autorisé tente d’accéder à un cours, il est redirigé vers `/subscribe`.**  
✅ **La page `/my-courses` affiche dynamiquement les cours achetés.**  

🔥 **Une fois que ton développeur backend implémente `/api/user/courses`, tu peux finaliser l’intégration Docusaurus !** 🚀  


## **📌 8.3. Sécurisation dans Docusaurus**
**Si un utilisateur tente d’accéder à `/docs/category/ansible` sans avoir acheté le cours :**  
1️⃣ **Docusaurus interroge `/api/user/courses`** pour voir s’il a accès.  
2️⃣ **Si le cours n’est pas dans sa liste, redirection vers `/subscribe`**.  

Dans **`protectedRoutes.js`** :
```js
import { useEffect, useState } from 'react';
import { useHistory } from '@docusaurus/router';

export default function ProtectedRoutes({ course }) {
    const history = useHistory();
    const [authorized, setAuthorized] = useState(false);

    useEffect(() => {
        fetch('/api/user/courses', {
            headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
        })
        .then(res => res.json())
        .then(data => {
            const hasAccess = data.courses.some(c => c.category === course);
            if (!hasAccess) {
                history.push('/subscribe'); // Rediriger si l’utilisateur n’a pas accès
            } else {
                setAuthorized(true);
            }
        });
    }, []);

    return authorized ? null : <p>Chargement...</p>;
}
```
👉 **Ce composant protège les pages `/docs/*`.**  
👉 **Si l’utilisateur n’a pas acheté le cours, il est redirigé vers `/subscribe`.**  

---

## **📌 8.4. Plan de Validation**
| **Cas de Test** | **Attendu** |
|---------------|------------|
| L’utilisateur appelle `/api/user/courses` sans JWT | **Erreur 401 Unauthorized** |
| L’utilisateur appelle `/api/user/courses` avec un JWT valide | **Liste des cours achetés** |
| L’utilisateur accède à `/docs/category/ansible` sans achat | **Redirection `/subscribe`** |
| L’utilisateur accède à `/docs/category/ansible` après achat | **Accès autorisé** |

---

## **📌 8.5. Résumé et Étapes Suivantes**
✅ **Le backend expose `/api/user/courses` pour retourner les cours achetés.**  
✅ **Docusaurus affiche uniquement les cours achetés sur `/my-courses`.**  
✅ **Les routes `/docs/category/*` sont protégées et redirigent vers `/subscribe` si l’utilisateur n’a pas accès.**  

🔥 **Une fois cette API en place, je pourrai finaliser l’intégration Docusaurus !** 🚀  
💬 **Dis-moi si tu veux ajuster encore quelque chose !**





---

## **📌 9. Conclusion et Étapes Suivantes**
🔹 **Le frontend (Docusaurus) appelle le backend pour gérer l'accès aux cours.**  
🔹 **Le backend protège les endpoints et vérifie les abonnements.**  
🔹 **Docusaurus redirige vers `/subscribe` si l’accès est refusé.**  
