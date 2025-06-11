# Building a Flask To-Do App with Template Inheritance

This guide explains Flask, a Python microframework, and demonstrates how to build a To-Do app with template inheritance using Jinja. It covers setup, directory structure, routing, templates, best practices, and common beginner mistakes to avoid.

## What is Flask and Why Use It?

Flask is a lightweight, micro web framework for Python, designed to be minimalistic yet flexible. Unlike full-stack frameworks like Django, Flask provides only the essentials—routing, request handling, and templating—allowing developers to choose their database, ORM, or other tools. This makes Flask ideal for:

- **Small to Medium Projects**: Perfect for a To-Do app or prototypes due to its simplicity.
- **Flexibility**: Choose your database (e.g., SQLite, PostgreSQL) or templating engine (Jinja by default).
- **Learning Curve**: Easy for beginners to grasp while powerful for advanced users.

**Why Use Flask?**
- Rapid development with minimal boilerplate.
- Extensible with libraries like Flask-SQLAlchemy or Flask-WTF.
- Ideal for APIs, web apps, or microservices.

**Question to Explore**: How does Flask’s lack of built-in features (e.g., no default ORM) affect your project’s setup compared to Django?

## Directory Structure

A well-organized directory structure keeps your Flask app maintainable. Here’s a recommended structure for a To-Do app:

```
todo_app/
│
├── static/                   # CSS, JavaScript, images, etc.
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── script.js
├── templates/                # HTML templates
│   ├── base.html
│   ├── index.html
│   └── about.html
├── app.py                    # Main Flask application
├── requirements.txt          # Dependencies
└── instance/
    └── todo.db               # SQLite database
```

