# **Task Management System with User Authentication**

*Duration: 1 Week*
*Technologies: FastAPI, PostgreSQL, Streamlit*

### **Project Overview**:

You will build a **Task Management System** where users can register, log in, and manage their tasks. The system will support the following features:

* **User Authentication** (via JWT tokens)
* **Task CRUD Operations** (Create, Read, Update, Delete)
* **Task Categorization and Tagging**
* **Task Prioritization**
* **Task Reminders and Notifications**
* **Task History/Activity Log**
* **Task Search and Full-Text Search**

The backend will be implemented using **FastAPI** and **PostgreSQL** for the database, while the frontend will be built with **Streamlit** to provide a simple UI for users to interact with the system.

---

## **Backend Functionality (FastAPI + PostgreSQL)**

### **1. User Authentication**

* Implement user registration and login functionalities.
* Use **JWT tokens** for secure user authentication.
* Store user information (username, email, and hashed password) in the **users** table in PostgreSQL.

**Endpoints**:

* `POST /auth/register`: Register a new user.
* `POST /auth/login`: Log in and receive a JWT token.

### **2. Task Management**

Tasks can be created, updated, deleted, and viewed by users. Each task will be associated with a specific user.

**Task Schema**:

* `task_id`: UUID (Primary Key)
* `title`: String (Task title)
* `description`: Text (Task details)
* `status`: Enum (e.g., 'Incomplete', 'Completed')
* `priority`: Enum (e.g., 'Low', 'Medium', 'High')
* `due_date`: DateTime
* `user_id`: UUID (Foreign Key to Users)

**Endpoints**:

* `POST /tasks`: Create a new task.
* `GET /tasks`: List all tasks for the authenticated user.
* `PUT /tasks/{task_id}`: Update a task (e.g., mark as completed, change title/description).
* `DELETE /tasks/{task_id}`: Delete a task.

### **3. Task Categorization & Tagging**

* Tasks can be assigned to **categories** (e.g., "Work", "Personal", "Shopping").
* Tasks can have **tags** (e.g., "Urgent", "Reminder").

**Database Tables**:

* **Categories Table**: `category_id`, `category_name`
* **Tags Table**: `tag_id`, `tag_name`
* **TaskTags Table** (many-to-many relationship): `task_id`, `tag_id`

**Endpoints**:

* `POST /tasks/{task_id}/add-category`: Add a category to a task.
* `POST /tasks/{task_id}/add-tag`: Add a tag to a task.
* `GET /tasks/tags`: List all available tags.

### **4. Task Prioritization and Filtering**

Allow users to sort and filter tasks by priority, due date, and completion status.

**Endpoints**:

* `GET /tasks/filter`: Filter tasks based on criteria such as priority, status (completed/incomplete), or due date.

### **5. Task Reminders**

* Users can set reminders for tasks. When a task's due date approaches (e.g., 24 hours before), a notification will be triggered.

**Endpoints**:

* `POST /tasks/{task_id}/set-reminder`: Set a reminder for a task.
* `GET /tasks/reminders`: Get all tasks with upcoming reminders.

### **6. Task History/Activity Log**

* Track changes made to tasks (e.g., updates, completions).
* Maintain a history of actions with timestamps and user details.

**Database Schema**:

* **TaskHistory Table**: `task_id`, `action`, `previous_value`, `new_value`, `timestamp`, `user_id`.

**Endpoints**:

* `GET /tasks/{task_id}/history`: View the history of changes for a task.

### **7. Full-Text Search for Tasks**

* Implement full-text search functionality for searching tasks based on keywords in the title or description.

**Endpoints**:

* `GET /tasks/search?q={query}`: Search for tasks by keywords.

---

## **Frontend Functionality (Streamlit)**

### **1. User Registration & Login**

* Provide simple forms for user registration and login.
* After successful login, redirect the user to their task dashboard.

### **2. Task Dashboard**

* After logging in, users will be taken to a dashboard that displays all their tasks.
* Tasks should be listed with key information (title, priority, due date, and completion status).
* Users can filter tasks by completion status, due date, or priority.

### **3. Task Management**

* Allow users to create new tasks by filling out a form (title, description, due date, priority).
* Allow users to edit or delete their existing tasks.

### **4. Task Sorting**

* Provide sorting options for tasks (e.g., by priority or due date).
* Allow users to click on column headers (e.g., "Priority", "Due Date") to sort tasks.

### **5. Task History View**

* Display the history of task changes below each task.
* Show information such as previous values, new values, and timestamps.

### **6. Task Reminders**

* Display upcoming reminders for tasks due soon (within 24 hours) in a separate section or highlighted in the task list.

---

## **Additional Functionality (Optional Enhancements)**

### **1. Task Assignment for Teams**

* Allow tasks to be assigned to different users. Each user can see tasks assigned to them.

### **2. Batch Operations**

* Implement batch operations for updating or deleting multiple tasks at once (e.g., mark several tasks as complete).

### **3. Recurring Tasks**

* Allow users to create recurring tasks (e.g., a task that repeats every week or month).

---

## **Database Design**

### **Tables**:

1. **Users**: Store user data (username, email, password hash).
2. **Tasks**: Store task details.
3. **Categories**: Store task categories.
4. **Tags**: Store task tags.
5. **TaskTags**: Many-to-many relationship between tasks and tags.
6. **TaskHistory**: Track task updates.
7. **Reminders**: Store reminders for tasks.

---

## **Development Guidelines**

1. **Authentication**: Use JWT tokens for authentication and protect routes that require a user to be logged in.
2. **API Design**: Follow RESTful conventions for endpoint naming and HTTP methods.
3. **Error Handling**: Ensure proper error messages for invalid data and failed operations.
4. **Code Quality**: Write clean, maintainable, and well-documented code.
5. **Testing**: Write unit tests for critical logic (e.g., task creation, sorting, and filtering).

---

## **Evaluation Criteria**

* **Backend**:

  * Proper use of FastAPI and PostgreSQL.
  * Clean and efficient database queries.
  * Implementation of business logic for task prioritization, reminders, and history tracking.
  * Secure and effective user authentication with JWT tokens.

* **Frontend**:

  * Usability and functionality of the Streamlit app.
  * Interaction with the backend via API calls.
  * Clean and responsive UI for managing tasks.

* **Code Quality**:

  * Well-structured code, following Python best practices.
  * Clear comments and documentation where necessary.
