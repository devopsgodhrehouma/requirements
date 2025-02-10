Below is a **simplified specification** focusing on **purchasing PDF files** and **accessing specific course content**. The emphasis is on **protected routes** that require a **logged-in user** with the correct **role** (e.g., `admin`, `instructor`, `student`) to perform or access specific actions.

---
# O1

# **Simple Backend Specification for PDF Purchases & Course Access**

## **1. User Roles**
1. **Admin**  
   - Can manage (create/update/delete) PDFs and courses.  
   - Can view all orders and handle refunds.

2. **Instructor**  
   - Can create and manage **courses** (but not necessarily PDFs).  
   - Can see enrolled students for their courses.

3. **Student**  
   - Can **buy PDFs** and **enroll** in accessible courses.  
   - Has read-only access to purchased PDFs or enrolled courses.

4. **Guest** (not logged in)  
   - Can only **view** public information (list of PDFs or courses).  
   - **Cannot purchase** or view protected content.

---

## **2. Authentication Endpoints**

All **Authentication** endpoints are **unprotected** (no token required to access them).

1. **POST** `/api/auth/register`  
   - **Description:** Register a new user account (defaults to `student` role).  
   - **Body (JSON):**  
     ```json
     {
       "email": "user@example.com",
       "password": "Password123!",
       "role": "student" 
     }
     ```
   - **Response:** `201 Created` on success or `400 Bad Request` on errors.

2. **POST** `/api/auth/login`  
   - **Description:** Login with valid credentials.  
   - **Body (JSON):**  
     ```json
     {
       "email": "user@example.com",
       "password": "Password123!"
     }
     ```
   - **Response:** Returns a **JWT** token (and optionally a refresh token).

3. **POST** `/api/auth/logout`  
   - **Description:** Invalidate the current user’s token (if using token blacklisting).  
   - **Headers:** Must include a valid `Authorization: Bearer <token>`.

4. **GET** `/api/auth/profile`  
   - **Description:** Retrieve the logged-in user’s profile.  
   - **Protected:** Must have a valid token in the headers.  
   - **Response Example:**  
     ```json
     {
       "id": "1234",
       "email": "user@example.com",
       "role": "student"
     }
     ```

---

## **3. PDF Management**

### **3.1 Admin Routes** 
(*Only `admin` role can access these.*)

1. **POST** `/api/pdfs`  
   - **Description:** Create a new PDF resource (upload or specify URL).  
   - **Protected:** Must have `admin` role.  
   - **Body (JSON):**  
     ```json
     {
       "title": "Intro to Programming",
       "description": "A beginner's PDF.",
       "price": 9.99,
       "file_url": "https://example.com/pdfs/intro_programming.pdf"
     }
     ```
   - **Response:** `201 Created` + JSON with PDF details.

2. **PUT** `/api/pdfs/:pdfId`  
   - **Description:** Update an existing PDF’s details.  
   - **Protected:** `admin` role only.  
   - **Body (JSON):** Any fields to update (e.g., `title`, `price`, `file_url`).  
   - **Response:** `200 OK` + updated PDF object or `404 Not Found`.

3. **DELETE** `/api/pdfs/:pdfId`  
   - **Description:** Delete a PDF resource.  
   - **Protected:** `admin` role only.  
   - **Response:** `204 No Content` or `404 Not Found`.

### **3.2 Public & Protected Routes**

1. **GET** `/api/pdfs`  
   - **Description:** List all available PDFs.  
   - **Public:** Anyone can view.  
   - **Response:** Array of PDFs with basic info (e.g., ID, title, price).

2. **GET** `/api/pdfs/:pdfId`  
   - **Description:** Get detailed information about a single PDF (title, description, price).  
   - **Public:** Anyone can view the **info**.  
   - **Response:** PDF details.

3. **GET** `/api/pdfs/:pdfId/download`  
   - **Description:** Download the PDF **file**.  
   - **Protected:** Must be logged in **and** must have **purchased** the PDF.  
   - **Response:** PDF file (or a 403 error if unauthorized).

---

## **4. Purchasing PDFs**

### **4.1 Create Order & Payment**

