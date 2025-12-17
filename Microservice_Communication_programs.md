# Cloud Native Applications - Complete Beginner's Guide

A comprehensive tutorial series for learning microservices architecture and Docker containerization through hands-on projects.

## Table of Contents

- [Week 1-2: Microservices Communication](#week-1-microservices-communication)
- [Week 3: Docker Compose Multi-Container Application](#week-3-docker-compose-multi-container-application)

---

## Week 1-2: Microservices Communication

### Prerequisites to Install

1. **Node.js** (JavaScript runtime)
   - Download from: https://nodejs.org/
   - Choose the LTS (Long Term Support) version
   - Verify installation:
     ```bash
     node --version
     npm --version
     ```
   - You should see version numbers (e.g., v18.17.0 and 9.6.7)

2. **VS Code** (Code editor)
   - Download from: https://code.visualstudio.com/
   - Free and beginner-friendly

3. **Postman** (For testing APIs)
   - Download from: https://www.postman.com/downloads/
   - Or use the web version

### What We're Building

A simple e-commerce system with 3 separate services:
- **Product Service**: Manages product information (like a catalog)
- **Order Service**: Handles customer orders
- **Notification Service**: Sends notifications when orders are placed

**Important Concept**: Each service is a separate program that runs independently. They communicate with each other over the network (just like how your browser talks to websites).

---

### Step 1: Setting Up the Project Structure

#### 1.1 Create Project Folders

Open your terminal (Mac/Linux) or Command Prompt (Windows) and run:

```bash
# Create main project folder
mkdir ecommerce-microservices
cd ecommerce-microservices

# Create folders for each service
mkdir product-service
mkdir order-service
mkdir notification-service
```

Your folder structure should look like:
```
ecommerce-microservices/
├── product-service/
├── order-service/
└── notification-service/
```

---

### Step 2: Building the Product Service

#### 2.1 Initialize the Product Service

```bash
cd product-service
npm init -y
```

**What just happened?**
- `npm init -y` creates a package.json file
- This file tracks your project's dependencies (libraries you'll use)

#### 2.2 Install Required Libraries

```bash
npm install express
```

**What is Express?**
- Express is a framework that makes it easy to create web servers in Node.js
- Think of it as a toolkit for building APIs

#### 2.3 Create the Product Service Code

Create a new file called `server.js` in the product-service folder:

**File: `product-service/server.js`**

```javascript
// Import the express library
const express = require('express');

// Create an Express application
const app = express();

// This allows our server to understand JSON data
app.use(express.json());

// Our in-memory database (just an array for now)
let products = [
  { id: 1, name: "Laptop", price: 999, stock: 10 },
  { id: 2, name: "Mouse", price: 25, stock: 50 },
  { id: 3, name: "Keyboard", price: 75, stock: 30 }
];

// ROUTE 1: Get all products
// When someone visits http://localhost:3001/products
app.get('/products', (req, res) => {
  console.log('📦 Request received: Get all products');
  res.json(products); // Send products as JSON response
});

// ROUTE 2: Get a single product by ID
// When someone visits http://localhost:3001/products/1
app.get('/products/:id', (req, res) => {
  const productId = parseInt(req.params.id); // Get ID from URL
  console.log(`📦 Request received: Get product with ID ${productId}`);
  
  // Find the product in our array
  const product = products.find(p => p.id === productId);
  
  if (product) {
    res.json(product); // Send the product
  } else {
    res.status(404).json({ error: "Product not found" });
  }
});

// ROUTE 3: Add a new product
app.post('/products', (req, res) => {
  console.log('📦 Request received: Create new product');
  
  const newProduct = {
    id: products.length + 1,
    name: req.body.name,
    price: req.body.price,
    stock: req.body.stock
  };
  
  products.push(newProduct);
  res.status(201).json(newProduct);
});

// Start the server on port 3001
const PORT = 3001;
app.listen(PORT, () => {
  console.log(`✅ Product Service is running on http://localhost:${PORT}`);
  console.log(`📦 Try: http://localhost:${PORT}/products`);
});
```

#### 2.4 Test the Product Service

**Terminal 1** (keep this open):
```bash
node server.js
```

You should see:
```
✅ Product Service is running on http://localhost:3001
📦 Try: http://localhost:3001/products
```

**Test in Browser:**
- Open browser and go to: http://localhost:3001/products
- You should see your product list in JSON format!

**Test with Postman (Better Option):**
1. Open Postman
2. Create a new request
3. Set method to GET
4. Enter URL: http://localhost:3001/products
5. Click "Send"
6. You should see all products!

**Try getting a single product:**
- URL: http://localhost:3001/products/1
- Should return just the Laptop

---

### Step 3: Building the Order Service

#### 3.1 Initialize the Order Service

Open a **NEW terminal** (keep the first one running!) and:

```bash
cd ecommerce-microservices/order-service
npm init -y
npm install express axios
```

**What is Axios?**
- Axios is a library for making HTTP requests
- We'll use it to communicate with the Product Service

#### 3.2 Create the Order Service Code

**File: `order-service/server.js`**

```javascript
const express = require('express');
const axios = require('axios'); // For making HTTP requests
const app = express();

