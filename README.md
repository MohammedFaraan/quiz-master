# QuizMaster – Online Quiz Management System

## 1. Project Overview

**Project Title:** QuizMaster – Online Quiz Management System

**Technology Stack:**

- Frontend: React.js
- Backend: Python FastAPI
- Database: MySQL
- ORM: SQLAlchemy
- API Testing: Postman

## 2. Problem Statement

Traditional classroom-based quizzes and assessments often require teachers to manually prepare questions, conduct assessments, evaluate student responses, and calculate scores.

This process can be time-consuming, especially when multiple quizzes and students are involved. Maintaining quiz records, student responses, and assessment results manually can also lead to errors and difficulties in tracking student performance.

There is a need for a centralized web-based platform that simplifies quiz creation, student participation, automatic evaluation, and result management.

## 3. Proposed Solution

QuizMaster is a web-based quiz management system that allows teachers to create and manage quizzes consisting of multiple-choice questions (MCQs), while students can enroll in available quizzes and attempt them within a specified time limit.

The system automatically evaluates submitted answers, calculates scores, and stores the results in a MySQL database.

### Main functionalities

**Teacher:**

- Register and log in.
- Create, view, update, and delete quizzes.
- Add, update, and delete questions.
- Add multiple-choice options and specify the correct answer.
- Publish or deactivate quizzes.
- View enrolled students and their results.

**Student:**

- Register and log in.
- Browse available quizzes.
- Enroll in quizzes.
- Start an enrolled quiz.
- Attempt MCQ questions within the specified time limit.
- Submit answers.
- View quiz scores and results.

**System:**

- Automatically evaluate submitted answers.
- Calculate scores.
- Enforce quiz time limits.
- Prevent duplicate enrollments.
- Maintain quiz attempts and student responses.

## 4. Project Scope and Limitations

The initial implementation will support:

- Only multiple-choice questions.
- Four options per question.
- One correct option per question.
- One attempt per student per quiz.
- Fixed question order.
- Automatic score calculation.
- Quiz duration configured by the teacher.

The following features are excluded to maintain a manageable project scope:

- Descriptive questions.
- Negative marking.
- Random question generation.
- Question shuffling.
- Live proctoring.
- Payment integration.
- Notifications and real-time communication.

## 5. Database Schema

The database will contain eight primary tables.

### 5.1 Users

Stores teacher and student account information.

| Column     | Type                      | Constraint                  |
| ---------- | ------------------------- | --------------------------- |
| user_id    | INT                       | Primary Key, Auto Increment |
| name       | VARCHAR(100)              | NOT NULL                    |
| email      | VARCHAR(150)              | UNIQUE, NOT NULL            |
| password   | VARCHAR(255)              | NOT NULL                    |
| role       | ENUM('teacher','student') | NOT NULL                    |
| created_at | DATETIME                  | DEFAULT CURRENT_TIMESTAMP   |

### 5.2 Subjects

Stores the subjects under which quizzes are organized.

| Column       | Type         | Constraint                  |
| ------------ | ------------ | --------------------------- |
| subject_id   | INT          | Primary Key, Auto Increment |
| subject_name | VARCHAR(100) | UNIQUE, NOT NULL            |
| description  | TEXT         | NULL                        |

### 5.3 Quizzes

Stores quiz information.

| Column      | Type                               | Constraint                  |
| ----------- | ---------------------------------- | --------------------------- |
| quiz_id     | INT                                | Primary Key, Auto Increment |
| subject_id  | INT                                | Foreign Key → Subjects      |
| teacher_id  | INT                                | Foreign Key → Users         |
| title       | VARCHAR(150)                       | NOT NULL                    |
| description | TEXT                               | NULL                        |
| duration    | INT                                | NOT NULL                    |
| status      | ENUM('draft','published','closed') | DEFAULT 'draft'             |
| created_at  | DATETIME                           | DEFAULT CURRENT_TIMESTAMP   |

### 5.4 Questions

Stores individual questions belonging to quizzes.

| Column        | Type | Constraint                  |
| ------------- | ---- | --------------------------- |
| question_id   | INT  | Primary Key, Auto Increment |
| quiz_id       | INT  | Foreign Key → Quizzes       |
| question_text | TEXT | NOT NULL                    |
| marks         | INT  | DEFAULT 1                   |

### 5.5 Options

Stores answer options for individual questions.