1. **POST** `/api/orders/pdfs`  
   - **Description:** Create a new order for one or multiple PDFs.  
   - **Protected:** Must be logged in (e.g., `student`).  
   - **Body (JSON):**  
     ```json
     {
       "pdfIds": ["pdf123", "pdf456"]
     }
     ```
   - **Response:** Returns an **order** in `pending` status with total amount.

2. **POST** `/api/payments/pdfs`  
   - **Description:** Complete payment for a PDF order (integrated with a payment gateway).  
   - **Protected:** Logged in user only.  
   - **Body (JSON):**  
     ```json
     {
       "orderId": "order_1234",
       "paymentMethod": "stripe"   // or "paypal"
     }
     ```
   - **Response:**  
     - On **success**, the system updates the order to `paid`, grants user access.  
     - Returns `200 OK` with payment confirmation.  
     - On **failure**, `400 Bad Request` or `402 Payment Required`.

3. **GET** `/api/orders/history`  
   - **Description:** Retrieve user’s past orders (PDF purchases).  
   - **Protected:** Must be the logged-in user.  
   - **Response:** Array of orders with status, total amount, item details.

---

## **5. Course Management**

### **5.1 Admin & Instructor Routes**

1. **POST** `/api/courses`  
   - **Description:** Create a new course.  
   - **Protected:** `admin` **or** `instructor` role.  
   - **Body (JSON):**  
     ```json
     {
       "title": "Intro to Web Development",
       "description": "Learn the basics of HTML, CSS, JS.",
       "price": 49.99
     }
     ```
   - **Response:** `201 Created` + new course data.

2. **PUT** `/api/courses/:courseId`  
   - **Description:** Update course details (title, description, price, etc.).  
   - **Protected:** `admin` or **the** `instructor` who created the course.  
   - **Response:** `200 OK` or `404 Not Found`.

3. **DELETE** `/api/courses/:courseId`  
   - **Description:** Remove or archive a course.  
   - **Protected:** `admin` only **or** the course’s instructor.  
   - **Response:** `204 No Content` or `404 Not Found`.

### **5.2 Public & Protected Routes**

1. **GET** `/api/courses`  
   - **Description:** List all courses (title, price, brief info).  
   - **Public:** Anyone can view.

2. **GET** `/api/courses/:courseId`  
   - **Description:** Get detailed info (modules, sections) of a single course.  
   - **Public:** Basic info visible to everyone, but **full content** only if purchased/enrolled (see below).

3. **POST** `/api/courses/:courseId/enroll`  
   - **Description:** Enroll in a course (free or paid).  
   - **Protected:** Must be logged in (`student`).  
   - **Behavior:**
     - If the course is free, enroll immediately.  
     - If the course is paid, create an **order** (like PDFs), then require payment.

4. **GET** `/api/courses/:courseId/content`  
   - **Description:** Fetch the actual course lessons, videos, or materials.  
   - **Protected:** User **must be enrolled** in the course to access.  
   - **Response:** If authorized, return full content; otherwise return `403 Forbidden`.

5. **GET** `/api/courses/:courseId/progress`  
   - **Description:** Retrieve the student’s progress in the course.  
   - **Protected:** Must be the **enrolled** student (or instructor for monitoring).

---

## **6. Role-Based Access Summary**

- **Admin** can do everything: manage PDFs, courses, orders.  
- **Instructor** can create and manage **their own** courses.  
- **Student** can **buy PDFs** and **enroll** in courses, **access** purchased PDFs and enrolled course content.  
- **Guest** can only view public listings (course/pdf listings, not detailed or paid content).

**Implementation Detail**:  
- Check **JWT** in request headers: `Authorization: Bearer <token>`.  
- A **middleware** or **guard** checks the user’s **role** and **ownership** (for example, only the instructor who created the course can update it, or an admin can do it as well).

---

## **7. Error Handling & Status Codes**

- **401 Unauthorized**: If no token or invalid token is provided for protected routes.  
- **403 Forbidden**: If the user’s role is not permitted to perform the action.  
- **404 Not Found**: If the resource (PDF, course, order) doesn’t exist.  
- **400 Bad Request**: Missing or invalid request body.  
- **500 Internal Server Error**: Unexpected server issues.

