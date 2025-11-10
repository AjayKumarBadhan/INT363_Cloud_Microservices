# Complete Guide: Microservice Experiments with Docker and Flask

## Table of Contents
1. [Lab Experiment 2: Hello World Microservice](#lab-experiment-2)
2. [Lab Experiment 3: Login Page Microservice](#lab-experiment-3)
3. [Lab Experiment 4: Login Microservice on AWS](#lab-experiment-4)

---

## Lab Experiment 2: Hello World Microservice

### Objective
Create a simple microservice application using Docker and Python Flask that serves a "Hello, World!" message. This experiment introduces the fundamental concepts of containerization and microservice architecture.

### Prerequisites
- Docker installed on your system
- Basic understanding of Python
- Text editor or IDE

---

### Step 1: Create a Directory for the Application

```bash
mkdir flask-hello-microservice
cd flask-hello-microservice
```

**Explanation:**
- `mkdir flask-hello-microservice`: Creates a new directory to organize all project files
- `cd flask-hello-microservice`: Changes the current working directory to the newly created folder
- **Why this matters**: Organizing files in a dedicated directory follows best practices for project management and makes deployment easier

---

### Step 2: Create a Python File (app.py)

```bash
touch app.py
```

**Explanation:**
- Creates an empty Python file that will contain our Flask application code
- `app.py` is the conventional name for the main application file in Flask projects

---

### Step 3: Add the Following Code to 'app.py'

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Detailed Line-by-Line Explanation:**

**Line 1: `from flask import Flask`**
- Imports the Flask class from the flask module
- Flask is a micro web framework for Python that allows us to build web applications
- The Flask class is used to create our application instance

**Line 3: `app = Flask(__name__)`**
- Creates an instance of the Flask application
- `__name__` is a special Python variable that represents the name of the current module
- Flask uses this to determine the root path of the application, which helps locate resources like templates and static files
- The `app` object will handle HTTP requests and responses

**Line 5: `@app.route('/')`**
- This is a Python decorator that binds a URL route to a function
- The `'/'` represents the root URL of the application (e.g., http://localhost:5000/)
- When a user visits this URL, Flask will execute the function immediately below this decorator
- **Routes** are the mechanism by which Flask maps URLs to Python functions

**Line 6-7: `def hello():`**
- Defines a view function named `hello`
- This function will be called when someone accesses the root URL
- View functions are responsible for generating responses to HTTP requests

**Line 7: `return "Hello, World!"`**
- Returns a simple string response to the client's browser
- This text will be displayed when the user visits the application
- Flask automatically converts this string into an HTTP response with status code 200 (OK)

**Line 9: `if __name__ == '__main__':`**
- This conditional checks if the script is being run directly (not imported as a module)
- `__name__` equals `'__main__'` only when the file is executed directly
- This is a Python idiom that prevents code from running when the module is imported

**Line 10: `app.run(host='0.0.0.0', port=5000)`**
- Starts the Flask development server
- `host='0.0.0.0'`: Makes the server accessible from any network interface
  - `0.0.0.0` means "all IPv4 addresses on the local machine"
  - This is crucial for Docker containers to accept external connections
  - Without this, the server would only be accessible from within the container itself
- `port=5000`: Specifies that the server should listen on port 5000
  - Port 5000 is the default port for Flask applications
  - This port must match the port we expose in our Dockerfile

---

### Step 4: Create a Requirements File

```bash
touch requirements.txt
```

**Explanation:**
- Creates a file that lists all Python dependencies needed for the project
- `requirements.txt` is the standard convention in Python projects for dependency management
- This file enables reproducible environments across different systems

---

### Step 5: Add the Following Content to 'requirements.txt'

```
Flask==2.3.0
```

**Explanation:**
- Specifies that our application requires Flask version 2.3.0
- The `==` syntax pins the exact version, ensuring consistency across deployments
- **Why version pinning matters**: Different Flask versions may have breaking changes; pinning ensures your code works reliably
- When Docker builds the image, it will install this specific version of Flask

---

### Step 6: Create a Dockerfile

```bash
touch Dockerfile
```

**Explanation:**
- Creates a Dockerfile, which is a text document containing instructions to build a Docker image
- Docker images are templates used to create containers
- The Dockerfile has no file extension by convention

---

### Step 7: Add the Following Content to the "Dockerfile"

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

**Detailed Line-by-Line Explanation:**

**Line 1: `FROM python:3.9-slim`**
- Specifies the base image for our Docker container
- `python:3.9-slim` is an official Python image with Python 3.9 pre-installed
- `slim` variant is a smaller image that includes only essential packages, reducing the final image size
- **Why this matters**: Starting from a base image saves time and ensures a consistent environment
- This layer contains the Python interpreter and basic system libraries

**Line 3: `WORKDIR /app`**
- Sets the working directory inside the container to `/app`
- All subsequent commands will be executed relative to this directory
- If the directory doesn't exist, Docker creates it automatically
- **Best practice**: Using a dedicated working directory keeps the container filesystem organized

**Line 5: `COPY requirements.txt requirements.txt`**
- Copies the `requirements.txt` file from your local machine (source) to the container (destination)
- Syntax: `COPY <source> <destination>`
- The destination path is relative to the WORKDIR (`/app/requirements.txt`)
- **Why copy this first**: Docker caches layers; by copying requirements separately, we can leverage caching if dependencies don't change

**Line 6: `RUN pip install -r requirements.txt`**
- Executes a command during the image build process
- `pip install -r requirements.txt` reads the requirements file and installs all listed packages
- This creates a new layer in the Docker image with all Python dependencies installed
- **Important**: This happens at build time, not runtime

**Line 8: `COPY . .`**
- Copies all remaining files from the current directory (`.`) on your host to the working directory (`.`) in the container
- This includes `app.py` and any other project files
- Placed after installing dependencies to optimize Docker's layer caching

**Line 10: `EXPOSE 5000`**
- Documents that the container listens on port 5000 at runtime
- This is primarily documentation; it doesn't actually publish the port
- Other developers and tools can see which ports the application uses
- **Note**: You still need to use `-p` flag when running the container to actually map ports

**Line 12: `CMD ["python", "app.py"]`**
- Specifies the default command to run when the container starts
- Uses JSON array syntax (exec form), which is preferred over shell form
- Executes `python app.py`, starting our Flask application
- **Important**: Only the last CMD instruction in a Dockerfile takes effect
- This runs at container runtime, not during the build

---

### Step 8: Build the Docker Container

```bash
docker build -t flask-hello-microservice .
```

**Explanation:**
- `docker build`: Command to build a Docker image from a Dockerfile
- `-t flask-hello-microservice`: Tags (names) the image as "flask-hello-microservice"
  - The `-t` flag stands for "tag"
  - This name will be used to reference the image later
- `.`: Specifies the build context (current directory)
  - Docker will look for a Dockerfile in this directory
  - All files in this directory are available to COPY commands

**What happens during the build:**
1. Docker reads the Dockerfile
2. Downloads the Python base image (if not cached)
3. Creates layers for each instruction
4. Installs Flask and dependencies
5. Copies application files
6. Creates a final image tagged with the specified name

---

### Step 9: Run the Docker Container

```bash
docker run -d -p 5000:5000 flask-hello-microservice
```

**Explanation:**
- `docker run`: Command to create and start a container from an image
- `-d`: Runs the container in detached mode (background)
  - The container runs independently of your terminal session
  - Returns the container ID and gives you back the command prompt
- `-p 5000:5000`: Maps port 5000 on the host to port 5000 in the container
  - Syntax: `-p <host_port>:<container_port>`
  - Allows external access to the containerized application
  - The first 5000 is your local machine's port
  - The second 5000 is the container's internal port
- `flask-hello-microservice`: Name of the image to run

**Access the application:**
- Open a web browser and navigate to: `http://localhost:5000`
- You should see: "Hello, World!"

---

### Step 10: Stop and Remove the Container

**List running containers:**
```bash
docker ps
```
- Shows all running containers
- Note the CONTAINER ID or NAME from the output

**Stop the container:**
```bash
docker stop <container_id>
```
- Gracefully stops the running container
- Replace `<container_id>` with the actual ID from `docker ps`
- The container is stopped but not deleted

**Remove the container:**
```bash
docker rm <container_id>
```
- Permanently removes the stopped container
- Frees up system resources
- The image remains available for creating new containers

**Alternative (force remove running container):**
```bash
docker rm -f <container_id>
```
- Combines stop and remove in one command
- The `-f` flag forces removal even if the container is running

---

## Lab Experiment 3: Login Page Microservice

### Objective
Create a more advanced microservice application that serves an HTML login page with username and password fields. This experiment introduces templating and form handling in Flask.

### Prerequisites
- Completed Lab Experiment 2
- Understanding of HTML basics
- Docker installed and running

---

### Step 1: Create a Directory for the Application

```bash
mkdir flask-login-microservice
cd flask-login-microservice
```

**Explanation:**
- Creates a new project directory for our login microservice
- Keeps this project separate from the previous experiment
- Follows microservice principles of separation of concerns

---

### Step 2: Create a Python File (app.py)

```bash
touch app.py
```

---

### Step 3: Add the Following Code to 'app.py'

```python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/')
def home():
    return "Welcome to the Login Microservice!"

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        return f"Username: {username}, Password: {password}"
    return render_template('login.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Detailed Line-by-Line Explanation:**

**Line 1: `from flask import Flask, render_template, request`**
- Imports three essential components from Flask:
  - `Flask`: The main application class (as before)
  - `render_template`: Function to render HTML templates with dynamic data
  - `request`: Object that contains data from incoming HTTP requests (forms, query parameters, etc.)

**Line 3: `app = Flask(__name__)`**
- Creates the Flask application instance (same as Experiment 2)

**Line 5-7: Home route**
```python
@app.route('/')
def home():
    return "Welcome to the Login Microservice!"
```
- Defines a simple home page at the root URL
- Returns a welcome message
- This demonstrates that a single application can have multiple routes

**Line 9: `@app.route('/login', methods=['GET', 'POST'])`**
- Defines a route for the `/login` URL path
- `methods=['GET', 'POST']`: Specifies that this route accepts both GET and POST HTTP methods
  - **GET**: Used when users navigate to the page (displaying the form)
  - **POST**: Used when users submit the form with credentials
- Without specifying methods, Flask only accepts GET requests by default

**Line 10-15: Login function**
```python
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        return f"Username: {username}, Password: {password}"
    return render_template('login.html')
```

**Line 11: `if request.method == 'POST':`**
- Checks if the incoming request is a POST request (form submission)
- `request.method` contains the HTTP method used
- This conditional allows one function to handle both displaying and processing the form

**Line 12: `username = request.form['username']`**
- Retrieves the value of the 'username' field from the submitted form
- `request.form` is a dictionary-like object containing form data
- The key `'username'` must match the `name` attribute in the HTML form input

**Line 13: `password = request.form['password']`**
- Retrieves the value of the 'password' field from the submitted form
- Similar to username extraction
- **Security note**: In production, passwords should never be displayed or logged in plain text

**Line 14: `return f"Username: {username}, Password: {password}"`**
- Uses an f-string (formatted string literal) to create a response
- Displays the submitted credentials back to the user
- **Important**: This is for demonstration only; real applications would authenticate against a database

**Line 15: `return render_template('login.html')`**
- Executes when the request method is GET (user navigates to the page)
- `render_template()` searches for 'login.html' in the 'templates' folder
- Returns the rendered HTML to the client's browser
- Flask automatically looks for templates in a folder named 'templates' in the application root

**Line 17-18: Application runner**
- Same as Experiment 2
- Starts the development server when the script is run directly

---

### Step 4: Create a Requirements File

```bash
touch requirements.txt
```

---

### Step 5: Add the Following Content to 'requirements.txt'

```
Flask==2.3.0
```

**Explanation:**
- Same as Experiment 2
- Flask is the only external dependency needed
- The templating engine (Jinja2) is included with Flask

---

### Step 6: Create a Dockerfile

```bash
touch Dockerfile
```

---

### Step 7: Add the Following Content to the "Dockerfile"

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

**Explanation:**
- Identical to Experiment 2's Dockerfile
- The `COPY . .` command will now also copy the 'templates' directory
- Docker preserves directory structure when copying

---

### Step 8: Create a Directory Named "templates"

```bash
mkdir templates
cd templates
```

**Explanation:**
- Flask follows a convention-over-configuration approach
- By default, Flask looks for HTML templates in a folder named 'templates'
- This folder must be at the same level as `app.py`
- **Why this matters**: Flask's `render_template()` function automatically searches this directory

---

### Step 9: Create a File "login.html"

```bash
touch login.html
```

**Explanation:**
- Creates the HTML template file for our login page
- This file will be rendered by Flask when users visit `/login`

---

### Step 10: Add the Following Content in the "login.html" File

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Page</title>
</head>
<body>
    <h1>Login</h1>
    <form method="POST" action="/login">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required><br><br>
        
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required><br><br>
        
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

**Detailed Line-by-Line Explanation:**

**Line 1: `<!DOCTYPE html>`**
- Document type declaration that tells browsers this is an HTML5 document
- Ensures the browser renders the page in standards mode
- Must be the very first line in an HTML document

**Line 2: `<html lang="en">`**
- Root element of the HTML document
- `lang="en"` specifies the document language is English
- Helps search engines and screen readers

**Line 3-7: `<head>` section**
- Contains metadata about the document
- Information here is not displayed on the page itself

**Line 4: `<meta charset="UTF-8">`**
- Specifies the character encoding for the document
- UTF-8 supports all characters and symbols from all languages
- Essential for proper text rendering

**Line 5: `<meta name="viewport" content="width=device-width, initial-scale=1.0">`**
- Controls the page's dimensions and scaling on mobile devices
- `width=device-width`: Makes the width match the device's screen width
- `initial-scale=1.0`: Sets the initial zoom level
- **Important for responsive design**: Ensures the page looks good on all devices

**Line 6: `<title>Login Page</title>`**
- Sets the title displayed in the browser tab
- Also used by search engines and when bookmarking

**Line 8-20: `<body>` section**
- Contains all visible content on the page

**Line 9: `<h1>Login</h1>`**
- Displays "Login" as a level-1 heading
- The largest and most important heading tag
- Good for SEO and accessibility

**Line 10: `<form method="POST" action="/login">`**
- Creates an HTML form for user input
- `method="POST"`: Specifies that form data should be sent via HTTP POST
  - POST is more secure for sensitive data like passwords
  - Data is sent in the request body, not in the URL
- `action="/login"`: Specifies where to send the form data when submitted
  - Points to our Flask `/login` route
  - The form submits to the same URL we're on

**Line 11: `<label for="username">Username:</label>`**
- Creates a text label for the username input field
- `for="username"`: Associates this label with the input element that has `id="username"`
- **Accessibility benefit**: Users can click the label to focus the input field
- Screen readers announce the label when the input is focused

**Line 12: `<input type="text" id="username" name="username" required>`**
- Creates a text input field for the username
- `type="text"`: Specifies a single-line text input
- `id="username"`: Unique identifier for this element
  - Used by the label's `for` attribute
  - Can be used for CSS styling or JavaScript
- `name="username"`: The key used when form data is submitted
  - **Critical**: This is how Flask's `request.form['username']` retrieves the value
  - Must match the key used in the Python code
- `required`: HTML5 attribute that prevents form submission if this field is empty
  - Provides client-side validation
  - Browser will show an error message if the user tries to submit without filling this field
- `<br><br>`: Two line breaks for vertical spacing

**Line 14: `<label for="password">Password:</label>`**
- Label for the password input field
- Same principle as the username label

**Line 15: `<input type="password" id="password" name="password" required>`**
- Creates a password input field
- `type="password"`: Masks the entered characters (shows dots or asterisks)
  - **Security feature**: Prevents shoulder-surfing
  - The actual value is still sent to the server; masking is only visual
- Other attributes work the same as the username field

**Line 17: `<button type="submit">Login</button>`**
- Creates a submit button for the form
- `type="submit"`: Indicates this button submits the form
  - When clicked, triggers form validation and submission
  - Sends a POST request to `/login` with the form data
- **Note**: Could also use `<input type="submit" value="Login">` with the same effect

**Line 18-20: Closing tags**
- `</form>`: Closes the form element
- `</body>`: Closes the body section
- `</html>`: Closes the HTML document

---

### Step 11: Build and Run the Docker Container

**Navigate back to the project root:**
```bash
cd ..
```

**Build the Docker image:**
```bash
docker build -t flask-login-microservice .
```

**Explanation:**
- Same build process as Experiment 2
- Docker will now include the `templates` directory in the image
- Tags the image as "flask-login-microservice"

**Run the container:**
```bash
docker run -d -p 5000:5000 flask-login-microservice
```

**Explanation:**
- Starts the container in detached mode
- Maps port 5000 for external access

**Access the application:**
- Home page: `http://localhost:5000` (displays welcome message)
- Login page: `http://localhost:5000/login` (displays login form)
- Submit the form to see the credentials displayed

---

### Step 13: Stop and Remove the Container

```bash
docker ps
docker stop <container_id>
docker rm <container_id>
```

**Explanation:**
- Same cleanup process as Experiment 2
- Always clean up containers when done to free resources

---

## Lab Experiment 4: Login Microservice on AWS

### Objective
Deploy the login microservice to AWS EC2, demonstrating how to run containerized applications in the cloud. This experiment introduces cloud deployment and security group configuration.

### Prerequisites
- AWS account with EC2 access
- SSH key pair for EC2 instance
- Docker installed on EC2 instance
- Basic understanding of AWS services

---

### Project Structure

```
flask-login-microservice/
├── Dockerfile
├── requirements.txt
├── app.py
└── templates/
    ├── login.html
    └── error.html
```

**Explanation:**
- This structure shows all files needed for deployment
- `error.html` is added to handle invalid login attempts
- Organized structure makes deployment and maintenance easier

---

### Complete Application Code

#### app.py

```python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/')
def home():
    return "Welcome to the AWS Login Microservice!"

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        
        # Simple validation (in production, use proper authentication)
        if username == 'admin' and password == 'password123':
            return f"<h1>Welcome, {username}!</h1>"
        else:
            return render_template('error.html')
    
    return render_template('login.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Key Differences from Experiment 3:**

**Lines 15-19: Authentication logic**
```python
if username == 'admin' and password == 'password123':
    return f"<h1>Welcome, {username}!</h1>"
else:
    return render_template('error.html')
```
- Adds basic credential checking
- Compares submitted username and password against hardcoded values
- **Success case**: Returns a personalized welcome message
- **Failure case**: Renders an error page
- **Production note**: Real applications should:
  - Hash passwords using bcrypt or Argon2
  - Store credentials in a secure database
  - Implement rate limiting to prevent brute force attacks
  - Use HTTPS to encrypt data in transit

---

#### requirements.txt

```
flask
```

**Explanation:**
- Simplified to just `flask` without version pinning
- Will install the latest version available
- **Best practice**: In production, always pin versions for stability

---

#### Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

**Explanation:**
- Identical to previous experiments
- This consistency demonstrates the power of Docker
- The same Dockerfile works locally and in the cloud

---

#### templates/login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AWS Login Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        .login-container {
            background: white;
            padding: 2rem;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            width: 300px;
        }
        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 1.5rem;
        }
        input {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
        }
        button {
            width: 100%;
            padding: 10px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background: #5568d3;
        }
    </style>
</head>
<body>
    <div class="login-container">
        <h1>Login</h1>
        <form method="POST" action="/login">
            <input type="text" name="username" placeholder="Username" required>
            <input type="password" name="password" placeholder="Password" required>
            <button type="submit">Login</button>
        </form>
    </div>
</body>
</html>
```

**Key Enhancements:**

**Lines 7-50: `<style>` section**
- Adds CSS styling directly in the HTML (inline styles)
- **Production note**: Typically, CSS would be in separate files

**Body styling (lines 8-16):**
- `font-family: Arial, sans-serif`: Sets a clean, readable font
- `display: flex`: Enables flexbox layout
- `justify-content: center`: Centers content horizontally
- `align-items: center`: Centers content vertically
- `height: 100vh`: Makes body full viewport height (100% of visible screen)
- `margin: 0`: Removes default margins
- `background: linear-gradient(...)`: Creates an attractive gradient background
  - Goes from purple-blue (#667eea) to deeper purple (#764ba2)
  - `135deg`: Diagonal gradient angle

**Login container styling (lines 17-23):**
- `background: white`: White background for the form
- `padding: 2rem`: Internal spacing (2 × root font size)
- `border-radius: 10px`: Rounded corners
- `box-shadow`: Adds depth with a subtle shadow
  - `0 4px 6px`: Horizontal offset, vertical offset, blur radius
  - `rgba(0, 0, 0, 0.1)`: Black color with 10% opacity
- `width: 300px`: Fixed width for consistent appearance

**Input styling (lines 29-36):**
- `width: 100%`: Makes inputs fill container width
- `padding: 10px`: Internal spacing for comfortable text entry
- `margin: 10px 0`: Vertical spacing between inputs
- `border: 1px solid #ddd`: Light gray border
- `box-sizing: border-box`: Includes padding in width calculation

**Button styling (lines 37-46):**
- `background: #667eea`: Purple background matching the theme
- `cursor: pointer`: Shows hand cursor on hover
- **Hover state (lines 45-47)**: Darker purple when mouse hovers
  - Provides visual feedback to users
  - Improves user experience

**Form changes (line 53):**
- `placeholder="Username"`: Shows hint text inside the input
- Cleaner appearance without separate labels

---

#### templates/error.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Error</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
        }
        .error-container {
            background: white;
            padding: 2rem;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            text-align: center;
            width: 300px;
        }
        h1 {
            color: #f5576c;
            margin-bottom: 1rem;
        }
        p {
            color: #666;
            margin-bottom: 1.5rem;
        }
        a {
            display: inline-block;
            padding: 10px 20px;
            background: #f5576c;
            color: white;
            text-decoration: none;
            border-radius: 5px;
        }
        a:hover {
            background: #e04455;
        }
    </style>
</head>
<body>
    <div class="error-container">
        <h1>Login Failed!</h1>
        <p>Invalid username or password.</p>
        <a href="/login">Try Again</a>
    </div>
</body>
</html>
```

**Explanation:**

**Purpose:**
- Displayed when login credentials are incorrect
- Provides user feedback and navigation back to login

**Styling differences:**
- Red/pink color scheme (`#f5576c`) to indicate error
- Different gradient background for visual distinction
- Error-specific messaging

**Line 50: `<a href="/login">Try Again</a>`**
- Creates a link back to the login page
- Styled as a button for better UX
- Allows users to easily retry authentication

---

### Deployment Steps

#### Step 1: Build the Docker Image

```bash
docker build -t flask-login-microservice .
```

**Explanation:**
- Execute this command in the project directory (where Dockerfile is located)
- Creates a Docker image with all application files and dependencies
- The image can be deployed to any Docker-enabled environment

---

#### Step 2: Run the Container

```bash
docker run -d -p 5000:5000 flask-login-microservice
```

**Explanation:**
- Starts the container in detached mode
- Maps port 5000 from the container to port 5000 on the EC2 instance
- The application is now running but not yet accessible from the internet

---

#### Step 3: Update Your EC2 Security Group

**What is a Security Group?**
- A virtual firewall that controls inbound and outbound traffic to your EC2 instance
- Acts as the first line of defense for your cloud resources
- Rules are stateful: if you allow incoming traffic, the response is automatically allowed

**Steps to Configure:**

1. **Navigate to EC2 Dashboard in AWS Console**
   - Log into your AWS account
   - Go to Services → EC2
   - Select "Security Groups" from the left sidebar

2. **Select Your Instance's Security Group**
   - Find the security group attached to your EC2 instance
   - Click on the security group ID

3. **Add Inbound Rule:**
   - Click "Edit inbound rules"
   - Click "Add rule"
   - Configure the rule:
     - **Type**: Custom TCP Rule
     - **Protocol**: TCP
     - **Port Range**: 5000
     - **Source**: 
       - `0.0.0.0/0` (for testing - allows access from anywhere)
       - OR your specific IP address (more secure)
   - Click "Save rules"

**Security Configuration Options:**

**Option 1: Open to All (Testing Only)**
```
Type: Custom TCP
Port: 5000
Source: 0.0.0.0/0
Description: Allow HTTP traffic on port 5000 from anywhere
```

**Explanation:**
- `0.0.0.0/0` means "any IP address"
- **Warning**: This makes your application accessible to anyone on the internet
- **Use only for testing**: Never use this in production with sensitive data
- Exposes your application to potential security risks

**Option 2: Restrict to Your IP (Recommended)**
```
Type: Custom TCP
Port: 5000
Source: <Your IP Address>/32
Description: Allow HTTP traffic on port 5000 from my IP only
```

**Explanation:**
- Replace `<Your IP Address>` with your actual public IP
- `/32` means exactly this one IP address (CIDR notation)
- **More secure**: Only you can access the application
- **Note**: Your IP may change if you have a dynamic IP from your ISP

**Option 3: Restrict to IP Range (Enterprise)**
```
Type: Custom TCP
Port: 5000
Source: 10.0.0.0/8
Description: Allow HTTP traffic on port 5000 from corporate network
```

**Explanation:**
- Useful for restricting access to specific networks (like your company's network)
- The `/8` notation specifies a range of IP addresses

---

#### Step 4: Access the Login Page

**Find Your EC2 Public IP:**
1. In the EC2 Dashboard, click "Instances"
2. Select your instance
3. Find the "Public IPv4 address" in the instance details
4. Example: `54.123.45.67`

**Access the Application:**
```
http://<your-ec2-public-ip>:5000/login
```

**Example:**
```
http://54.123.45.67:5000/login
```

**What Happens When You Visit:**
1. Your browser sends an HTTP GET request to the EC2 instance
2. The request passes through the security group firewall
3. Port 5000 is forwarded from the EC2 instance to the Docker container
4. Flask receives the request and renders `login.html`
5. The HTML is sent back through the same path to your browser

**Testing the Application:**

1. **Access the home page:**
   - Navigate to: `http://<your-ec2-public-ip>:5000/`
   - Should display: "Welcome to the AWS Login Microservice!"

2. **Access the login page:**
   - Navigate to: `http://<your-ec2-public-ip>:5000/login`
   - Should display the styled login form

3. **Test successful login:**
   - Enter username: `admin`
   - Enter password: `password123`
   - Click "Login"
   - Should display: "Welcome, admin!"

4. **Test failed login:**
   - Enter any incorrect credentials
   - Click "Login"
   - Should display the error page with "Login Failed!"
   - Click "Try Again" to return to the login page

---

### AWS Deployment Workflow Diagram

```
Developer's Machine → Build Docker Image → Upload to EC2
                                              ↓
                                    Run Container on EC2
                                              ↓
                                    Configure Security Group
                                              ↓
Internet Users ← Access Application ← Port 5000 Exposed
```

---

### Troubleshooting Common Issues

#### Issue 1: Cannot Access the Application

**Symptom:** Browser shows "This site can't be reached" or times out

**Possible Causes and Solutions:**

1. **Security Group Not Configured:**
   - Verify port 5000 is open in the security group
   - Check that the source IP is correct
   - Wait a few seconds after modifying security groups

2. **Container Not Running:**
   ```bash
   docker ps
   ```
   - If the container isn't listed, check logs:
   ```bash
   docker logs <container_id>
   ```

3. **Flask Not Binding to 0.0.0.0:**
   - Ensure `app.run(host='0.0.0.0', port=5000)` in app.py
   - `localhost` or `127.0.0.1` won't work in containers

4. **Wrong Port:**
   - Verify you're using port 5000 in the URL
   - Check that Docker port mapping is correct: `-p 5000:5000`

5. **EC2 Instance Stopped:**
   - Check EC2 instance state in AWS Console
   - Ensure the instance is "Running"

---

#### Issue 2: Login Always Fails

**Symptom:** Always redirected to error page, even with correct credentials

**Possible Causes and Solutions:**

1. **Credentials Hardcoded Incorrectly:**
   - Verify `username == 'admin' and password == 'password123'` in app.py
   - Check for typos or extra spaces

2. **Form Data Not Received:**
   - Ensure form inputs have correct `name` attributes
   - Check that form method is POST
   - Verify `action="/login"` in the form tag

3. **Template Not Found:**
   - Ensure `error.html` exists in the `templates/` folder
   - Check file permissions on EC2

---

#### Issue 3: CSS Not Loading

**Symptom:** Login page appears unstyled

**Possible Causes and Solutions:**

1. **HTML File Not Updated:**
   - Rebuild the Docker image after modifying HTML:
   ```bash
   docker build -t flask-login-microservice .
   docker stop <old_container_id>
   docker rm <old_container_id>
   docker run -d -p 5000:5000 flask-login-microservice
   ```

2. **Browser Cache:**
   - Hard refresh the page: Ctrl+F5 (Windows) or Cmd+Shift+R (Mac)
   - Clear browser cache

---

#### Issue 4: Container Exits Immediately

**Symptom:** Container appears in `docker ps -a` but not in `docker ps`

**Solution:**
```bash
docker logs <container_id>
```

**Common Errors:**

1. **ModuleNotFoundError: No module named 'flask':**
   - Flask not installed in the image
   - Verify `requirements.txt` includes Flask
   - Rebuild the image

2. **SyntaxError in Python Code:**
   - Check app.py for syntax errors
   - Ensure proper indentation (Python is whitespace-sensitive)

3. **FileNotFoundError: templates/login.html:**
   - Verify `templates/` folder exists
   - Ensure HTML files are in the correct location
   - Rebuild the image to include templates

---

### Best Practices for Production Deployment

#### Security Enhancements

1. **Use HTTPS:**
   - Set up an SSL/TLS certificate
   - Use a reverse proxy like Nginx or AWS Application Load Balancer
   - Encrypt data in transit

2. **Implement Proper Authentication:**
   ```python
   from werkzeug.security import check_password_hash
   from flask_login import LoginManager
   ```
   - Hash passwords using bcrypt or Argon2
   - Store user credentials in a database
   - Use session management libraries like Flask-Login
   - Implement CSRF protection

3. **Environment Variables:**
   ```python
   import os
   SECRET_KEY = os.environ.get('SECRET_KEY')
   DATABASE_URL = os.environ.get('DATABASE_URL')
   ```
   - Never hardcode credentials
   - Use environment variables or AWS Secrets Manager
   - Keep sensitive configuration separate from code

4. **Rate Limiting:**
   ```python
   from flask_limiter import Limiter
   limiter = Limiter(app, key_func=get_remote_address)
   
   @app.route('/login', methods=['POST'])
   @limiter.limit("5 per minute")
   def login():
       # ...
   ```
   - Prevent brute force attacks
   - Limit login attempts per IP address

5. **Input Validation:**
   ```python
   from flask_wtf import FlaskForm
   from wtforms import StringField, PasswordField
   from wtforms.validators import DataRequired, Length
   ```
   - Validate and sanitize all user inputs
   - Prevent SQL injection and XSS attacks
   - Use form validation libraries

#### Scalability Considerations

1. **Use Docker Compose for Multi-Container Apps:**
   ```yaml
   version: '3'
   services:
     web:
       build: .
       ports:
         - "5000:5000"
     db:
       image: postgres
       environment:
         POSTGRES_PASSWORD: example
   ```
   - Separate application components
   - Manage multiple services together

2. **Container Orchestration:**
   - Use AWS ECS (Elastic Container Service) for production
   - Or Kubernetes for more complex deployments
   - Enables automatic scaling and load balancing

3. **Database Integration:**
   - Don't store data in containers (ephemeral)
   - Use AWS RDS or DynamoDB for persistent storage
   - Implement proper database connection pooling

4. **Logging and Monitoring:**
   ```python
   import logging
   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)
   
   @app.route('/login', methods=['POST'])
   def login():
       logger.info(f"Login attempt for user: {request.form.get('username')}")
   ```
   - Use AWS CloudWatch for log aggregation
   - Monitor application performance and errors
   - Set up alerts for critical issues

5. **CI/CD Pipeline:**
   - Automate building and deployment
   - Use AWS CodePipeline or GitHub Actions
   - Implement automated testing before deployment

---

## Key Concepts Summary

### Microservices Architecture

**What is a Microservice?**
- A small, independent service that performs a specific business function
- Communicates with other services via APIs
- Can be developed, deployed, and scaled independently

**Benefits:**
- **Scalability**: Scale individual services based on demand
- **Flexibility**: Use different technologies for different services
- **Resilience**: Failure of one service doesn't bring down the entire application
- **Development Speed**: Teams can work on services independently

**Our Examples:**
- Experiment 2: Simple greeting microservice
- Experiment 3: Authentication microservice
- In production, these could be combined with other services (e.g., user management, data processing)

---

### Docker Containerization

**What is Docker?**
- A platform for developing, shipping, and running applications in containers
- Containers package application code with all dependencies
- Ensures consistency across different environments

**Key Components:**

1. **Docker Image:**
   - A lightweight, standalone, executable package
   - Contains everything needed to run the application
   - Built from a Dockerfile
   - Immutable (doesn't change once created)

2. **Docker Container:**
   - A running instance of a Docker image
   - Isolated from the host system and other containers
   - Can be started, stopped, and deleted
   - Multiple containers can run from the same image

3. **Dockerfile:**
   - A text file with instructions to build an image
   - Defines the environment, dependencies, and commands
   - Version-controllable and reproducible

**Docker Workflow:**
```
Write Dockerfile → Build Image → Run Container → Deploy
```

---

### Flask Web Framework

**Why Flask?**
- Lightweight and flexible Python web framework
- Easy to learn and use
- Perfect for microservices and APIs
- Extensive ecosystem of extensions

**Core Concepts:**

1. **Routes:** Map URLs to Python functions
2. **Templates:** HTML files with dynamic content
3. **Request/Response:** Handle HTTP communication
4. **Development Server:** Built-in server for testing

**Flask vs. Other Frameworks:**
- **Flask**: Minimal, flexible, microframework
- **Django**: Full-featured, opinionated, includes ORM and admin panel
- **FastAPI**: Modern, fast, API-focused with automatic documentation

---

### Cloud Deployment (AWS)

**EC2 (Elastic Compute Cloud):**
- Virtual servers in the cloud
- Choose instance type based on CPU, memory, and storage needs
- Pay only for what you use
- Can be stopped/started as needed

**Security Groups:**
- Virtual firewalls for EC2 instances
- Control inbound and outbound traffic
- Essential for protecting cloud resources

**Benefits of Cloud Deployment:**
- **Availability**: 99.99% uptime SLA
- **Global Reach**: Deploy in multiple regions
- **Cost-Effective**: No upfront hardware costs
- **Elasticity**: Scale resources up or down based on demand

---

## Learning Outcomes

After completing these experiments, students should be able to:

1. **Understand Microservice Architecture:**
   - Explain the principles of microservices
   - Identify when to use microservices vs. monolithic architecture
   - Design simple microservice applications

2. **Work with Docker:**
   - Write Dockerfiles for Python applications
   - Build and run Docker containers
   - Understand Docker networking and port mapping
   - Manage container lifecycle

3. **Develop with Flask:**
   - Create web applications with Flask
   - Implement routing and view functions
   - Use templates for dynamic HTML
   - Handle form submissions

4. **Deploy to the Cloud:**
   - Launch and configure EC2 instances
   - Set up security groups
   - Deploy containerized applications to AWS
   - Troubleshoot common deployment issues

5. **Apply Security Best Practices:**
   - Understand basic authentication mechanisms
   - Configure firewalls and access controls
   - Recognize security vulnerabilities
   - Plan for production-grade security

---

## Further Exploration

### Next Steps for Students

1. **Add a Database:**
   - Integrate PostgreSQL or MongoDB
   - Store user credentials securely
   - Implement CRUD operations

2. **Create Multiple Microservices:**
   - User service, authentication service, data service
   - Use Docker Compose to manage them
   - Implement service-to-service communication

3. **Implement RESTful APIs:**
   - Create JSON endpoints
   - Use HTTP methods correctly (GET, POST, PUT, DELETE)
   - Add API documentation with Swagger

4. **Add Testing:**
   - Write unit tests with pytest
   - Implement integration tests
   - Set up automated testing in CI/CD

5. **Enhance the UI:**
   - Use CSS frameworks (Bootstrap, Tailwind)
   - Add JavaScript for interactivity
   - Implement responsive design

6. **Production Deployment:**
   - Use Gunicorn or uWSGI instead of Flask development server
   - Set up Nginx as a reverse proxy
   - Implement SSL/TLS certificates
   - Use container orchestration (ECS, Kubernetes)

---

## Additional Resources

### Official Documentation
- **Flask**: https://flask.palletsprojects.com/
- **Docker**: https://docs.docker.com/
- **AWS EC2**: https://docs.aws.amazon.com/ec2/

### Recommended Reading
- "Building Microservices" by Sam Newman
- "Docker Deep Dive" by Nigel Poulton
- "Flask Web Development" by Miguel Grinberg

### Online Courses
- Docker Mastery on Udemy
- AWS Certified Solutions Architect on A Cloud Guru
- Python and Flask Bootcamp on Coursera

### Practice Projects
- Build a Todo List microservice
- Create a Weather API microservice
- Develop a User Management system
- Implement a Blog platform with microservices

---

## Conclusion

These lab experiments provide a foundation in modern application development and deployment. By mastering containerization with Docker, building microservices with Flask, and deploying to AWS, students gain practical skills that are highly valuable in the industry.

The journey from a simple "Hello, World!" application to a cloud-deployed login service demonstrates the entire development lifecycle. Each experiment builds upon the previous one, gradually introducing more complex concepts and real-world scenarios.

Remember: The key to mastering these technologies is practice. Experiment with the code, break things, fix them, and most importantly, understand why each component works the way it does.

**Happy Learning! 🚀**
