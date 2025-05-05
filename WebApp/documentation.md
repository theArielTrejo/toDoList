# Task Management Web Application Documentation

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Backend Components](#backend-components)
  - [Models](#models)
  - [Views](#views)
  - [Forms](#forms)
  - [URLs and Routing](#urls-and-routing)
  - [API Endpoints](#api-endpoints)
- [Frontend Components](#frontend-components)
  - [Templates](#templates)
  - [JavaScript Components](#javascript-components)
  - [CSS Components](#css-components)
  - [Interactive Elements](#interactive-elements)
- [Authentication System](#authentication-system)
- [Database Schema](#database-schema)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Integration with Desktop Application](#integration-with-desktop-application)

## Overview

The Task Management Web Application is a Django-based web application for managing tasks and assignments. It shares the same PostgreSQL database with a Tkinter desktop application. The web application provides a responsive interface built with Material Design 3 principles.

### Key Features

- User authentication (login, registration)
- Role-based access control (Admin/Assigner vs Regular User/Assignee)
- Task/Assignment management (Create, Read, Update, Delete)
- Status tracking for assignments
- Subtask management with automatic status updates
- Responsive web interface using Material Design 3
- Interactive cat mascots with motivational messages

## Architecture

The web application follows the standard Django MVT (Model-View-Template) architecture:

- **Models**: Define the database structure and business logic
- **Views**: Handle HTTP requests and return responses
- **Templates**: Define the HTML structure and presentation
- **Forms**: Handle form validation and processing
- **URLs**: Define the routing for the application
- **Static Files**: Contain CSS, JavaScript, and images

## Backend Components

### Models

#### Assignment Model

The core model for managing tasks and assignments.

```python
class Assignment(models.Model):
    STATUS_CHOICES = [
        ('Not Started', 'Not Started'),
        ('In Progress', 'In Progress'),
        ('Completed', 'Completed'),
    ]

    name = models.CharField(max_length=200)
    due_date = models.DateField()
    description = models.TextField()
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='Not Started')
    creator = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='created_assignments')
    assignees = models.ManyToManyField(User, related_name='assigned_tasks', blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

Key functionality:
- Tracks assignment name, due date, description, and status
- Links to creator (admin/staff user) and assignees (regular users)
- Includes timestamps for creation and updates
- Provides method to update status based on subtask completion

#### SubTask Model

Represents individual subtasks within an assignment.

```python
class SubTask(models.Model):
    name = models.CharField(max_length=200)
    is_completed = models.BooleanField(default=False)
    assignment = models.ForeignKey(Assignment, on_delete=models.CASCADE, related_name='subtasks')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

Key functionality:
- Tracks subtask name and completion status
- Links to parent assignment
- Automatically updates parent assignment status when saved

### Views

#### HomeView

Dashboard view displaying assignments grouped by due date.

```python
class HomeView(LoginRequiredMixin, ListView):
    model = Assignment
    template_name = 'tasks/home.html'
    context_object_name = 'assignments'
    ordering = ['-created_at']
    paginate_by = 10
```

Key functionality:
- Displays assignments grouped by today, this week, this month, and future
- Shows progress statistics and completion percentages
- Filters assignments based on user role (all for staff, assigned only for regular users)

#### AssignmentDetailView

Detailed view of a single assignment.

```python
class AssignmentDetailView(LoginRequiredMixin, DetailView):
    model = Assignment
    template_name = 'tasks/assignment_detail.html'
    context_object_name = 'assignment'
```

Key functionality:
- Shows all assignment details including description, due date, status
- Displays subtasks with completion status
- Provides access to edit/delete for staff users

#### AssignmentCreateView and AssignmentUpdateView

Views for creating and updating assignments.

Key functionality:
- Form handling for assignment creation/editing
- Advanced mode for adding subtasks
- Automatic status updates based on subtask completion
- Permission checks to ensure only staff can create/edit

#### API Endpoints

#### update_subtask

```python
@login_required
def update_subtask(request, pk):
    """API endpoint to update a subtask's completion status"""
```

**URL**: `/api/subtask/<pk>/update/`
**Method**: POST
**Authentication**: Required
**Parameters**:
- `pk` (URL parameter): The ID of the subtask to update

**Request Body**:
```json
{
  "is_completed": true|false
}
```

**Response**:
```json
{
  "success": true,
  "is_completed": true|false,
  "completion_percentage": 75,
  "assignment_status": "In Progress"
}
```

Key functionality:
- Updates subtask completion status via AJAX
- Returns updated assignment status and completion percentage
- Handles authorization to ensure only staff or assignees can update
- Automatically updates parent assignment status based on subtask completion

#### update_status

```python
@login_required
def update_status(request, pk):
    """Update the status of an assignment"""
```

**URL**: `/assignment/<pk>/status/`
**Method**: POST
**Authentication**: Required
**Parameters**:
- `pk` (URL parameter): The ID of the assignment to update

**Form Data**:
- `status`: New status value ("Not Started", "In Progress", or "Completed")

Key functionality:
- Updates assignment status directly
- Handles form validation
- Redirects to assignment detail page after update
- Provides success/error messages

### Forms

#### AssignmentForm

Form for creating and editing assignments.

```python
class AssignmentForm(forms.ModelForm):
    advanced_mode = forms.BooleanField(
        required=False,
        initial=False,
        label='Enable Advanced Mode',
        help_text='Add subtasks and track progress'
    )
```

Key functionality:
- Handles basic assignment fields (name, due date, description, status, assignees)
- Provides advanced mode for subtask management
- Integrates with subtask formsets
- Automatically sets status for new assignments to 'Not Started'

#### UserRegistrationForm

Form for user registration with simplified password requirements.

### URLs and Routing

Main URL patterns:

- `/` - Home dashboard (HomeView)
- `/assignment/<pk>/` - Assignment detail view
- `/assignment/new/` - Create new assignment
- `/assignment/<pk>/update/` - Update existing assignment
- `/assignment/<pk>/delete/` - Delete assignment
- `/assignment/<pk>/status/` - Update assignment status
- `/register/` - User registration
- `/login/` - User login (Django auth views)
- `/logout/` - User logout (Django auth views)
- `/api/subtask/<pk>/update/` - API endpoint for updating subtasks

## Frontend Components

### Templates

#### Base Template

`base.html` - The main template that all other templates extend.

Key components:
- Material Design 3 header with navigation
- Message handling system
- Common scripts and styles
- Responsive layout structure

#### Page Templates

- `home.html` - Dashboard with assignment lists and statistics
- `assignment_detail.html` - Detailed view of a single assignment
- `assignment_form.html` - Form for creating/editing assignments
- `login.html` - User login page
- `register_form.html` - User registration page

#### Partial Templates

- `cat_mascot.html` - Reusable template for cat mascots
- Other partial templates for components and sections

### JavaScript Components

#### Main JavaScript

`main.js` - Entry point for all JavaScript functionality.

```javascript
// Import component scripts
import {initializeSnackbar, processDjangoMessages} from './components/snackbar.js';
import './components/forms.js';
import './components/progress.js';

// Import form handlers
import './forms/assignment-form.js';
import './forms/auth-forms.js';

// Initialize Material Design components
document.addEventListener('DOMContentLoaded', function() {
  // Auto-initialize all MDC components
  window.mdc.autoInit();

  // Initialize custom components
  initializeCustomComponents();

  // Initialize snackbar and process Django messages
  initializeSnackbar();
  processDjangoMessages();
});
```

Key functionality:
- Initializes Material Design components
- Imports and initializes custom components
- Sets up event listeners and handlers
- Processes Django messages for display in snackbars

#### Component Scripts

##### Snackbar Component

`snackbar.js` - Material Design 3 snackbar for notifications.

```javascript
function showMessage(message, type = 'success', duration = 5000) {
  // Implementation details...
}

function showSuccess(message, duration = 5000) {
  showMessage(message, 'success', duration);
}

function showWarning(message, duration = 7000) {
  showMessage(message, 'warning', duration);
}

function showError(message, duration = 10000) {
  showMessage(message, 'error', duration);
}
```

Key functionality:
- Displays toast notifications for success, warning, and error messages
- Processes Django messages from the server
- Configurable duration and styling based on message type

##### Form Handling

`forms.js` - Form validation and handling.

Key functionality:
- Initializes Material Design form components
- Handles form validation
- Manages form state and submission

##### Progress Indicators

`progress.js` - Progress indicators for task completion.

Key functionality:
- Circular progress indicators for overall task completion
- Linear progress bars for individual assignments
- Dynamic updates via JavaScript

##### Cat Mascots

`cat_mascots.js` - Interactive cat mascots with messages.

```javascript
// Motivational messages for the Lockedin Cat
const lockedinCatMessages = [
    "You're doing purrfect work today!",
    "Stay pawsitive, you've got this!",
    "Feline good about your progress!",
    // More messages...
];

// Hurry-up messages for the Bosscat
const bosscatMessages = [
    "Hurry up, I don't have nine lives to wait!",
    "Stop procrastinating, it's meow or never!",
    // More messages...
];
```

Key functionality:
- Displays cat mascots with SVG styling
- Shows random motivational messages on click
- Animates message bubbles
- Different message sets for different cat types (Lockedin Cat, Bosscat)

### CSS Components

The CSS is organized using a modular approach with separate files for different components and functionality. The main CSS file imports all component styles:

```css
/**
 * Task Manager - Main CSS File
 * This file imports all component, layout, page, and theme styles
 */

/* Base styles and variables */
@import 'themes/variables.css';
@import 'themes/base.css';

/* Component styles */
@import 'components/buttons.css';
@import 'components/cards.css';
@import 'components/forms.css';
@import 'components/form-inputs.css';
@import 'components/form-validation.css';
@import 'components/cat-mascots.css';
@import 'components/progress.css';
@import 'components/snackbar.css';
@import 'components/lists.css';

/* Layout styles */
@import 'layouts/header.css';
@import 'layouts/main-content.css';
@import 'layouts/responsive.css';

/* Page-specific styles */
@import 'pages/home.css';
@import 'pages/login.css';
@import 'pages/assignment-form.css';
@import 'pages/assignment-detail.css';

/* Utility classes */
@import 'utils.css';
```

#### Component Styles

##### Cat Mascots

`components/cat-mascots.css` - Styling for the interactive cat mascots.

```css
.cat-mascot {
    position: relative;
    width: 80px;
    height: 80px;
    margin: var(--spacing-md) auto;
    cursor: pointer;
    transition: transform var(--transition-medium);
}

.cat-message-bubble {
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    background-color: var(--mdc-theme-surface);
    color: var(--mdc-theme-on-surface);
    padding: var(--spacing-sm) var(--spacing-md);
    border-radius: var(--border-radius-md);
    box-shadow: var(--shadow-2);
    width: 250px;
    max-width: calc(100vw - 40px);
    margin-bottom: 10px;
    text-align: center;
    z-index: 10;
    opacity: 0;
    transition: opacity var(--transition-medium), transform var(--transition-medium);
}
```

##### Forms and Inputs

Styling for Material Design 3 form components including:
- Text fields with floating labels
- Checkboxes and radio buttons
- Select dropdowns
- Form validation states
- Form layout and spacing

##### Progress Indicators

Styling for Material Design 3 progress indicators:
- Circular progress for overall completion
- Linear progress bars for individual items
- Color coding based on status

#### Layout Styles

- **Header**: Top app bar with navigation and actions
- **Main Content**: Content layout and container styles
- **Responsive**: Media queries for different screen sizes

#### Page-Specific Styles

- **Home**: Dashboard layout and card styles
- **Login/Register**: Authentication form styling
- **Assignment Form**: Create/edit form layout
- **Assignment Detail**: Detail view styling

### Interactive Elements

#### Cat Mascots

Interactive cat mascots that display motivational messages when clicked.

```javascript
// Motivational messages for the Lockedin Cat
const lockedinCatMessages = [
    "You're doing purrfect work today!",
    "Stay pawsitive, you've got this!",
    // More messages...
];

// Hurry-up messages for the Bosscat
const bosscatMessages = [
    "Hurry up, I don't have nine lives to wait!",
    "Stop procrastinating, it's meow or never!",
    // More messages...
];
```

#### Snackbar Notifications

Material Design 3 snackbar for displaying success, warning, and error messages.

## Authentication System

The application uses Django's built-in authentication system with custom forms and views.

### Login System

The login functionality uses Django's built-in `LoginView` with a custom template:

```python
path('login/', auth_views.LoginView.as_view(template_name='tasks/login.html'), name='login'),
```

Key features:
- Material Design 3 styled login form
- Error handling and validation
- Session-based authentication
- Redirect to dashboard after successful login
- Interactive Bosscat mascot with motivational messages

### Registration System

The registration system uses a custom view and form:

```python
def register_view(request):
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)
        if form.is_valid():
            form.save()
            messages.success(request, 'Account created successfully! You can now log in.')
            return redirect('login')
    else:
        form = UserRegistrationForm()
    return render(request, 'tasks/register_form.html', {'form': form})
```

Key features:
- Simplified password requirements for easier registration
- Custom form validation
- Success messages displayed on the login page after registration
- Material Design 3 styled registration form

### User Roles and Permissions

The application implements role-based access control:

- **Staff Users (Admins/Assigners)**:
  - Can create, edit, and delete assignments
  - Can view all assignments in the system
  - Can assign tasks to regular users

- **Regular Users (Assignees)**:
  - Can view only assignments assigned to them
  - Can update the status of their assignments
  - Can mark subtasks as completed

Permission checks are implemented using Django's `UserPassesTestMixin` and custom test functions:

```python
def is_admin(user):
    return user.is_staff

class AssignmentCreateView(LoginRequiredMixin, UserPassesTestMixin, CreateView):
    # View implementation...

    def test_func(self):
        return self.request.user.is_staff
```

### Logout System

The logout functionality uses Django's built-in `LogoutView`:

```python
path('logout/', auth_views.LogoutView.as_view(next_page='login'), name='logout'),
```

Key features:
- Secure session termination
- Redirect to login page after logout

## Database Schema

The application uses a PostgreSQL database with the following main tables:

- **auth_user**: Django's built-in User model
- **tasks_assignment**: Stores assignment data
- **tasks_subtask**: Stores subtask data
- **tasks_assignment_assignees**: Many-to-many relationship between assignments and users

## Configuration

The application is configured using environment variables loaded from a `.env` file:

- `SECRET_KEY`: Django secret key
- `DEBUG`: Debug mode flag
- `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`: Database connection parameters


## Integration with Desktop Application

The web application shares the same PostgreSQL database with a Tkinter-based desktop application, allowing for seamless data synchronization between both interfaces.

### Shared Database Access

Both applications access the same database tables:
- Users can create accounts and assignments in either application
- Changes made in one application are immediately visible in the other
- The same authentication credentials work in both applications

### Database Synchronization

When setting up the application for the first time, you must run the sync command to configure the database tables correctly:

```bash
cd WebApp
python manage.py sync_db
```

This command ensures that the database schema is properly configured for both applications.

### Use Cases for Web vs Desktop

- **Web Application**: Ideal for users who need to access their tasks from multiple devices or locations
- **Desktop Application**: Provides a native experience for users who primarily work from a single computer

### Technical Considerations

- Both applications use the same models and database schema
- The web application uses Django's ORM for database access
- The desktop application uses a custom database access layer
- Care must be taken when making schema changes to ensure compatibility with both applications