---

## **8. Deployment & Security Notes (Simplified)**

1. **JWT-Based Auth**  
   - Use **bcrypt** for password hashing.  
   - Store tokens client-side in a secure manner (e.g., HttpOnly cookies).

2. **API Gateway / Reverse Proxy**  
   - Optionally place behind **NGINX** or **AWS API Gateway** for SSL/TLS.

3. **Database**  
   - Keep a simple relational structure: `Users`, `PDFs`, `Courses`, `Orders`, `OrderItems`, `Enrollments`.

4. **Payment Gateway**  
   - Integrate **Stripe** or **PayPal** for secure transactions.  
   - Use **webhooks** to confirm payment success/failure.

5. **Monitoring**  
   - Log important actions (user login, purchase, enroll).  
   - Track errors via a service like **Sentry** or logs in **CloudWatch** (if on AWS).

---

## **9. Example Usage Flow**

1. **User Registration**  
   - `POST /api/auth/register` → `200 OK` + JWT tokens.

2. **User Buys a PDF**  
   - `POST /api/orders/pdfs` with `[pdfId]` → Creates `order` with status `pending`.  
   - `POST /api/payments/pdfs` → Processes payment, order moves to `paid`.  
   - **User** can now download the PDF at `GET /api/pdfs/:pdfId/download`.

3. **User Enrolls in a Paid Course**  
   - `POST /api/courses/:courseId/enroll` → Creates an order if it’s a paid course.  
   - `POST /api/payments/pdfs` or a similar `/api/payments/courses` endpoint → Payment success.  
   - **User** can then access lesson materials via `GET /api/courses/:courseId/content`.

---

### **That’s It!** 
This simplified specification should guide you in building **protected endpoints** where **only authenticated users** with the **appropriate role** can buy PDFs, download them, enroll in courses, and view course content. 

Use **middleware** to ensure:
- **Auth** is required on certain routes.
- **Role checks** are in place for admin/instructor vs. student vs. guest routes.

This minimal scope handles:
- **User login & registration**  
- **PDF purchase & download**  
- **Course creation, enrollment, and content access**  

…and sets the foundation for a basic yet secure system.



--------------

# DEEPSEEK

Voici une version simplifiée des **routes (endpoints)** pour un système d’achat de PDFs et d’accès à des cours. Les routes sont **protégées** par authentification (JWT) et **rôle-based access control (RBAC)**. Seuls les utilisateurs connectés avec le bon rôle peuvent accéder à certaines routes.

---

## **Routes Protégées pour l'Achat de PDFs et l'Accès aux Cours**

### **1. Authentification**
- **POST /api/auth/register**  
  - **Description** : Permet à un utilisateur de s’inscrire.  
  - **Body** :  
    ```json
    {
      "email": "user@example.com",
      "password": "password123",
      "role": "student" // ou "admin", "instructor"
    }
    ```
  - **Réponse** :  
    ```json
    {
      "message": "User registered successfully",
      "userId": "uuid"
    }
    ```

- **POST /api/auth/login**  
  - **Description** : Permet à un utilisateur de se connecter et de recevoir un JWT.  
  - **Body** :  
    ```json
    {
      "email": "user@example.com",
      "password": "password123"
    }
    ```
  - **Réponse** :  
    ```json
    {
      "token": "jwt_token",
      "role": "student" // ou "admin", "instructor"
    }
    ```

---

### **2. Gestion des PDFs (Accessible aux Admins et Instructeurs)**
- **POST /api/pdfs** (Admin/Instructor)  
  - **Description** : Permet à un admin ou un instructeur d’ajouter un nouveau PDF.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Body** :  
    ```json
    {
      "title": "Introduction to Programming",
      "description": "Learn the basics of programming.",
      "price": 29.99,
      "file_url": "https://example.com/pdf/intro.pdf"
    }
    ```
  - **Réponse** :  
    ```json
    {
      "message": "PDF added successfully",
      "pdfId": "uuid"
    }
    ```