app.use(express.json());

// Our orders database
let orders = [];

// Configuration: URLs of other services
const PRODUCT_SERVICE_URL = 'http://localhost:3001';
const NOTIFICATION_SERVICE_URL = 'http://localhost:3003';

// ROUTE 1: Create a new order
app.post('/orders', async (req, res) => {
  console.log('🛒 New order request received');
  
  try {
    // STEP 1: Get product information from Product Service
    console.log(`📞 Calling Product Service for product ${req.body.productId}`);
    const productResponse = await axios.get(
      `${PRODUCT_SERVICE_URL}/products/${req.body.productId}`
    );
    const product = productResponse.data;
    console.log(`✅ Product found: ${product.name}`);
    
    // STEP 2: Check if we have enough stock
    if (product.stock < req.body.quantity) {
      return res.status(400).json({
        error: "Not enough stock available"
      });
    }
    
    // STEP 3: Calculate total price
    const totalPrice = product.price * req.body.quantity;
    
    // STEP 4: Create the order
    const order = {
      id: orders.length + 1,
      productId: product.id,
      productName: product.name,
      quantity: req.body.quantity,
      totalPrice: totalPrice,
      customerEmail: req.body.customerEmail,
      status: 'confirmed',
      timestamp: new Date().toISOString()
    };
    
    orders.push(order);
    console.log(`✅ Order created: #${order.id}`);
    
    // STEP 5: Send notification (fire and forget - we don't wait for response)
    console.log('📧 Sending notification...');
    axios.post(`${NOTIFICATION_SERVICE_URL}/notify`, {
      orderId: order.id,
      customerEmail: order.customerEmail,
      message: `Order confirmed! ${order.quantity}x ${order.productName} - Total: $${order.totalPrice}`
    }).catch(err => {
      console.log('⚠️ Notification failed (but order is still valid):', err.message);
    });
    
    // STEP 6: Return the order to the customer
    res.status(201).json(order);
    
  } catch (error) {
    console.error('❌ Error creating order:', error.message);
    if (error.response && error.response.status === 404) {
      res.status(404).json({ error: "Product not found" });
    } else {
      res.status(500).json({
        error: "Failed to create order",
        details: error.message
      });
    }
  }
});

// ROUTE 2: Get all orders
app.get('/orders', (req, res) => {
  console.log('🛒 Request: Get all orders');
  res.json(orders);
});

// ROUTE 3: Get a specific order
app.get('/orders/:id', (req, res) => {
  const orderId = parseInt(req.params.id);
  console.log(`🛒 Request: Get order #${orderId}`);
  
  const order = orders.find(o => o.id === orderId);
  
  if (order) {
    res.json(order);
  } else {
    res.status(404).json({ error: "Order not found" });
  }
});

