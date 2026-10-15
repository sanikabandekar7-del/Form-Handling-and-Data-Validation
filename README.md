# Form-Handling-and-Data-Validation
Developed backend functionality to process form submissions, validate user inputs, and handle errors
Form Handling and Data Validation — Student Management API
This project extends a Student Management REST API built with Node.js, Express, and MySQL. It demonstrates how a backend receives form data, validates user input, and returns useful error messages before processing database operations.
Features
- Process JSON form submissions using Express
- Validate registration fields: name, email, and password
- Validate login fields: email and password
- Validate student fields: name and course
- Validate student IDs in URL parameters
- Return clear 400 Bad Request responses for invalid input
- Trim whitespace from names, course values, and email addresses; convert emails to lowercase
- Work alongside MySQL, JWT authentication, and role-based authorization
Technologies
- Node.js
- Express.js
- MySQL
- mysql2
- bcryptjs
- jsonwebtoken
- dotenv
- Postman
Prerequisites
Install Node.js and MySQL Server. The existing Student Management API should already be configured and connected to its MySQL database.
Setup
1. Open a terminal in the project folder (for example, student-api).
2. Install project dependencies:
   npm install
   If required packages are not installed yet:
   npm install express mysql2 bcryptjs jsonwebtoken dotenv
3. Confirm that .env exists beside server.js and contains your local database settings:
   DB_HOST=localhost
   DB_USER=root
   DB_PASSWORD=your_mysql_password
   DB_NAME=student_db
   JWT_SECRET=replace_with_a_long_private_secret
   PORT=3000
   Replace the example values with your own local settings. Never share real passwords or JWT secrets.
4. Start the API:
   node server.js
   Keep the terminal open while testing.
Validation Rules
Input	Validation
Registration name	Must be a string containing 2–100 characters
Registration email	Must match a basic email format
Registration password	Must contain 8–72 characters
Login email	Required and must match a basic email format
Login password	Required
Student name	Must be a string containing 2–100 characters
Student course	Must be a string containing 2–100 characters
Student ID	Must be a positive integer in the URL


Whitespace is trimmed from names, course names, and email addresses. Emails are converted to lowercase. Passwords are not trimmed because whitespace can be part of a password.
Validation Middleware
Declare these functions after Express setup and before the routes that use them:
- validateRegistration — checks registration data.
- validateLogin — checks login data.
- validateStudent — checks student name and course.
- validateStudentId — checks IDs in route parameters.
Attach middleware to your existing route declarations instead of creating duplicate routes. For example:
app.post("/api/auth/register", validateRegistration, existingRegisterHandler);
app.post("/api/auth/login", validateLogin, existingLoginHandler);
app.post("/api/students", validateStudent, existingCreateStudentHandler);
app.put(
  "/api/students/:id",
  validateStudentId,
  validateStudent,
  existingUpdateStudentHandler
);
These lines illustrate middleware order; existingRegisterHandler and the other names are placeholders, not literal functions to paste into your project. Keep your current route handlers and database logic. Retain authenticateToken on protected student endpoints and requireRole("admin") on the DELETE route if deletion is admin-only.
API Routes
The following paths are expected; actual routes depend on the handlers implemented in server.js.
Method	Endpoint	Purpose	Validation / access
POST	/api/auth/register	Register an account	Registration validation
POST	/api/auth/login	Log in	Login validation
GET	/api/students	List students	JWT required if configured
GET	/api/students/:id	Get one student	ID validation and JWT if configured
POST	/api/students	Add a student	Student validation and JWT if configured
PUT	/api/students/:id	Update a student	ID + student validation and JWT if configured
DELETE	/api/students/:id	Delete a student	ID validation, JWT, and admin role if configured


Test with Postman
For JSON requests, select Body → raw → JSON.
1. Valid registration
POST http://localhost:3000/api/auth/register
{
  "name": "Test User",
  "email": "testuser@example.com",
  "password": "TestPass123!"
}
Expected: registration succeeds, normally with 201 Created, unless the email is already registered.
2. Missing name
POST http://localhost:3000/api/auth/register
{
  "email": "testuser@example.com",
  "password": "TestPass123!"
}
Expected: 400 Bad Request with a message that name, email, and password are required.
3. Invalid email
{
  "name": "Test User",
  "email": "wrong-email",
  "password": "TestPass123!"
}
Expected: 400 Bad Request with a valid-email message.
4. Password too short
{
  "name": "Test User",
  "email": "anotheruser@example.com",
  "password": "123"
}
Expected: 400 Bad Request with a password-length message.
5. Add a student
POST http://localhost:3000/api/students
{
  "name": "Aarav Sharma",
  "course": "Computer Science"
}
If student endpoints require authentication, select Authorization → Bearer Token and paste a current login token.
Expected: the student is created if the request is valid and the database operation succeeds.
6. Invalid student ID
Request:
DELETE http://localhost:3000/api/students/abc
Expected: 400 Bad Request for an invalid ID, provided validateStudentId is attached and any earlier authentication middleware allows the request to reach it.
Common Errors
- 400 Bad Request: A required field is missing or input fails validation. Correct the request body or URL.
- 401 Unauthorized: The token is missing, invalid, or expired. Log in again and use the latest token.
- 403 Forbidden: The account does not have the required role. Verify the account role and log in again to get a token containing the current role.
- 409 Conflict: May occur when registering an email that already exists, depending on the registration handler.
- 500 Internal Server Error: Check the VS Code terminal for the database or server error. Confirm .env settings and table columns.
Security Notes
- Store passwords as bcryptjs hashes, never plain text.
- Keep .env out of version control by adding it to .gitignore.
- Keep the JWT secret private and use HTTPS in production.
- Validate all untrusted input on the server, even if a frontend also validates it.
- Use parameterized SQL queries rather than string concatenation.
- Return useful client errors without exposing database credentials or stack traces.
- Review authorization on every protected route.
Learning Outcomes
This implementation demonstrates:
- Express middleware and request processing
- Server-side input validation
- Handling invalid form submissions
- HTTP status codes and structured error responses
- Integration with MySQL-backed API routes
- Compatibility with JWT authentication and role-based authorization