- **GET /api/pdfs** (Tous les utilisateurs connectés)  
  - **Description** : Liste tous les PDFs disponibles.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Réponse** :  
    ```json
    [
      {
        "id": "uuid",
        "title": "Introduction to Programming",
        "description": "Learn the basics of programming.",
        "price": 29.99,
        "file_url": "https://example.com/pdf/intro.pdf"
      }
    ]
    ```

---

### **3. Achat de PDFs (Accessible aux Étudiants)**
- **POST /api/pdfs/purchase** (Student)  
  - **Description** : Permet à un étudiant d’acheter un PDF.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Body** :  
    ```json
    {
      "pdf_id": "uuid",
      "payment_method": "stripe" // ou "paypal"
    }
    ```
  - **Réponse** :  
    ```json
    {
      "message": "PDF purchased successfully",
      "orderId": "uuid"
    }
    ```

---

### **4. Accès aux PDFs Achetés (Accessible aux Étudiants)**
- **GET /api/pdfs/:id** (Student)  
  - **Description** : Permet à un étudiant d’accéder à un PDF qu’il a acheté.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Réponse** :  
    ```json
    {
      "id": "uuid",
      "title": "Introduction to Programming",
      "description": "Learn the basics of programming.",
      "file_url": "https://example.com/pdf/intro.pdf"
    }
    ```

---

### **5. Gestion des Cours (Accessible aux Admins et Instructeurs)**
- **POST /api/courses** (Admin/Instructor)  
  - **Description** : Permet à un admin ou un instructeur d’ajouter un nouveau cours.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Body** :  
    ```json
    {
      "title": "Advanced JavaScript",
      "description": "Master advanced JavaScript concepts.",
      "price": 49.99,
      "video_url": "https://example.com/videos/js.mp4"
    }
    ```
  - **Réponse** :  
    ```json
    {
      "message": "Course added successfully",
      "courseId": "uuid"
    }
    ```

- **GET /api/courses** (Tous les utilisateurs connectés)  
  - **Description** : Liste tous les cours disponibles.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Réponse** :  
    ```json
    [
      {
        "id": "uuid",
        "title": "Advanced JavaScript",
        "description": "Master advanced JavaScript concepts.",
        "price": 49.99,
        "video_url": "https://example.com/videos/js.mp4"
      }
    ]
    ```

---

### **6. Achat de Cours (Accessible aux Étudiants)**
- **POST /api/courses/purchase** (Student)  
  - **Description** : Permet à un étudiant d’acheter un cours.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Body** :  
    ```json
    {
      "course_id": "uuid",
      "payment_method": "stripe" // ou "paypal"
    }
    ```
  - **Réponse** :  
    ```json
    {
      "message": "Course purchased successfully",
      "orderId": "uuid"
    }
    ```

---

### **7. Accès aux Cours Achetés (Accessible aux Étudiants)**
- **GET /api/courses/:id** (Student)  
  - **Description** : Permet à un étudiant d’accéder à un cours qu’il a acheté.  
  - **Headers** : `Authorization: Bearer <jwt_token>`  
  - **Réponse** :  
    ```json
    {
      "id": "uuid",
      "title": "Advanced JavaScript",
      "description": "Master advanced JavaScript concepts.",
      "video_url": "https://example.com/videos/js.mp4"
    }
    ```

---

### **8. Rôles et Permissions**
- **Student** : Peut acheter et accéder aux PDFs et cours.  
- **Instructor** : Peut ajouter des PDFs et des cours.  
- **Admin** : Peut gérer tout le contenu (PDFs, cours, utilisateurs).  

---

### **Exemple de Middleware pour Vérifier les Rôles**
```javascript
const checkRole = (role) => (req, res, next) => {
  if (req.user.role !== role) {
    return res.status(403).json({ message: "Access denied" });
  }
  next();
};
```

---

### **Exemple d’Utilisation du Middleware**
```javascript
app.post("/api/pdfs", checkRole("admin"), (req, res) => {
  // Logique pour ajouter un PDF
});
```

---

Ce document fournit une structure claire et simple pour les routes protégées. Les développeurs peuvent l’utiliser pour implémenter rapidement le backend. 🚀