const PORT = 3002;
app.listen(PORT, () => {
  console.log(`✅ Order Service is running on http://localhost:${PORT}`);
  console.log(`🛒 Try: POST http://localhost:${PORT}/orders`);
});
```

#### 3.3 Test the Order Service

**Terminal 2** (new terminal):
```bash
cd ecommerce-microservices/order-service
node server.js
```

You should see:
```
✅ Order Service is running on http://localhost:3002
```

**Now you have TWO services running!**
- Terminal 1: Product Service (port 3001)
- Terminal 2: Order Service (port 3002)

---

### Step 4: Building the Notification Service

#### 4.1 Initialize the Notification Service

Open a **THIRD terminal** and:

```bash
cd ecommerce-microservices/notification-service
npm init -y
npm install express
```

#### 4.2 Create the Notification Service Code

**File: `notification-service/server.js`**

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Storage for sent notifications (just for tracking)
let notifications = [];

// ROUTE: Receive notification request
app.post('/notify', (req, res) => {
  console.log('📧 ================================');
  console.log('📧 NEW NOTIFICATION RECEIVED');
  console.log('📧 ================================');
  console.log(`📧 Order ID: ${req.body.orderId}`);
  console.log(`📧 Email: ${req.body.customerEmail}`);
  console.log(`📧 Message: ${req.body.message}`);
  console.log('📧 ================================');
  
  // In a real application, you would:
  // - Send an actual email using SendGrid, Mailgun, etc.
  // - Send SMS using Twilio
  // - Send push notification
  // - Write to a message queue
  
  // For now, we just log it and store it
  const notification = {
    id: notifications.length + 1,
    orderId: req.body.orderId,
    email: req.body.customerEmail,
    message: req.body.message,
    sentAt: new Date().toISOString(),
    status: 'sent'
  };
  
  notifications.push(notification);
  
  // Simulate some processing time
  setTimeout(() => {
    console.log('✅ Notification processed successfully!');
  }, 1000);
  
  res.json({
    status: 'notification sent',
    notificationId: notification.id
  });
});

// ROUTE: Get all notifications (for debugging)
app.get('/notifications', (req, res) => {
  res.json(notifications);
});

const PORT = 3003;
app.listen(PORT, () => {
  console.log(`✅ Notification Service is running on http://localhost:${PORT}`);
  console.log(`📧 Ready to send notifications!`);
});
```

#### 4.3 Test the Notification Service

**Terminal 3**:
```bash
cd ecommerce-microservices/notification-service
node server.js
```

---

### Step 5: Testing the Complete System

Now you should have **THREE services running**:
- Terminal 1: Product Service (3001)
- Terminal 2: Order Service (3002)
- Terminal 3: Notification Service (3003)

#### Test 1: View Products

**Postman**:
- Method: GET
- URL: http://localhost:3001/products
- Click "Send"

✅ You should see all products

#### Test 2: Create an Order (The Magic Moment!)

**Postman**:
- Method: POST
- URL: http://localhost:3002/orders
- Select "Body" tab → "raw" → "JSON"
- Paste this:

```json
{
  "productId": 1,
  "quantity": 2,
  "customerEmail": "student@example.com"
}
```

- Click "Send"

**What should happen:**
1. ✅ Order Service receives your request
2. 📞 Order Service calls Product Service to get product info
3. 💰 Order Service calculates total price
4. ✅ Order Service creates the order
5. 📧 Order Service calls Notification Service
6. 📧 Notification Service logs the notification
7. ✅ You receive the order confirmation!

**Check all three terminals** - you'll see logs showing the communication between services!

#### Test 3: View All Orders

**Postman**:
- Method: GET
- URL: http://localhost:3002/orders
- Click "Send"

You'll see your order!

#### Test 4: View Notifications

**Postman**:
- Method: GET
- URL: http://localhost:3003/notifications
- Click "Send"

You'll see the notification that was sent!

---

### Understanding What Happened

#### Communication Flow:

```
YOU (Postman)
    ↓
    ↓ POST /orders
    ↓
ORDER SERVICE (port 3002)
    ↓
    ↓ GET /products/1
    ↓
PRODUCT SERVICE (port 3001)
    ↓
    ↓ Returns product info
    ↓
ORDER SERVICE (calculates price, creates order)
    ↓
    ↓ POST /notify
    ↓
NOTIFICATION SERVICE (port 3003)
    ↓
    ↓ Logs notification
    ↓
Returns success to ORDER SERVICE
    ↓