| Column      | Type         | Constraint                  |
| ----------- | ------------ | --------------------------- |
| option_id   | INT          | Primary Key, Auto Increment |
| question_id | INT          | Foreign Key → Questions     |
| option_text | VARCHAR(255) | NOT NULL                    |
| is_correct  | BOOLEAN      | DEFAULT FALSE               |

### 5.6 Enrollments

Maintains student registrations for quizzes.

| Column        | Type     | Constraint                  |
| ------------- | -------- | --------------------------- |
| enrollment_id | INT      | Primary Key, Auto Increment |
| quiz_id       | INT      | Foreign Key → Quizzes       |
| student_id    | INT      | Foreign Key → Users         |
| enrolled_at   | DATETIME | DEFAULT CURRENT_TIMESTAMP   |

Unique constraint: (quiz_id, student_id).

### 5.7 Attempts

Stores information about student quiz attempts.

| Column        | Type                                      | Constraint                        |
| ------------- | ----------------------------------------- | --------------------------------- |
| attempt_id    | INT                                       | Primary Key, Auto Increment       |
| enrollment_id | INT                                       | Foreign Key → Enrollments, UNIQUE |
| start_time    | DATETIME                                  | NOT NULL                          |
| end_time      | DATETIME                                  | NULL                              |
| score         | INT                                       | DEFAULT 0                         |
| status        | ENUM('in_progress','submitted','expired') | DEFAULT 'in_progress'             |

### 5.8 Responses

Stores answers submitted by students.

| Column             | Type | Constraint                          |
| ------------------ | ---- | ----------------------------------- |
| response_id        | INT  | Primary Key, Auto Increment         |
| attempt_id         | INT  | Foreign Key → Attempts              |
| question_id        | INT  | Foreign Key → Questions             |
| selected_option_id | INT  | Foreign Key → Options, NULL allowed |

Unique constraint: (attempt_id, question_id).

## 6. Database Relationships

- Users (Teacher) → Quizzes: One-to-Many.
- Subjects → Quizzes: One-to-Many.
- Quizzes → Questions: One-to-Many.
- Questions → Options: One-to-Many.
- Users (Student) → Enrollments: One-to-Many.
- Quizzes → Enrollments: One-to-Many.
- Enrollments → Attempts: One-to-One.
- Attempts → Responses: One-to-Many.
- Questions → Responses: One-to-Many.
- Options → Responses: One-to-Many.

Many-to-many relationships:

- Students and Quizzes through Enrollments.
- Questions and Attempts through Responses.

## 7. Backend Architecture

The backend will be developed using FastAPI and organized into separate modules.

Suggested folder structure:

backend/

- app/
  - main.py
  - database.py
  - models/
  - schemas/
  - routers/
  - services/
  - dependencies/
  - utils/
- requirements.txt
- .env

Responsibilities:

- models/: SQLAlchemy database models.
- schemas/: Pydantic request and response schemas.
- routers/: API endpoints.
- services/: Business logic and evaluation.
- dependencies/: Database sessions and authentication dependencies.
- utils/: Shared helper functions.

## 8. API Endpoints

### Authentication

- POST /api/auth/register
- POST /api/auth/login
- GET /api/auth/me

### Subjects

- POST /api/subjects
- GET /api/subjects
- GET /api/subjects/{subject_id}
- PUT /api/subjects/{subject_id}
- DELETE /api/subjects/{subject_id}

### Quizzes

- POST /api/quizzes
- GET /api/quizzes
- GET /api/quizzes/{quiz_id}
- PUT /api/quizzes/{quiz_id}
- DELETE /api/quizzes/{quiz_id}
- PATCH /api/quizzes/{quiz_id}/publish

### Questions and Options

- POST /api/quizzes/{quiz_id}/questions
- GET /api/quizzes/{quiz_id}/questions
- PUT /api/questions/{question_id}
- DELETE /api/questions/{question_id}
- POST /api/questions/{question_id}/options
- PUT /api/options/{option_id}
- DELETE /api/options/{option_id}

### Enrollments

- POST /api/quizzes/{quiz_id}/enroll
- GET /api/students/me/enrollments
- GET /api/quizzes/{quiz_id}/enrollments

### Quiz Attempts

- POST /api/quizzes/{quiz_id}/start
- GET /api/attempts/{attempt_id}
- POST /api/attempts/{attempt_id}/submit
- GET /api/attempts/{attempt_id}/result

### Results

- GET /api/students/me/results
- GET /api/quizzes/{quiz_id}/results
- GET /api/teachers/me/dashboard

