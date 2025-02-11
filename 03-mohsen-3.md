### **📌 Workflow : Accès à une Documentation Premium `/docs/ansible-complete`**  

#### **1️⃣ L'utilisateur tente d’accéder à `/docs/ansible-complete`**  
- **S’il n’est pas connecté** → Redirection vers `/login`.  
- **S’il est connecté**, son token JWT est envoyé au backend.  

#### **2️⃣ Backend Spring Boot vérifie l’accès**  
- Vérifie si le token JWT est valide.  
- Vérifie dans la base de données si l’utilisateur est **premium** et si son abonnement est **actif**.  

#### **3️⃣ Deux cas possibles :**  
✅ **Utilisateur premium** → Le backend retourne une réponse **200 OK** et autorise l’accès.  
❌ **Utilisateur gratuit ou abonnement expiré** → Réponse **403 Forbidden**, redirection vers `/subscribe`.  

#### **4️⃣ Expérience utilisateur côté Docusaurus**  
- **Utilisateur premium** : Contenu de la documentation premium s'affiche normalement.  
- **Utilisateur non premium** : Un message apparaît → "Ce contenu est réservé aux abonnés premium. Abonnez-vous ici."  

### **🎯 Résumé**  
🔹 **Le frontend détecte l’accès premium mais ne protège pas directement** (expérience utilisateur).  
🔹 **Le backend valide et bloque réellement l’accès** (sécurité renforcée).  
🔹 **Tout utilisateur non premium est redirigé vers une offre d’abonnement.**  

👉 **Besoin d’un schéma illustrant ce workflow ?** 🚀
