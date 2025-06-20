# lost-and-found

# Lost & Found UIUC

A simple campus-wide Lost & Found service built with React on the frontend and AWS Serverless (API Gateway, Lambda, DynamoDB, S3, SNS) on the backend. Users can report lost or found items; an admin can browse all reports, mark them solved (deletes), or notify the owner via email through SNS.

---

## 📦 Tech Stack

- **Frontend**  
  - React 18 + React Router  
  - Axios for HTTP + automatic JWT injection  
  - CSS modules / custom CSS (no Tailwind)
- **Backend**  
  - AWS Lambda (Node.js 18.x) via Serverless Framework  
  - API Gateway (HTTP API + CORS)  
  - DynamoDB (LostItems, FoundItems, Users tables)  
  - S3 for storing item images  
  - SNS topic for “notify owner” emails  
  - JWT (jsonwebtoken) + bcryptjs for auth

---

## 🚀 Features

- **User**  
  - Sign up & confirm subscription to notifications  
  - Login to receive a JWT  
  - Report a lost item (with optional photo)  
  - Report a found item (with optional photo)  
  - Browse *only your* lost & found reports  
  - Mark your reports “solved” (deletes them)  
- **Admin**  
  - Browse **all** reports  
  - Mark *any* report solved  
  - “Notify Owner” → sends an SNS‐driven email to the original reporter  
- **Images**  
  - Uploaded to S3 under `lost/` or `found/` prefixes  
  - Publicly served via S3 object URLs  
- **Notifications**  
  - On sign-up → auto-subscribe users’ emails to the SNS topic (confirmation email)  
  - On admin “Notify Owner” → publishes SNS message with a custom subject & body

---

## 🏗️ Architecture