## 9. Quiz Attempt Workflow

1. Student enrolls in a published quiz.
2. Student clicks Start Quiz.
3. Backend verifies enrollment and checks whether an attempt already exists.
4. Backend records the start timestamp.
5. Backend calculates the deadline using the quiz duration.
6. Frontend displays the questions and starts the countdown.
7. Student selects answers.
8. Frontend submits selected option IDs.
9. Backend validates the attempt and deadline.
10. Backend evaluates answers using the correct options stored in MySQL.
11. Backend calculates and stores the final score.
12. Student receives the result.

The timer will be displayed on the frontend, but the backend will remain the authority for deadline validation.

## 10. Frontend Architecture

The frontend will use React.js with a component-based structure.

Suggested folder structure:

frontend/

- src/
  - components/
  - pages/
  - layouts/
  - services/
  - context/
  - hooks/
  - App.jsx
  - main.jsx

### Teacher Pages

- Teacher Dashboard
- Subject Management
- Quiz Management
- Create/Edit Quiz
- Question Management
- Enrolled Students
- Quiz Results

### Student Pages

- Student Dashboard
- Available Quizzes
- Enrolled Quizzes
- Quiz Attempt
- Quiz Result
- My Quiz History

### Shared Pages

- Login
- Registration
- Profile

## 11. Important Business Rules

- Only teachers can create and manage quizzes.
- Only students can enroll in and attempt quizzes.
- Only published quizzes are available for enrollment.
- Duplicate enrollment is not allowed.
- A student can attempt each quiz only once.
- A quiz must have questions before it can be published.
- Each question must have four options and exactly one correct answer.
- The backend must validate the quiz deadline.
- Correct answers must never be exposed through student-facing APIs before submission.
- Results must be calculated and stored by the backend.
- Published quiz questions cannot be modified once an attempt has started.

## 12. SQL Queries for DBMS Demonstration

The project will demonstrate the following SQL concepts:

1. CRUD operations on all major entities.
2. INNER JOIN – Display quizzes with their subject and teacher details.
3. Multiple JOINs – Display questions and their options.
4. JOIN – Display students enrolled in a particular quiz.
5. GROUP BY – Calculate the average score for each quiz.
6. HAVING – Find quizzes with more than a specified number of enrollments.
7. Subqueries – Find the highest-scoring student in each quiz.
8. NOT EXISTS – Find students who enrolled but never attempted a quiz.
9. Aggregate functions – Calculate total enrollments and average scores.
10. Transactions – Save responses and finalize an attempt consistently.

## 13. Testing

### Backend Testing

- Test all CRUD endpoints.
- Verify foreign key constraints.
- Test duplicate enrollment prevention.
- Test quiz attempt creation.
- Verify automatic score calculation.
- Test expired attempt submission.
- Test teacher/student authorization.

### Frontend Testing

- Verify registration and login.
- Verify teacher quiz creation.
- Verify question and option management.
- Verify student enrollment.
- Verify timer behavior.
- Verify quiz submission and result display.

### Database Testing

- Verify primary and foreign keys.
- Verify unique constraints.
- Verify data consistency.
- Test JOIN and aggregate queries.

## 14. Development Phases

**Phase 1: Planning and Database Design**

- Finalize requirements.
- Create ER diagram.
- Create MySQL database and tables.
- Define relationships and constraints.

**Phase 2: Backend Foundation**

- Set up FastAPI.
- Configure MySQL connection.
- Create SQLAlchemy models.
- Implement authentication.

**Phase 3: Teacher Functionality**

- Implement subject CRUD.
- Implement quiz CRUD.
- Implement question and option management.
- Implement publishing.

**Phase 4: Student Functionality**

- Implement quiz listing.
- Implement enrollment.
- Implement quiz attempt creation.
- Implement response submission.
- Implement timer validation and automatic evaluation.

**Phase 5: Frontend Development**

- Build teacher dashboard.
- Build student dashboard.
- Build quiz creation forms.
- Build quiz attempt interface.
- Build results pages.
- Connect frontend to backend APIs.

**Phase 6: Integration and Testing**

- Test complete teacher workflow.
- Test complete student workflow.
- Verify database relationships.
- Test SQL queries.
- Fix bugs and validate results.

**Phase 7: Final Demonstration**

- Prepare project documentation.
- Prepare ER diagram and schema.
- Prepare SQL query demonstrations.
- Demonstrate the complete application workflow.

## 15. Expected Outcome

