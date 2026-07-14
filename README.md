# Travel Memory Deployment on AWS

## Project Overview

This project demonstrates the deployment of the **Travel Memory MERN application** on AWS using:

- Amazon EC2
- MongoDB Atlas
- NGINX Reverse Proxy
- Amazon Machine Images (AMI)
- Application Load Balancer (ALB)

The application architecture consists of separate Frontend and Backend servers, with NGINX configured as a reverse proxy and Load Balancers providing high availability.

---

# Architecture

```text
                    User
                      |
          Frontend Load Balancer (ALB)
                      |
          -----------------------------
          |                           |
     Frontend EC2-1              Frontend EC2-2
          |                           |
          -----------------------------
                      |
           Backend Load Balancer (ALB)
                      |
          -----------------------------
          |                           |
      Backend EC2-1               Backend EC2-2
                      |
                MongoDB Atlas
```

---

# AWS Resources Used

| Resource | Purpose |
|----------|---------|
| EC2 | Host Frontend & Backend |
| Security Groups | Allow required ports |
| MongoDB Atlas | Database |
| NGINX | Reverse Proxy |
| AMI | Create additional instances |
| Target Groups | Register EC2 instances |
| Application Load Balancer | Distribute traffic |

---

# Security Group Configuration

| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP (NGINX) |
| 3000 | TCP | React Frontend |
| 3001 | TCP | Node.js Backend |

---

# Backend Deployment

## Install Node.js and npm

```bash
sudo apt update
sudo apt install nodejs
sudo apt install npm
```

Verify installation.

```bash
node -v
npm -v
```

Example Output

```text
v22.22.1
9.2.0
```

---

## Clone Repository

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
```

Navigate to backend.

```bash
cd TravelMemory/backend
```

---

## Configure Environment Variables

Create `.env`

```env
MONGO_URI='mongodb+srv://<username>:<password>@<cluster>.mongodb.net/?appName=<app-name>'
PORT=3001
```

---

## Install Dependencies

```bash
npm install
```

---

## Start Backend

```bash
node index.js
```

Backend will run on

```text
http://<Backend-Public-IP>:3001
```

---

# Frontend Deployment

Navigate to frontend.

```bash
cd ~/TravelMemory/frontend
```

Create `.env`

```env
REACT_APP_BACKEND_URL=http://<Backend-Public-IP>:3001
```

Install dependencies.

```bash
npm install
```

Start frontend.

```bash
npm start
```

Application will be available at

```text
http://<Frontend-Public-IP>:3000
```

---

# Configure NGINX on Backend

Install NGINX.

```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

Edit configuration.

```bash
sudo nano /etc/nginx/sites-available/default
```

Paste the following configuration.

```nginx
server {
    listen 80;
    server_name <Backend-Public-IP>;

    location / {
        proxy_pass http://127.0.0.1:3001;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Validate configuration.

```bash
sudo nginx -t
```

Restart NGINX.

```bash
sudo systemctl restart nginx
```

Run backend.

```bash
node index.js
```

Backend is now accessible without specifying port **3001**.

Example

```text
http://<Backend-Public-IP>/trip
```

---

# Configure NGINX on Frontend

Install NGINX.

```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

Edit configuration.

```bash
sudo nano /etc/nginx/sites-available/default
```

Paste the following configuration.

```nginx
server {
    listen 80;
    server_name <Frontend-Public-IP>;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Validate configuration.

```bash
sudo nginx -t
```

Restart NGINX.

```bash
sudo systemctl restart nginx
```

Run frontend.

```bash
npm start
```

Application is now accessible without specifying port **3000**.

```text
http://<Frontend-Public-IP>
```

---

# Create AMIs

Create Amazon Machine Images (AMI) for:

- Backend EC2 Instance
- Frontend EC2 Instance

These AMIs are used to launch additional identical instances.

---

# Launch Additional EC2 Instances

Launch:

- Backend Instance 2
- Frontend Instance 2

using the AMIs.

If the public IP addresses change, update the `server_name` in the NGINX configuration.

---

# Create Target Groups

## Backend Target Group

Register:

- Backend EC2 Instance 1
- Backend EC2 Instance 2

---

## Frontend Target Group

Register:

- Frontend EC2 Instance 1
- Frontend EC2 Instance 2

Verify that all instances pass health checks.

---

# Create Application Load Balancers

Create two Application Load Balancers.

## Backend ALB

Forward traffic to the Backend Target Group.

---

## Frontend ALB

Forward traffic to the Frontend Target Group.

---

# Testing

Access the application using the Frontend Load Balancer DNS.

```text
http://<Frontend-ALB-DNS>
```

Verify the following:

- Frontend loads successfully
- Backend API is reachable
- MongoDB Atlas connection is successful
- CRUD operations work correctly
- NGINX reverse proxy works as expected
- Traffic is distributed across multiple EC2 instances

---

# Tech Stack

- React
- Node.js
- Express.js
- MongoDB Atlas
- NGINX
- Git
- GitHub
- AWS EC2
- Amazon Machine Images (AMI)
- Application Load Balancer (ALB)

---

# Deployment Flow

```text
Developer
     │
     ▼
GitHub Repository
     │
     ▼
Clone Repository on EC2
     │
     ▼
Backend EC2  ─────────────► MongoDB Atlas
     ▲
     │
Frontend EC2
     ▲
     │
NGINX Reverse Proxy
     ▲
     │
Application Load Balancer
     ▲
     │
Users
```

---

# Conclusion

The Travel Memory MERN application was successfully deployed on AWS using separate Frontend and Backend EC2 instances. NGINX was configured as a reverse proxy to expose the application over HTTP without requiring application port numbers. Amazon Machine Images (AMIs) were used to create additional Frontend and Backend instances, which were registered with Application Load Balancers through target groups. The application was tested successfully, demonstrating end-to-end connectivity between the Frontend, Backend, and MongoDB Atlas, while providing scalability and high availability through load balancing.