ORDER SERVICE returns order to YOU
```

#### Key Concepts Demonstrated:

1. **Synchronous Communication** (Order → Product)
   - Order Service WAITS for Product Service response
   - If Product Service is down, order creation fails
   - Uses: When you need the data immediately

2. **Asynchronous Communication** (Order → Notification)
   - Order Service DOESN'T wait for notification
   - If Notification Service is down, order still succeeds
   - Uses: When the operation is not critical for the main flow

---

### Experiments for Students

#### Experiment 1: What happens when Product Service is down?

1. **Stop Product Service** (Press Ctrl+C in Terminal 1)
2. Try creating an order in Postman
3. **Question**: What error do you get? Why?
4. **Restart Product Service** and try again

#### Experiment 2: What happens when Notification Service is down?

1. **Stop Notification Service** (Press Ctrl+C in Terminal 3)
2. Try creating an order
3. **Question**: Does the order still get created? Why?

#### Experiment 3: Invalid Product ID

Try creating an order with:
```json
{
  "productId": 999,
  "quantity": 1,
  "customerEmail": "test@example.com"
}
```

**Question**: What error message do you get?

#### Experiment 4: Not Enough Stock

Try ordering 100 laptops (we only have 10):
```json
{
  "productId": 1,
  "quantity": 100,
  "customerEmail": "test@example.com"
}
```

**Question**: How does the system handle this?

---

### Common Issues and Solutions

#### Issue 1: "Port already in use"

**Error**: `EADDRINUSE: address already in use :::3001`

**Solution**:
- Another program is using that port
- Close the other program or change the port number
- Find what's using the port:
  - Mac/Linux: `lsof -i :3001`
  - Windows: `netstat -ano | findstr :3001`

#### Issue 2: "Cannot GET /orders"

**Problem**: You're using GET instead of POST

**Solution**:
- Make sure you're using POST method in Postman
- Check the URL is exactly: http://localhost:3002/orders

#### Issue 3: Services can't communicate

**Error**: `ECONNREFUSED`

**Solution**:
- Make sure all services are running
- Check the terminal outputs
- Verify the PORT numbers match in the URLs

#### Issue 4: "npm: command not found"

**Solution**: Node.js isn't installed properly
- Reinstall Node.js from https://nodejs.org/
- Restart your terminal after installation

---

### Assignment for Students

#### Part A: Modify Product Service

Add a new endpoint to update product stock:

```javascript
app.put('/products/:id/stock', (req, res) => {
  // Find product
  // Update stock
  // Return updated product
});
```

#### Part B: Enhance Order Service

Add validation to check if quantity is positive:

```javascript
if (req.body.quantity <= 0) {
  return res.status(400).json({
    error: "Quantity must be greater than 0"
  });
}
```

#### Part C: Add New Feature

Create a new endpoint in Order Service to cancel an order:
- Route: DELETE /orders/:id
- Should change order status to 'cancelled'
- Should send a cancellation notification

---

### Learning Outcomes

After completing Week 1, students should understand:

✅ What microservices are (independent, small services)  
✅ How services communicate (HTTP/REST APIs)  
✅ Synchronous vs Asynchronous communication  
✅ How to use Express.js to build APIs  
✅ How to use Axios to make HTTP requests  
✅ How to test APIs with Postman  
✅ Basic error handling in distributed systems

---

### Quick Reference

#### Starting All Services:

**Terminal 1**:
```bash
cd product-service
node server.js
```

**Terminal 2**:
```bash
cd order-service
node server.js
```

**Terminal 3**:
```bash
cd notification-service
node server.js
```

#### Stopping Services:
Press Ctrl+C in each terminal

#### Service URLs:
- Product Service: http://localhost:3001
- Order Service: http://localhost:3002
- Notification Service: http://localhost:3003

---

### Discussion Questions

1. **What are the advantages of splitting the application into three services?**
2. **What would happen if we combined all three services into one application?**
3. **Why do we use different ports for each service?**
4. **How would you handle the situation if Product Service takes 30 seconds to respond?**
5. **What security concerns exist with services communicating over HTTP?**

---

## Week 3: Docker Compose Multi-Container Application

### Prerequisites to Install

1. **Docker Desktop**
   - **Windows/Mac**: Download from https://www.docker.com/products/docker-desktop/
   - **Linux**: Follow instructions at https://docs.docker.com/engine/install/
   - After installation, verify:
     ```bash
     docker --version
     docker-compose --version
     ```
   - You should see version numbers (e.g., Docker version 24.0.6)

2. **Your Week 1-2 Project**
   - Make sure you have the ecommerce-microservices folder with all three services
   - If not, we'll create it again

---

### What We're Building

#### Current Problem:
- Opening 3 terminals manually
- Starting each service one by one
- Remembering which port each service uses
- Services can't find each other if you restart

#### Solution with Docker Compose:
- Start ALL services with ONE command
- Services automatically discover each other
- Everything runs in isolated containers
- Easy to share with others (it works the same on any computer!)

---

### Understanding Docker Basics

#### What is Docker?

Think of Docker like a **shipping container** for software:
- Just like shipping containers hold goods and can be transported anywhere
- Docker containers hold your application and can run anywhere
- Each container is isolated (like separate apartments in a building)

#### What is a Docker Image?
- An **image** is like a **recipe** or **blueprint**
- It contains your code + all dependencies
- Example: "Recipe for Product Service"

#### What is a Docker Container?
- A **container** is a **running instance** of an image
- Example: "Product Service actually running"
- You can have multiple containers from one image

#### What is Docker Compose?
- A tool to manage **multiple containers**
- Uses a single configuration file (docker-compose.yml)
- Starts/stops all services together

---

### Step 1: Prepare Your Project Structure

Make sure your folder structure looks like this:

```
ecommerce-microservices/
├── product-service/
│   ├── server.js
│   └── package.json
├── order-service/
│   ├── server.js
│   └── package.json
├── notification-service/
│   ├── server.js
│   └── package.json
└── docker-compose.yml (we'll create this)
```

---

### Step 2: Create Dockerfile for Product Service

#### What is a Dockerfile?

A Dockerfile is a text file with instructions on how to build a Docker image. Think of it as a recipe.

#### 2.1 Create the Dockerfile

In the product-service folder, create a new file called `Dockerfile` (no extension!):

**File: `product-service/Dockerfile`**

```dockerfile
# Start with a base image that has Node.js installed
# alpine = lightweight version
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first
# Why? Docker caches layers, so if dependencies don't change,
# this step will be faster next time
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy all other files
COPY . .

# Expose port 3001 (documentation - tells others which port is used)
EXPOSE 3001

# Command to run when container starts
CMD ["node", "server.js"]
```

#### 2.2 Understanding Each Line

**FROM node:18-alpine**
- **FROM**: Start with this base image
- **node:18-alpine**: Official Node.js version 18, Alpine Linux (small)
- Like saying: "Start with a computer that has Node.js pre-installed"

**WORKDIR /app**
- **WORKDIR**: Set working directory
- All subsequent commands run from /app folder
- Like saying: "cd into /app folder"

**COPY package*.json ./**
- **COPY**: Copy files from your computer to the container
- **package*.json**: Copies both package.json and package-lock.json
- **./**: Copy to current directory (/app)

**RUN npm install**
- **RUN**: Execute a command during image build
- Installs all Node.js dependencies

**COPY . .**
- Copy everything else from current folder to /app
- **First .** = Your computer's current folder
- **Second .** = Container's /app folder

**EXPOSE 3001**
- Documents which port the service uses
- Doesn't actually open the port (that's done in docker-compose)

**CMD ["node", "server.js"]**
- **CMD**: Command to run when container starts
- Starts your Node.js application

#### 2.3 Create .dockerignore File

In product-service, create `.dockerignore`:

**File: `product-service/.dockerignore`**

```
node_modules
npm-debug.log
.DS_Store
.env
```

**Why?**
- We don't want to copy node_modules (will be installed fresh in container)
- Keeps the image smaller and build faster

---

### Step 3: Create Dockerfiles for Other Services

#### 3.1 Order Service Dockerfile

**File: `order-service/Dockerfile`**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3002

CMD ["node", "server.js"]
```

**File: `order-service/.dockerignore`**

```
node_modules
npm-debug.log
.DS_Store
.env
```

#### 3.2 Notification Service Dockerfile

**File: `notification-service/Dockerfile`**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3003

CMD ["node", "server.js"]
```

**File: `notification-service/.dockerignore`**

```
node_modules
npm-debug.log
.DS_Store
.env
```

---

### Step 4: Update Service Code for Docker

#### Problem with Current Code

In Week 1-2, our services used localhost to communicate:
```javascript
const PRODUCT_SERVICE_URL = 'http://localhost:3001';
```

**This won't work in Docker!** Each container has its own localhost.

#### Solution: Use Service Names

Docker Compose creates a network where services can find each other by name!

#### 4.1 Update Order Service

**File: `order-service/server.js`** (Update these lines)

```javascript
// OLD (Week 1-2):
// const PRODUCT_SERVICE_URL = 'http://localhost:3001';
// const NOTIFICATION_SERVICE_URL = 'http://localhost:3003';

// NEW (Docker):
const PRODUCT_SERVICE_URL = process.env.PRODUCT_SERVICE_URL || 'http://localhost:3001';
const NOTIFICATION_SERVICE_URL = process.env.NOTIFICATION_SERVICE_URL || 'http://localhost:3003';
```

**What Changed?**
- `process.env.PRODUCT_SERVICE_URL`: Read from environment variable
- `|| 'http://localhost:3001'`: Fallback to localhost if not set
- This makes it work BOTH locally AND in Docker!

The rest of the order-service code stays the same!

---

### Step 5: Create Docker Compose File

This is the **magic file** that brings everything together!

#### 5.1 Create docker-compose.yml

In the **root folder** (ecommerce-microservices), create:

**File: `docker-compose.yml`**

```yaml
version: '3.8'

# Define our services
services:
  # Product Service
  product-service:
    build: ./product-service  # Build from this folder
    container_name: product-service  # Name of the container
    ports:
      - "3001:3001"  # Map port 3001 on your computer to port 3001 in container
    networks:
      - ecommerce-network  # Connect to this network
    healthcheck:  # Check if service is healthy
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3001/products"]
      interval: 30s  # Check every 30 seconds
      timeout: 10s
      retries: 3
      start_period: 40s

  # Order Service
  order-service:
    build: ./order-service
    container_name: order-service
    ports:
      - "3002:3002"
    environment:  # Set environment variables
      - PRODUCT_SERVICE_URL=http://product-service:3001
      - NOTIFICATION_SERVICE_URL=http://notification-service:3003
    depends_on:  # Wait for these services
      - product-service
      - notification-service
    networks:
      - ecommerce-network

  # Notification Service
  notification-service:
    build: ./notification-service
    container_name: notification-service
    ports:
      - "3003:3003"
    networks:
      - ecommerce-network

# Define networks
networks:
  ecommerce-network:
    driver: bridge  # Bridge network - allows containers to communicate
```

#### 5.2 Understanding docker-compose.yml

**Version**
```yaml
version: '3.8'
```
- Docker Compose file format version
- Use '3.8' for modern features

**Services**
```yaml
services:
  product-service:
```
- List all your microservices here
- Each service is a container

**Build**
```yaml
build: ./product-service
```
- Tell Docker to build an image from this folder
- Looks for Dockerfile in that folder

**Ports**
```yaml
ports:
  - "3001:3001"
```
- **Format**: "HOST_PORT:CONTAINER_PORT"
- **3001** (left): Port on your computer
- **3001** (right): Port inside container
- You can access at http://localhost:3001

**Environment**
```yaml
environment:
  - PRODUCT_SERVICE_URL=http://product-service:3001
```
- Set environment variables
- **Magic**: Use service names as hostnames!
- `product-service` resolves to the container's IP automatically

**Depends On**
```yaml
depends_on:
  - product-service
```
- Start this service AFTER product-service
- Ensures order of startup

**Networks**
```yaml
networks:
  - ecommerce-network
```
- Connect this container to this network
- All containers on same network can communicate

**Health Check**
```yaml
healthcheck:
  test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3001/products"]
```
- Periodically check if service is running
- If it fails 3 times, container is marked unhealthy
- Docker can restart unhealthy containers

---

### Step 6: Building and Running with Docker Compose

#### 6.1 Build the Images

Open terminal in the **root folder** (ecommerce-microservices) and run:

```bash
docker-compose build
```

**What happens?**
- Docker reads docker-compose.yml
- For each service, it finds the Dockerfile
- Builds an image for each service
- Downloads Node.js base image if needed (first time only)

**You'll see output like:**
```
[+] Building 45.2s (30/30) FINISHED
 => [product-service internal] load .dockerignore
 => [product-service internal] load build definition from Dockerfile
 => [product-service] CACHED [1/5] FROM docker.io/library/node:18-alpine
 ...
```

**This may take 2-5 minutes the first time!**

#### 6.2 Start All Services

```bash
docker-compose up
```

**You'll see logs from ALL services in one terminal!**

```
product-service      | ✅ Product Service is running on http://localhost:3001
order-service        | ✅ Order Service is running on http://localhost:3002
notification-service | ✅ Notification Service is running on http://localhost:3003
```

**🎉 Success! All three services are running!**

#### 6.3 Run in Background (Detached Mode)

To run without seeing logs:

```bash
docker-compose up -d
```

**-d** = detached mode (runs in background)

#### 6.4 View Logs

To see logs of all services:
```bash
docker-compose logs
```

To see logs of one service:
```bash
docker-compose logs product-service
```

To follow logs (live updates):
```bash
docker-compose logs -f
```

#### 6.5 Stop All Services

```bash
docker-compose down
```

**This stops and removes all containers!**

---

### Step 7: Testing the Dockerized System

#### 7.1 Check Running Containers

```bash
docker-compose ps
```

**Output:**
```
NAME                  STATUS              PORTS
product-service       running             0.0.0.0:3001->3001/tcp
order-service         running             0.0.0.0:3002->3002/tcp
notification-service  running             0.0.0.0:3003->3003/tcp
```

#### 7.2 Test with Postman

**Same tests as Week 1-2!**

**Test 1: Get Products**
- Method: GET
- URL: http://localhost:3001/products
- Should work the same!

**Test 2: Create Order**
- Method: POST
- URL: http://localhost:3002/orders
- Body (JSON):
```json
{
  "productId": 1,
  "quantity": 2,
  "customerEmail": "student@example.com"
}
```

**Watch the logs!**
```bash
docker-compose logs -f
```

You'll see the communication between services!

#### 7.3 Test Service Communication

The **big difference**: Services now communicate using service names!

**Inside order-service**, it calls:
- `http://product-service:3001/products/1` (not localhost!)
- `http://notification-service:3003/notify` (not localhost!)

**Docker's network makes this work automatically!**

---

### Step 8: Docker Compose Commands Cheat Sheet

#### Starting Services

```bash
# Build images
docker-compose build

# Start services (see logs)
docker-compose up

# Start in background
docker-compose up -d

# Rebuild and start
docker-compose up --build
```

#### Stopping Services

```bash
# Stop services (keep containers)
docker-compose stop

# Stop and remove containers
docker-compose down

# Stop and remove containers + volumes
docker-compose down -v
```

#### Viewing Information

```bash
# List running containers
docker-compose ps

# View logs (all services)
docker-compose logs

# View logs (one service)
docker-compose logs product-service

# Follow logs (live)
docker-compose logs -f

# View last 50 lines
docker-compose logs --tail=50
```

#### Managing Individual Services

```bash
# Start only one service
docker-compose up product-service

# Stop one service
docker-compose stop order-service

# Restart one service
docker-compose restart order-service

# Rebuild one service
docker-compose build product-service
```

#### Debugging

```bash
# Execute command in running container
docker-compose exec product-service sh
# This opens a shell inside the container!
# Try: ls, pwd, cat server.js, etc.
# Exit with: exit

# View resource usage
docker stats
```

---

### Understanding Docker Networking

#### How Services Find Each Other

When you use Docker Compose:

1. **Docker creates a network** called ecommerce-network
2. **Each container gets**:
   - An IP address (e.g., 172.18.0.2)
   - A hostname (same as service name)
3. **Built-in DNS**: Docker translates product-service to its IP

**Example:**
```
order-service tries to call: http://product-service:3001
    ↓
Docker DNS resolves to: http://172.18.0.2:3001
    ↓
Request reaches product-service container
```

#### Network Isolation

- **Inside network**: Containers can talk to each other
- **Outside network**: Only exposed ports are accessible from your computer

```
Your Computer (localhost:3001)
    ↓
Docker Network
    ↓
product-service:3001
    ↑
order-service (can communicate)
```

---

### Experiments for Students

#### Experiment 1: Scale Services

**Question**: What if we have too many orders and need multiple order services?

```bash
docker-compose up -d --scale order-service=3
```

**This creates 3 instances of order-service!**

**Problem**: Port conflict! (3 services can't use port 3002)

**Solution**: Remove port mapping for scaled services and use a load balancer (advanced topic)

#### Experiment 2: Restart a Service

```bash
# Stop order service
docker-compose stop order-service

# Try to create an order - it should fail!

# Start it again
docker-compose start order-service

# Try again - it works!
```

#### Experiment 3: View Logs of Crash

**Modify order-service to crash:**

Add this to order-service/server.js before app.listen:
```javascript
setTimeout(() => {
  throw new Error('Intentional crash for testing!');
}, 10000); // Crash after 10 seconds
```

**Rebuild and run:**
```bash
docker-compose up --build
```

**Watch it crash in logs!**

Then check status:
```bash
docker-compose ps
```

**You'll see order-service has exited!**

#### Experiment 4: Network Inspection

```bash
# List networks
docker network ls

# Inspect our network
docker network inspect ecommerce-microservices_ecommerce-network

# You'll see all connected containers and their IPs!
```

#### Experiment 5: Execute Commands Inside Container

```bash
# Open shell in product-service
docker-compose exec product-service sh

# Now you're INSIDE the container!
# Try:
pwd           # Shows /app
ls            # Shows your files
cat server.js # Shows your code
node --version # Shows Node version
exit          # Exit container shell
```

---

### Step 9: Adding a Database (PostgreSQL)

Let's make it more realistic by adding a database!

#### 9.1 Update docker-compose.yml

Add this service to docker-compose.yml:

```yaml
services:
  # ... existing services ...

  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine  # Use official PostgreSQL image
    container_name: postgres-db
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password123
      - POSTGRES_DB=ecommerce
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data  # Persist data
    networks:
      - ecommerce-network

# Add volumes section at the bottom
volumes:
  postgres-data:  # Named volume for database persistence
```

#### 9.2 Update Product Service to Use Database

**Install PostgreSQL library:**

Update product-service/package.json:
```json
{
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.11.0"
  }
}
```

**Update product-service/server.js:**

```javascript
const express = require('express');
const { Pool } = require('pg');
const app = express();

app.use(express.json());

// Database connection
const pool = new Pool({
  host: process.env.DB_HOST || 'localhost',
  port: 5432,
  database: process.env.DB_NAME || 'ecommerce',
  user: process.env.DB_USER || 'postgres',
  password: process.env.DB_PASSWORD || 'password123'
});

// Initialize database table
async function initDatabase() {
  try {
    await pool.query(`
      CREATE TABLE IF NOT EXISTS products (
        id SERIAL PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        price DECIMAL(10, 2) NOT NULL,
        stock INTEGER NOT NULL
      )
    `);
    
    // Insert sample data if table is empty
    const result = await pool.query('SELECT COUNT(*) FROM products');
    if (result.rows[0].count === '0') {
      await pool.query(`
        INSERT INTO products (name, price, stock) VALUES
        ('Laptop', 999.00, 10),
        ('Mouse', 25.00, 50),
        ('Keyboard', 75.00, 30)
      `);
      console.log('✅ Sample products inserted');
    }
  } catch (error) {
    console.error('❌ Database initialization failed:', error);
  }
}

// Get all products
app.get('/products', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM products');
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get single product
app.get('/products/:id', async (req, res) => {
  try {
    const result = await pool.query(
      'SELECT * FROM products WHERE id = $1',
      [req.params.id]
    );
    
    if (result.rows.length > 0) {
      res.json(result.rows[0]);
    } else {
      res.status(404).json({ error: 'Product not found' });
    }
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

const PORT = 3001;

// Start server after database is initialized
initDatabase().then(() => {
  app.listen(PORT, () => {
    console.log(`✅ Product Service with PostgreSQL running on port ${PORT}`);
  });
});
```

#### 9.3 Update Product Service Environment Variables

In docker-compose.yml, update product-service:

```yaml
product-service:
  build: ./product-service
  container_name: product-service
  ports:
    - "3001:3001"
  environment:
    - DB_HOST=postgres  # Use service name!
    - DB_NAME=ecommerce
    - DB_USER=postgres
    - DB_PASSWORD=password123
  depends_on:
    - postgres
  networks:
    - ecommerce-network
```

#### 9.4 Test with Database

```bash
# Rebuild and start
docker-compose up --build

# Test - products now come from PostgreSQL!
```

**Check database directly:**

```bash
# Connect to PostgreSQL container
docker-compose exec postgres psql -U postgres -d ecommerce

# Run SQL query
SELECT * FROM products;

# Exit
\q
```

---

### Step 10: Monitoring with Docker Desktop

#### Using Docker Desktop GUI

1. **Open Docker Desktop**
2. **Click "Containers"** tab
3. You'll see your running services!

**For each container you can:**
- ▶️ Start/Stop
- 🔄 Restart
- 📋 View logs
- 📊 See resource usage (CPU, Memory)
- 🖥️ Open terminal (CLI)
- 🗑️ Delete

**Try it:**
- Click on product-service
- Click "Logs" tab
- You can see all logs with timestamps!

---

### Common Issues and Solutions

#### Issue 1: Port Already in Use

**Error**: `Bind for 0.0.0.0:3001 failed: port is already allocated`

**Solution**:
```bash
# Check what's using the port
lsof -i :3001  # Mac/Linux
netstat -ano | findstr :3001  # Windows

# Either stop that process, or change port in docker-compose.yml:
ports:
  - "3011:3001"  # Use port 3011 on your computer instead
```

#### Issue 2: Cannot Connect to Service

**Error**: `ECONNREFUSED` when services try to communicate

**Solution**:
- Check all services are running: `docker-compose ps`
- Check network: `docker network ls`
- Verify service names in environment variables match docker-compose.yml
- Check logs: `docker-compose logs`

#### Issue 3: Changes Not Reflected

**Problem**: Modified code but container still shows old behavior

**Solution**:
```bash
# Rebuild the specific service
docker-compose build product-service

# Or rebuild everything
docker-compose up --build
```

#### Issue 4: Database Data Persists

**Problem**: Want to start fresh, but old data remains

**Solution**:
```bash
# Remove containers and volumes
docker-compose down -v

# Start fresh
docker-compose up --build
```

#### Issue 5: Out of Disk Space

**Problem**: Docker images taking too much space

**Solution**:
```bash
# See disk usage
docker system df

# Remove unused images, containers, networks
docker system prune

# Remove everything (careful!)
docker system prune -a
```

---

### Assignment for Students

#### Part A: Add Redis Cache (Intermediate)

Add Redis to docker-compose.yml:

```yaml
redis:
  image: redis:7-alpine
  container_name: redis-cache
  ports:
    - "6379:6379"
  networks:
    - ecommerce-network
```

Modify product-service to cache product data in Redis.

#### Part B: Add Environment-Specific Configs (Advanced)

Create multiple docker-compose files:
- docker-compose.yml (base)
- docker-compose.dev.yml (development overrides)
- docker-compose.prod.yml (production overrides)

Run with:
```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

#### Part C: Health Check Implementation

Add health check endpoints to all services:

```javascript
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date() });
});
```

Update health checks in docker-compose.yml to use these endpoints.

#### Part D: Add nginx Load Balancer

Add nginx to distribute traffic across multiple order-service instances.

---

### Learning Outcomes

After completing Week 3, students should understand:

✅ What Docker containers and images are  
✅ How to write a Dockerfile  
✅ What Docker Compose is and why it's useful  
✅ How to orchestrate multiple services  
✅ Docker networking and service discovery  
✅ How to debug containerized applications  
✅ How to persist data with volumes  
✅ Basic Docker commands and troubleshooting

---

### Quick Reference Card

#### Essential Commands

```bash
# Start everything
docker-compose up -d