The final system will provide a simple and effective platform for creating quizzes, managing questions, enrolling students, conducting timed MCQ assessments, automatically evaluating responses, and maintaining results.

The project will demonstrate practical applications of relational database concepts through a functional web application built with React, FastAPI, and MySQL.

## **16. Important implementation decisions**

Before starting development, I want us to finalize a few technical decisions so that we don't encounter unnecessary database or backend problems later.

### **A. Authentication**

Use JWT-based authentication.

- User registers with name, email, password and role.
- Passwords are hashed using `passlib` or a suitable password-hashing library.
- Login returns an access token.
- Protected APIs verify the token.
- Teachers and students have different permissions.

### **B. Quiz timer**

Use the following approach:

| Responsibility              | Handled by |
| --------------------------- | ---------- |
| Store quiz duration         | MySQL      |
| Record attempt start time   | FastAPI    |
| Calculate deadline          | FastAPI    |
| Display countdown           | React      |
| Detect countdown completion | React      |
| Validate actual deadline    | FastAPI    |
| Evaluate answers            | FastAPI    |

The backend remains authoritative. Even if a student refreshes the browser or changes their device clock, the deadline cannot be extended.

### **C. Protecting correct answers**

This is particularly important.

When students fetch questions, the API should return:

- Question text.
- Question ID.
- Available options.
- Option IDs.

It must NOT return `is_correct`.

Only the backend should access the correct answer during evaluation.

### **D. Database integrity**

Add these constraints:

| Constraint                                | Purpose                       |
| ----------------------------------------- | ----------------------------- |
| UNIQUE Users.email                        | Prevent duplicate accounts    |
| UNIQUE Enrollments(quiz_id, student_id)   | Prevent duplicate enrollment  |
| UNIQUE Attempts.enrollment_id             | Enforce one attempt           |
| UNIQUE Responses(attempt_id, question_id) | Prevent duplicate responses   |
| Foreign keys                              | Maintain relational integrity |
| NOT NULL                                  | Prevent missing required data |

Also validate that a selected option actually belongs to the question being answered.

### **E. Important transaction**

When a student submits a quiz, the backend should:

1. Verify the attempt and deadline.
2. Retrieve submitted responses.
3. Evaluate answers.
4. Calculate the score.
5. Update the attempt status and score.
6. Commit the transaction.

If something fails, roll back the transaction rather than leaving partially saved results.

## **17. Development order – What we should build first**

I recommend following this order rather than developing the frontend and backend simultaneously.

## **Development roadmap**

1. **Database design**

   Start here
   - Create MySQL database.
   - Create eight tables.
   - Establish foreign keys.
   - Add constraints.
   - Insert sample data.
   - Verify relationships using SQL queries.
2. **Backend setup**
   - Initialize FastAPI.
   - Configure MySQL connection.
   - Set up SQLAlchemy.
   - Create database models.
   - Create Pydantic schemas.
3. **Authentication and authorization**
   - Registration.
   - Login.
   - JWT authentication.
   - Teacher/student authorization.
4. **Teacher functionality**
   - Subject management.
   - Quiz CRUD.
   - Question CRUD.
   - Option CRUD.
   - Quiz publishing.
5. **Student functionality**
   - Browse quizzes.
   - Enroll in quizzes.
   - Start attempts.
   - Save responses.
   - Submit quizzes.
   - Calculate results.
6. **Frontend development**
   - React project setup.
   - Authentication pages.
   - Teacher dashboard.
   - Student dashboard.
   - Quiz creation interface.
   - Quiz attempt interface.
   - Results interface.
7. **Integration and final testing**
   - Connect frontend and backend.
   - Test all CRUD operations.
   - Test SQL JOIN queries.
   - Test timer expiration.
   - Verify score calculations.
   - Prepare DBMS demonstration.

## **18. Tools and dependencies**

These are the recommended packages to get started.

| Component           | Package / Tool       |
| ------------------- | -------------------- |
| Backend framework   | FastAPI              |
| ASGI server         | Uvicorn              |
| ORM                 | SQLAlchemy           |
| MySQL driver        | PyMySQL              |
| Data validation     | Pydantic             |
| Authentication      | python-jose or PyJWT |
| Password hashing    | pwdlib with Argon2   |
| Frontend            | React + Vite         |
| Routing             | React Router         |
| API requests        | Axios                |
| Styling             | Tailwind CSS         |
| API testing         | Postman              |
| Database management | MySQL Workbench      |
| Version control     | Git + GitHub         |
