# lost-and-found

# Lost & Found UIUC

Built on **Node.js** (v18) in AWS Lambda functions, this service uses the full JavaScript stack end-to-end: React on the frontend, and Node.js handlers on the backend to power API Gateway, DynamoDB access, S3 image uploads, authentication (JWT+bcrypt), and SNS-driven email notifications.


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
React App
↕ axios + JWT header
API Gateway (HTTP API + CORS)
↕
Lambda handlers (createLost, createFound, listItems, deleteItem, signup, login, notifyOwner)
↕
DynamoDB (LostItems / FoundItems / Users)
↕
S3 (image upload & hosting)
↕
SNS Topic (LostFoundAlerts) → email subscriber


---

## 🔧 Local Development

### Prerequisites

- Node.js ≥ 18  
- AWS CLI configured (`aws configure`)  
- Serverless Framework (`npm i -g serverless`)  
- A `.env` file in **both** `/backend` and `/frontend` roots (see below)

### Environment Variables

Create an `env.example` in each folder:

```bash
# /backend/.env
LOST_TABLE=LostItems
FOUND_TABLE=FoundItems
USERS_TABLE=Users
IMAGE_BUCKET=lostfound-saas-<your-bucket>
ALERT_TOPIC
JWT_SECRET=<your-jwt-secret>

# /frontend/.env
REACT_APP_API_BASE=

# Backend
cd backend
npm install
# deploy your AWS infra & Lambdas
serverless deploy

# Frontend
cd ../frontend
npm install
npm start
# visits http://localhost:3000

### Project Structure
/
├── backend/
│   ├── handler.js          
│   ├── serverless.yml      
│   ├── package.json
│   └── .env.example
└── frontend/
    ├── src/
    │   ├── pages/          # React pages: Home, Signup, Login, ReportLost, ReportFound, Browse
    │   ├── services/api.js # Axios instance + exported helper functions
    │   ├── components/     # Navbar, form components, etc.
    │   └── styles/         # CSS modules / .css files
    ├── package.json
    └── .env.example

![image](https://github.com/user-attachments/assets/bf86ca78-0539-4afc-9570-38c604d92f36)
![image](https://github.com/user-attachments/assets/603e47e2-8f8a-4c22-a157-6284a6e53ec2)
![image](https://github.com/user-attachments/assets/58b02fa3-c9c2-4d44-a8c6-1c4cb73d1804)
![image](https://github.com/user-attachments/assets/0a6a4814-1bc6-4e47-9d27-6fc4c466af7d)
![image](https://github.com/user-attachments/assets/75ae0280-4067-4009-ac78-881307160a38)
![image](https://github.com/user-attachments/assets/b6b1f21b-d470-46b4-9f16-c50738bb8a45)
![image](https://github.com/user-attachments/assets/982b0658-8ac9-4569-95ee-8b9bc20e0196)