- **static/**: Stores files served as-is (e.g., CSS, images, videos). Flask automatically maps `/static/` to this folder.
- **templates/**: Stores Jinja templates (e.g., `index.html`, `base.html`). Flask looks here for HTML files.
- **instance/**: Stores instance-specific data like SQLite databases (kept outside version control).
- **app.py**: The main Flask application file.
- **requirements.txt**: Lists dependencies (e.g., `Flask==2.3.2`).

**Beginner Mistake**: Placing static files in `templates/` or templates in `static/` causes 404 errors. Always use `url_for('static', filename='css/style.css')` in templates to reference static files.

**Question**: Why might you want to separate static and template files? What issues arise if you misplace them?

## Setting Up Flask

Let’s create a Flask app from scratch for the To-Do app, including an "About Us" page.

### Step 1: Install Flask
Create a virtual environment and install Flask:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install flask flask-sqlalchemy
pip freeze > requirements.txt
```

### Step 2: Create `app.py`
Below is a Flask app with routes for the To-Do list and an "About Us" page, using SQLite for data storage.

```python
from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///instance/todo.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# Database Model
class Todo(db.Model):
    sno = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    desc = db.Column(db.String(500), nullable=False)
    date_created = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f"<Todo {self.title}>"

# Create database
with app.app_context():
    db.create_all()

# Routes
@app.route("/", methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        title = request.form['title']
        desc = request.form['desc']
        if title:  # Validate input
            todo = Todo(title=title, desc=desc)
            db.session.add(todo)
            db.session.commit()
    all_todos = Todo.query.all()
    return render_template('index.html', allTodo=all_todos)

@app.route("/about")
def about():
    return render_template('about.html')

@app.route("/delete/<int:sno>")
def delete(sno):
    todo = Todo.query.filter_by(sno=sno).first()
    if todo:
        db.session.delete(todo)
        db.session.commit()
    return redirect(url_for('index'))

@app.route("/update/<int:sno>", methods=['GET', 'POST'])
def update(sno):
    todo = Todo.query.filter_by(sno=sno).first()
    if not todo:
        return redirect(url_for('index'))
    if request.method == 'POST':
        todo.title = request.form['title']
        todo.desc = request.form['desc']
        db.session.commit()
        return redirect(url_for('index'))
    return render_template('update.html', todo=todo)

if __name__ == "__main__":
    app.run(debug=True)
```

**Key Features**:
- Uses Flask-SQLAlchemy for database management.
- Supports CRUD operations (Create, Read, Update, Delete).
- Includes routes for home (`/`), about (`/about`), delete (`/delete/<sno>`), and update (`/update/<sno>`).
- Runs in debug mode for development (`debug=True`).

**Beginner Mistake**: Forgetting to create the database with `db.create_all()` or misconfiguring `SQLALCHEMY_DATABASE_URI` can lead to database errors. Always test database connectivity early.

**Tip**: Use `url_for()` in routes (e.g., `redirect(url_for('index'))`) to avoid hardcoding URLs, which breaks if routes change.

## Template Inheritance with Jinja

Template inheritance allows you to define a `base.html` with common elements (e.g., navbar, footer) and extend it in other templates, reducing duplication. This is perfect for creating multiple pages, like your 10-page "About Us" requirement.

### Step 3: Create `base.html`
Place this in `templates/base.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My To-Do App{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.6/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <nav class="navbar navbar-expand-lg bg-light">
        <div class="container">
            <a class="navbar-brand fw-bold" href="{{ url_for('index') }}">My To-Do</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="{{ url_for('index') }}">Home</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{{ url_for('about') }}">About Us</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>

    <div class="container my-4">
        {% block body %}
        {% endblock %}
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.6/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

**Key Elements**:
- **Viewport Meta**: Ensures responsiveness.
- **Bootstrap**: Provides a responsive, modern UI.
- **Blocks**: `{% block title %}` and `{% block body %}` allow child templates to override the title and content.
- **Static Files**: Links to `style.css` using `url_for`.

**Beginner Mistake**: Forgetting to close `{% block %}` tags or nesting them incorrectly can cause rendering issues. Always test templates individually.

### Step 4: Create `index.html`
Extend `base.html` for the To-Do list:

```html
{% extends 'base.html' %}
{% block title %}To-Do List{% endblock %}
{% block body %}
    <div class="card p-4 mb-4">
        <h2 class="mb-4">Add To-Do</h2>
        <form action="{{ url_for('index') }}" method="post">
            <div class="mb-3">
                <label for="title" class="form-label">To-Do Title</label>
                <input type="text" class="form-control" name="title" id="title" required>
            </div>
            <div class="mb-3">
                <label for="desc" class="form-label">To-Do Description</label>
                <textarea name="desc" class="form-control" id="desc" rows="4"></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Add Task</button>
        </form>
    </div>

    <div class="card p-4">
        <h2 class="mb-4">Your To-Dos</h2>
        {% if allTodo|length == 0 %}
        <div class="alert alert-info" role="alert">
            No tasks found. Add your first to-do above!
        </div>
        {% else %}
        <table class="table table-hover">
            <thead>
                <tr>
                    <th scope="col">#</th>
                    <th scope="col">Title</th>
                    <th scope="col">Description</th>
                    <th scope="col">Created</th>
                    <th scope="col">Actions</th>
                </tr>
            </thead>
            <tbody>
                {% for todo in allTodo %}
                <tr>
                    <th scope="row">{{ loop.index }}</th>
                    <td>{{ todo.title }}</td>
                    <td>{{ todo.desc }}</td>
                    <td>{{ todo.date_created.strftime('%Y-%m-%d %H:%M') }}</td>
                    <td>
                        <a href="{{ url_for('update', sno=todo.sno) }}" class="btn btn-outline-dark btn-sm mx-1">Edit</a>
                        <a href="{{ url_for('delete', sno=todo.sno) }}" class="btn btn-outline-success btn-sm mx-1">Complete</a>
                    </td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
        {% endif %}
    </div>
{% endblock %}
```

### Step 5: Create `about.html`
For the "About Us" page (extendable to 10 pages by creating similar templates):

```html
{% extends 'base.html' %}
{% block title %}About Us{% endblock %}
{% block body %}
    <div class="card p-4">
        <h2>About Us</h2>
        <p>Welcome to My To-Do App! This application helps you manage tasks efficiently.</p>
        <p>Our mission is to simplify task management with a clean, user-friendly interface.</p>
    </div>
{% endblock %}
```

**Tip**: To create 10 "About Us" pages, create templates like `about_team.html`, `about_mission.html`, etc., all extending `base.html`. Use a route like `@app.route("/about/<section>")` to dynamically render them.

### Step 6: Add Static Files
Create `static/css/style.css` for custom styling:

```css
body {
    background-color: #f8f9fa;
}
.card {
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
.btn-outline-dark:hover {
    background-color: #343a40;
    color: #fff;
}
.btn-outline-success:hover {
    background-color: #28a745;
    color: #fff;
}
```

## Tips and Tricks for Flask Development

1. **Use Blueprints for Scalability**:
   - Organize routes into blueprints (e.g., `todo_blueprint`, `about_blueprint`) for modular code.
   - Example: `from flask import Blueprint; todo_bp = Blueprint('todo', __name__)`.

2. **Environment Configuration**:
   - Use `python-dotenv` to load environment variables from a `.env` file (e.g., `FLASK_ENV=development`).
   - Never hardcode sensitive data like database URIs.

3. **Debug Mode**:
   - Enable `debug=True` during development for live reloading and error traces.
   - **Mistake**: Running debug mode in production exposes security risks.

4. **Input Validation**:
   - Validate form inputs (e.g., check if `title` is empty) to prevent bad data.
   - Use Flask-WTF for form validation in larger apps.

5. **Static File Management**:
   - Always use `url_for('static', filename='...')` to reference static files.
   - **Mistake**: Hardcoding paths like `/static/css/style.css` breaks if the app is deployed under a subpath.

6. **Jinja Best Practices**:
   - Use `{% block %}` for all reusable sections (e.g., `title`, `body`, `scripts`).
   - Avoid duplicating code in templates; centralize in `base.html`.

7. **Routing Tips**:
   - Use meaningful route names (e.g., `/tasks` instead of `/products` for a To-Do app).
   - Support multiple HTTP methods (e.g., `GET`, `POST`) where needed.
   - **Mistake**: Forgetting to specify `methods` in `@app.route` can cause 405 errors.

## Common Beginner Mistakes to Avoid

1. **Not Using `url_for`**:
   - Hardcoding URLs (e.g., `href="/about"`) breaks if routes change. Always use `url_for('about')`.

2. **Improper Template Inheritance**:
   - Forgetting `{% extends 'base.html' %}` or mismatching block names causes content to disappear.

3. **Database Issues**:
   - Not initializing the database (`db.create_all()`) leads to table not found errors.
   - Forgetting `db.session.commit()` discards changes.

4. **Static File Errors**:
   - Misplacing files outside `static/` or using incorrect `url_for` syntax results in 404 errors.

5. **Running Debug in Production**:
   - `debug=True` exposes stack traces to users. Use environment variables to disable it in production.

6. **Ignoring Responsiveness**:
   - Not testing on mobile devices can lead to poor UX. Bootstrap’s viewport meta tag and classes help, but test thoroughly.

## Running the App

1. Activate the virtual environment:
   ```bash
   source venv/bin/activate
   ```

2. Run the app:
   ```bash
   python app.py
   ```

3. Access at `http://localhost:5000`. Test the To-Do list and "About Us" page.

## Extending to 10 Pages

To create 10 "About Us" pages:
- Create templates like `about_mission.html`, `about_team.html`, etc., each extending `base.html`.
- Add a dynamic route:
  ```python
  @app.route("/about/<section>")
  def about_section(section):
      return render_template(f'about_{section}.html')
  ```
- Ensure each template exists to avoid 404 errors.

**Question**: How would you structure the content for 10 "About Us" pages? Would a single dynamic route or separate routes be more maintainable?

## Conclusion

Flask’s minimalistic design, combined with Jinja’s template inheritance, makes it ideal for building a To-Do app with reusable components. By organizing routes, templates, and static files properly, and avoiding common pitfalls, you can create a scalable, user-friendly app. Test your app thoroughly, validate inputs, and use tools like Flask-SQLAlchemy to streamline development.

**Next Steps**:
- Add user authentication with Flask-Login.
- Implement search functionality for tasks.
- Deploy to a platform like Heroku or Render.