# View logs
docker-compose logs -f

# Stop everything
docker-compose down

# Rebuild after code changes
docker-compose up --build

# Check status
docker-compose ps

# Restart one service
docker-compose restart order-service

# Execute command in container
docker-compose exec product-service sh

# View resource usage
docker stats
```

#### File Locations

- **Dockerfile** → In each service folder
- **docker-compose.yml** → In root folder
- **.dockerignore** → In each service folder

---

### Discussion Questions

1. **What are the advantages of using Docker over running services directly?**
2. **How does Docker Compose make development easier?**
3. **What happens to data in containers when they stop?**
4. **Why do we use service names instead of IP addresses?**
5. **How would you deploy this to production (AWS, Azure, Google Cloud)?**
6. **What security concerns exist with containers?**

---

### Next Week Preview

**Week 4-5: Converting Monolith to Microservices**
- Start with a single large application
- Break it down into microservices
- Use Docker Compose to run the new architecture
- Compare complexity, performance, and maintainability

---

### Pro Tips

1. **Use `docker-compose up --build` when you change code** - ensures latest version runs
2. **Name your containers** - makes logs easier to read
3. **Use volumes for databases** - data persists across restarts
4. **Check logs first** when debugging - most issues show up there
5. **Use health checks** - automatically restart failed services
6. **Keep images small** - use alpine base images
7. **Don't store secrets in Dockerfiles** - use environment variables

---

### Additional Resources

- **Docker Official Docs**: https://docs.docker.com/
- **Docker Compose Reference**: https://docs.docker.com/compose/
- **Play with Docker**: https://labs.play-with-docker.com/ (free online playground)
- **Docker Hub**: https://hub.docker.com/ (find pre-built images)

---

## Contributing

Feel free to submit issues and pull requests to improve this guide!

## License

This educational material is provided for learning purposes.
