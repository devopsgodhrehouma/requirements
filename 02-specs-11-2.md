----
# Partie 1
----


Voici un **tableau comparatif** entre la protection des routes côté **Docusaurus (frontend)** et **Spring Boot (backend)** en prenant en compte plusieurs critères, notamment la **sécurité**, la **flexibilité**, et la **gestion des accès selon le paiement de l'utilisateur**.

| **Critères**            | **Protection côté Docusaurus (Frontend)** | **Protection côté Spring Boot (Backend)** |
|-------------------------|-----------------------------------------|------------------------------------------|
| **Sécurité**            | ❌ **Faible** - Une protection côté frontend peut être contournée via les outils de développement du navigateur. | ✅ **Élevée** - Le backend valide les accès indépendamment du client. Impossible à contourner via le navigateur. |
| **Gestion des rôles**    | ⚠️ **Limitée** - Possible via un stockage local (`localStorage`, `sessionStorage`), mais facilement manipulable. | ✅ **Robuste** - Gestion des rôles directement dans la base de données et validation via JWT ou OAuth2. |
| **Protection contre les attaques** | ❌ **Vulnérable** - Expose le frontend aux attaques (manipulation du code JavaScript, vol de jetons) | ✅ **Sécurisé** - Le backend contrôle toutes les requêtes et applique des règles strictes (CSRF, CORS, validation JWT). |
| **Facilité d’implémentation** | ✅ **Facile** - Un simple `useEffect` ou `useContext` pour bloquer l’accès aux routes privées. | ⚠️ **Moyennement complexe** - Nécessite une API d’authentification et un middleware de sécurité (Spring Security). |
| **Scalabilité** | ⚠️ **Moyenne** - Convient aux petites applications, mais peu robuste pour des centaines/milliers d’utilisateurs. | ✅ **Haute** - Adapté aux applications avec beaucoup d’utilisateurs et gestion centralisée des droits. |
| **Contrôle des abonnements/paiements** | ❌ **Non sécurisé** - L’utilisateur peut modifier ses données locales pour accéder à du contenu payant. | ✅ **Sécurisé** - La vérification des paiements se fait côté serveur via une base de données fiable. |
| **Facilité de maintenance** | ✅ **Simple** - Pas besoin de requêtes serveur pour vérifier l'accès. | ⚠️ **Plus complexe** - Nécessite une mise à jour régulière de la base de données et des endpoints d’authentification. |
| **Compatibilité avec Docusaurus** | ✅ **Facile** - Peut être implémenté directement via React (`useAuth`, `useEffect`). | ✅ **Possible** - Utilisation d’un proxy ou middleware pour restreindre l'accès aux documents privés. |
| **Exposition des données sensibles** | ❌ **Oui** - Le contenu premium peut être visible dans le code source si mal implémenté. | ✅ **Non** - Aucune donnée confidentielle n’est envoyée tant que l’authentification n’est pas validée. |

### **Conclusion : Quelle approche choisir ?**
- **Option recommandée : Protection des routes côté Spring Boot (Backend)**
  - ✅ Sécurise les accès aux contenus premium.
  - ✅ Impossibilité de contourner les restrictions via le navigateur.
  - ✅ Gestion centralisée des paiements et des rôles utilisateurs.
  - ⚠️ **Seul inconvénient** : nécessite une authentification via une API et un middleware de validation.

- **Option alternative (complémentaire) : Une protection partielle côté Docusaurus**
  - 🔹 Peut être utilisée pour une **expérience utilisateur fluide** (ex. masquer les boutons premium pour les non-abonnés).
  - 🔹 **Ne doit pas être la seule protection** car elle peut être contournée.

### **Solution idéale**
1. **Backend (Spring Boot) :** Vérifier les paiements et restreindre l’accès aux ressources privées via JWT ou OAuth2.
2. **Frontend (Docusaurus) :** Mettre en place un système d’affichage conditionnel pour éviter d’afficher les options premium aux non-abonnés.
3. **Sécurité avancée :** Protéger les requêtes avec des tokens JWT signés et un contrôle strict des sessions.

---
# Partie 2
----

### **Scénario : Protection des Routes Premium pour un Site Docusaurus avec un Backend Spring Boot**  

#### **Contexte :**
- **Frontend :** Site statique généré avec **Docusaurus**.
- **Backend :** API REST **Spring Boot** qui gère l'authentification et l'accès au contenu premium.
- **Objectif :** Restreindre l’accès aux articles premium selon l’abonnement de l’utilisateur.

---

## **🔹 Scénario d'Utilisation : Accès à un Article Premium**
### **Acteurs :**
- **Utilisateur Non Abonné (Gratuit)**
- **Utilisateur Premium (Payant)**
- **Serveur Backend (Spring Boot)**
- **Site Frontend (Docusaurus)**

### **1️⃣ Un utilisateur non connecté essaie d’accéder à une page premium**
- Il clique sur une page **premium** dans Docusaurus.
- Frontend vérifie s’il a un token JWT valide.
- ✅ **S’il n’a pas de token** → Redirection vers la page d’abonnement.

---

### **2️⃣ Un utilisateur connecté mais non premium essaie d’accéder à une page premium**
- Il est connecté avec un **compte gratuit**.
- Frontend envoie son token **JWT** au backend pour vérifier son rôle.
- Backend répond **"403 Forbidden"** car l’utilisateur n’a pas de compte premium.
- ✅ **Affichage d’un message** : "Ce contenu est réservé aux abonnés premium. Abonnez-vous ici."

