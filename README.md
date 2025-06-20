# lost-and-found

# Lost & Found UIUC

Built on **Node.js** (v18) in AWS Lambda functions, this service uses the full JavaScript stack end-to-end: React on the frontend, and Node.js handlers on the backend to power API Gateway, DynamoDB access, S3 image uploads, authentication (JWT+bcrypt), and SNS-driven email notifications.

### ScreenShots
![image](https://github.com/user-attachments/assets/bf86ca78-0539-4afc-9570-38c604d92f36)
![image](https://github.com/user-attachments/assets/603e47e2-8f8a-4c22-a157-6284a6e53ec2)
![image](https://github.com/user-attachments/assets/58b02fa3-c9c2-4d44-a8c6-1c4cb73d1804)
![image](https://github.com/user-attachments/assets/0a6a4814-1bc6-4e47-9d27-6fc4c466af7d)
![image](https://github.com/user-attachments/assets/75ae0280-4067-4009-ac78-881307160a38)
![image](https://github.com/user-attachments/assets/b6b1f21b-d470-46b4-9f16-c50738bb8a45)
![image](https://github.com/user-attachments/assets/982b0658-8ac9-4569-95ee-8b9bc20e0196)

---

### Flow of the Website
1. User visits the site (React frontend)
Tech: React

What happens:

User lands on the Home page.

React Router renders the Home component with “Get Started” button.

2. “Get Started” logic
Tech: React Router, localStorage

Flow:

User clicks Get Started.

If there is no token in localStorage, React navigates to /login.

Else, navigates directly to /lost (the “Report Lost” form).

3. Sign Up
Route: /signup

Component: Signup.jsx

Form fields: Email, Password, Confirm Password

On submit:

Client calls POST /signup via our Axios instance.

Backend (Node.js AWS Lambda):

Validates “password === confirm” on the frontend.

Hashes password with bcrypt.

Stores { email, passwordHash } in DynamoDB (Users table).

Automatically calls SNS.subscribe to your SNS topic (ARN from ALERT_TOPIC env var).

Returns 201 Created + message “check your email to confirm notifications.”

Result: User sees success banner, is redirected to /login.

4. Login
Route: /login

Component: Login.jsx

Form fields: Email, Password

On submit:

Client calls POST /login.

Backend:

Looks up user in DynamoDB (Users table) by email.

Verifies password with bcrypt.compare.

Issues a JWT signed with your JWT_SECRET.

Client stores JWT in localStorage and redirects to /browse.

5. Report Lost / Report Found
Routes: /lost and /found

Components: ReportLost.jsx / ReportFound.jsx

Form fields: Title, Category, Location, Description, Image upload

Image handling:

On file selection, frontend reads file as Base64 and includes imageBase64 in the payload.

On submit:

Client calls POST /lost or POST /found (via the same Axios instance, which injects the JWT in Authorization: Bearer … header).

Backend (Node.js Lambda):

Verifies JWT (extracts email from the token).

Writes the Base64 image to S3 under lost/<uuid>.png or found/<uuid>.png.

Saves an item in DynamoDB (LostItems or FoundItems table) with:

js
Copy
Edit
{ id, owner: email, title, category, location, description, imageUrl, createdAt }
Returns 201 Created + { id, imageUrl }.

Result: The new report appears for that user in /browse.

6. Browse Items
Route: /browse

Component: Browse.jsx

On mount:

Calls GET /items.

Backend:

Verifies JWT.

Checks if sub === 'admin@example.com' → isAdmin.

Scans both LostItems and FoundItems tables.

If admin, returns all items; otherwise filters to only those where owner === sub.

Frontend stores lostItems and foundItems in state.

7. Mark Solved (Delete)
UI: “Mark Solved” button on each card.

Tech: React event handler → deleteItem(id, type)

Client: calls DELETE /items with { id, type } in the body.

Backend:

Verifies JWT & extracts email.

Fetches the item by id from the appropriate table.

If owner !== email and not admin → 403 Forbidden.

Otherwise calls DynamoDB.delete.

Result: Frontend removes the card from state—no confirmation dialog.

8. Notify Owner (Admin Only)
UI: “Notify Owner” button appears only on Lost Items cards when isAdmin === true.

Client: calls POST /notify { id, type }.

Backend:

Verifies JWT & ensures sub === 'admin@example.com'.

Fetches item by id.

Publishes an SNS.publish to ALERT_TOPIC with a custom subject and message (“Hi… Something similar to your lost item has been found…”).

Result: SNS fans out an email to all confirmed subscribers for that topic.

Note: The user must first confirm their subscription (click the link in the SNS confirmation email) before they’ll receive notifications.

9. AWS Resources & Permissions
API Gateway (HTTP API) with CORS enabled (*) for dev.

Lambda functions (Node.js 18.x) for each route: createLost, createFound, listItems, deleteItem, signup, login, notifyOwner.

DynamoDB: three tables

LostItems

FoundItems

Users

S3 bucket for images, with PutObject and GetObject permissions from your Lambda role.

SNS topic (ALERT_TOPIC) for notifications:

Lambdas can Subscribe and Publish to this topic.

10. Frontend Stack
React (v18) with React Router for SPA navigation.

Axios instance that:

Points at your API Gateway base URL.

Automatically injects Authorization: Bearer <JWT> on each request.

Component hierarchy:

App.jsx: sets up <BrowserRouter> and a <Navbar> that shows/hides links based on login state.

Pages: Home, Signup, Login, ReportLost, ReportFound, Browse.

Services: /src/services/api.js exports all HTTP calls.

Security & Best Practices
Passwords hashed with bcrypt before storage.

JWT used for stateless auth; expires after 1 hour.

Environment variables for secrets and resource ARNs (never check these directly into Git).

IAM roles scoped narrowly—each Lambda has only the permissions it needs (least privilege).

Future Enhancements
Lock down CORS to your real domain in production.

Swap Node.js 18 → Node.js 20 (before Sept 2025) to keep runtime supported.

Add image-matching logic (e.g. Rekognition) for automated suggestions.

Improve UI/UX with loading spinners, pagination, and richer error handling.

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






