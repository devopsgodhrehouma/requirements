# **User Flow Scenarios for E-learning & E-commerce Platform**  

This document outlines various **user flows** for key functionalities in the system, including **course access, purchases, refunds, and account management**. Each scenario describes the **expected user journey**, **system interactions**, and **success/failure paths**.

---

## **1. User Wants to Access a Course**
### **Actors**: 
- **Student** (Registered user)
- **System** (Backend & Frontend)
- **Payment Gateway** (Stripe, PayPal)

### **Preconditions**:  
- The user must have an active account.  
- The course must be available for purchase or already owned.  
- If the course is paid, the user must have completed the payment.  

### **Flow Steps**:  
#### ✅ **Scenario 1: Accessing a Purchased Course**
1. User logs into their account.
2. User navigates to the **Dashboard**.
3. System fetches and displays the **list of purchased courses**.
4. User selects a course and clicks **"Start Course"**.
5. System verifies the **user’s access rights**.
6. System loads the course content (videos, assignments, materials).  
7. User interacts with the course.

✅ **Success Condition**:  
- The user successfully accesses the course and begins learning.

❌ **Failure Conditions**:
- **Course Not Found** → Show error **"Course is unavailable"**.
- **Access Denied** → Redirect user to **"Purchase Page"**.
- **System Error** → Show message **"An error occurred. Try again later"**.

#### ✅ **Scenario 2: Purchasing a Course**
1. User logs in.
2. User browses the **Course Catalog**.
3. User selects a course and clicks **"Enroll"**.
4. System checks if the course is **free** or **paid**.
   - If free → Auto-enroll and redirect to the course.
   - If paid → Redirect to **Checkout Page**.
5. User selects **payment method** and completes the transaction.
6. Payment Gateway processes the payment.
7. System **validates** the payment and grants **access** to the course.
8. User receives a **confirmation email**.
9. User can now access the course from the **Dashboard**.

✅ **Success Condition**:
- Payment is successful, and the course is accessible.

❌ **Failure Conditions**:
- **Payment Failed** → Show error **"Transaction unsuccessful"**.
- **Invalid Card Details** → Request a **new payment method**.
- **Insufficient Balance** → Ask user to **add funds**.
- **Session Expired** → Restart the checkout process.

---

## **2. User Wants to Request a Refund**
### **Actors**: 
- **Student**  
- **Support Team**  
- **Payment Gateway**  

### **Preconditions**:  
- The user must have purchased a course.  
- Refund must be within the **allowed timeframe** (e.g., **14 days**).  
- User must not have completed more than **30% of the course content** (if applicable).  

### **Flow Steps**:
1. User logs into their **Account**.
2. User navigates to **"My Orders"** or **"Purchase History"**.
3. User selects a course and clicks **"Request Refund"**.
4. System checks:
   - Is the refund **eligible** (based on date & progress)?
   - Does the payment gateway **support automatic refunds**?
5. If **eligible**, system processes the refund:
   - Sends a request to **Payment Gateway (Stripe, PayPal)**.
   - Marks the course as **revoked** in the database.
   - Notifies the **Admin Team**.
6. If **manual review** is needed, the request is sent to **Support**.
7. System sends **email confirmation** to the user.
8. Refund amount is credited back to the **original payment method** within **5-7 business days**.

✅ **Success Condition**:
- Refund is processed successfully, and the user is notified.

❌ **Failure Conditions**:
- **Refund Period Expired** → Display message **"Refund not eligible"**.
- **Too Much Course Progress** → Deny refund.
- **Payment Gateway Error** → Show message **"Refund processing failed. Contact support."**.
- **Unauthorized User** → Deny refund request.

---

## **3. User Wants to Reset Password**
### **Actors**:  
- **User (Student or Instructor)**  
- **System (Backend & Email Service)**  

### **Preconditions**:  
- The user must have a registered account.  

### **Flow Steps**:
1. User clicks **"Forgot Password?"** on the login page.
2. System asks for **email input**.
3. User enters their registered **email**.
4. System checks if the email exists:
   - ✅ If valid → System sends **password reset link** via email.
   - ❌ If invalid → Show **"Email not found"** error.
5. User clicks the **password reset link** in their email.
6. System verifies the **reset token**.
7. User enters a **new password** and confirms.
8. System updates the password and redirects the user to the **login page**.

✅ **Success Condition**:
- Password is successfully updated, and user can log in.

❌ **Failure Conditions**:
- **Email Not Found** → Display message **"This email is not registered"**.
- **Invalid Reset Token** → Ask user to **request a new link**.
- **Weak Password** → Reject and ask for a stronger password.

---

## **4. User Wants to Update Account Information**
### **Actors**:  
- **User (Student or Instructor)**  
- **System**  

### **Preconditions**:  
- The user must be logged in.  

### **Flow Steps**:
1. User navigates to **"Profile Settings"**.
2. User updates fields (e.g., **name, email, profile picture, bio**).
3. User clicks **"Save Changes"**.
4. System **validates the input**:
   - ✅ If valid → Updates profile in the database.
   - ❌ If invalid → Shows error message.
5. If email is changed, system sends a **confirmation email**.
6. User logs out and logs in again to confirm the update.

✅ **Success Condition**:
- Profile updates are saved, and changes are reflected.

❌ **Failure Conditions**:
- **Invalid Email Format** → Show error message.
- **Duplicate Email** → Reject the update.
- **Session Expired** → Ask user to re-login.

---

## **5. User Wants to Leave a Course Review**
### **Actors**:  
- **Student**  
- **System**  

### **Preconditions**:  
- The user must have completed at least **50% of the course**.  
- The course must still be available.  

### **Flow Steps**:
1. User navigates to **"My Courses"**.
2. Selects a course and clicks **"Write a Review"**.
3. User enters a **rating (1-5 stars)** and a **comment**.
4. System **validates** the input:
   - ✅ If valid → Saves review in the database.
   - ❌ If invalid → Shows error message.
5. System updates the **course rating**.
6. Review appears in the **public course page**.

✅ **Success Condition**:
- Review is published successfully.

❌ **Failure Conditions**:
- **User Not Eligible** → Display message **"You need to complete at least 50% of the course"**.
- **Spam Detected** → Reject review.

---

## **Conclusion**
These user flows ensure a **seamless, user-friendly experience** for students and instructors. Each scenario is designed to handle **success and failure cases gracefully**, keeping the platform **secure, scalable, and reliable**. 🚀