---

### **3️⃣ Un utilisateur premium accède à une page premium**
- Il est **connecté** et possède un abonnement actif.
- Il clique sur une page premium.
- **Docusaurus envoie une requête au backend avec son token JWT**.
- Backend vérifie :
  - 🔹 Si le token JWT est valide.
  - 🔹 Si l’utilisateur a un **rôle premium**.
  - 🔹 Si son abonnement est toujours actif.
- ✅ Réponse **200 OK** → Le contenu premium est chargé.

---

### **4️⃣ Un utilisateur premium dont l'abonnement a expiré essaie d'accéder à un article premium**
- Son JWT est encore valide, mais son abonnement a expiré.
- Il tente d’accéder à un article premium.
- Backend vérifie dans la base de données → Abonnement **expiré**.
- ❌ **Retourne 403 Forbidden avec message :** "Votre abonnement a expiré, veuillez le renouveler."
- ✅ **Redirection vers une page de renouvellement d’abonnement.**

---

## **🛠️ Mise en Place Technique**
### **🔹 1. Backend Spring Boot : Protéger les routes avec JWT et Spring Security**
#### **1️⃣ Modèle d’Utilisateur avec abonnement**
```java
@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;
    private String password;
    private String role; // "FREE" ou "PREMIUM"
    private LocalDate subscriptionEndDate;

    // Vérifie si l'abonnement est actif
    public boolean isPremium() {
        return role.equals("PREMIUM") && subscriptionEndDate.isAfter(LocalDate.now());
    }
}
```

#### **2️⃣ Middleware de Vérification des Utilisateurs Premium**
```java
@Component
public class PremiumUserFilter extends OncePerRequestFilter {
    @Autowired private JwtService jwtService;
    @Autowired private UserRepository userRepository;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        String token = jwtService.extractToken(request);
        if (token != null) {
            String email = jwtService.extractUsername(token);
            User user = userRepository.findByEmail(email);

            if (user == null || !user.isPremium()) {
                response.sendError(HttpServletResponse.SC_FORBIDDEN, "Abonnement requis !");
                return;
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

#### **3️⃣ Sécurisation des Routes Premium**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Autowired private JwtAuthFilter jwtAuthFilter;
    @Autowired private PremiumUserFilter premiumUserFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf().disable()
            .authorizeHttpRequests()
            .requestMatchers("/premium/**").authenticated() // Vérifie l'authentification
            .requestMatchers("/premium/**").addFilterBefore(premiumUserFilter, UsernamePasswordAuthenticationFilter.class)
            .anyRequest().permitAll()
            .and().build();
    }
}
```

---

### **🔹 2. Frontend Docusaurus : Protection des Pages**
#### **1️⃣ Vérifier l’abonnement avant d’afficher le contenu**
Dans **Docusaurus**, on peut utiliser un `useEffect()` pour **vérifier l’accès avant d’afficher une page**.

```tsx
import React, { useEffect, useState } from "react";
import { useNavigate } from "react-router-dom";

const PremiumPage = () => {
    const [isPremium, setIsPremium] = useState(false);
    const navigate = useNavigate();

    useEffect(() => {
        const token = localStorage.getItem("jwt");
        if (!token) {
            navigate("/subscribe"); // Redirige si non connecté
        } else {
            fetch("/api/user/check-premium", {
                headers: { Authorization: `Bearer ${token}` }
            })
            .then(res => res.json())
            .then(data => {
                if (!data.premium) {
                    navigate("/subscribe"); // Redirige si pas premium
                } else {
                    setIsPremium(true);
                }
            })
            .catch(() => navigate("/subscribe"));
        }
    }, []);

    return isPremium ? <div>🎉 Contenu Premium ici !</div> : <p>Chargement...</p>;
};

export default PremiumPage;
```

---

## **🎯 Conclusion**
| **Critères**                   | **Frontend Docusaurus** | **Backend Spring Boot** |
|--------------------------------|----------------------|----------------------|
| **Protection faible (facile à contourner)** | ❌ Oui | ✅ Non |
| **Sécurité des accès premium** | ❌ Faible | ✅ Sécurisé (base de données + JWT) |
| **Facilité d’intégration** | ✅ Facile (React Hooks) | ⚠️ Moyennement complexe (Spring Security) |
| **Vérification des abonnements** | ❌ Non sécurisé | ✅ Vérification en base de données |
| **Gestion des sessions utilisateurs** | ⚠️ Stockage local (`localStorage`) | ✅ JWT sécurisé côté serveur |
| **Protection des fichiers/documents premium** | ❌ Facile à contourner | ✅ Impossible sans authentification |

### **📌 Meilleure Solution :**
1. **Protéger les routes sensibles côté backend avec Spring Security + JWT.**
2. **Ajouter une vérification côté frontend pour améliorer l’expérience utilisateur.**
3. **Utiliser un middleware de filtrage (`PremiumUserFilter`) pour empêcher l’accès non autorisé aux routes premium.**
4. **Rediriger les utilisateurs non premium vers une page d’abonnement via Docusaurus.**

👉 **Tu veux que je te fasse un guide plus détaillé avec un tutoriel complet sur l’intégration JWT dans Spring Boot et Docusaurus ?** 🚀
