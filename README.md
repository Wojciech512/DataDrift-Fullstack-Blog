# DataDrift Fullstack Blog

## Project Overview

**DataDrift Fullstack Blog** is a full-stack web application (blog platform) that allows users to create, publish, and share articles with others – much like a Medium.com clone. The goal of this project is to provide a **knowledge-sharing platform** with a modern web interface and robust backend, serving as a practical example of a complete web application. Key features include user authentication (with JSON Web Tokens), rich text article creation (using Markdown), tagging, commenting, and social interactions such as favoriting articles and following other authors. In other words, DataDrift provides *“a place to share your knowledge.”*

**Main features:**

* **User Accounts & Profiles:** Users can register with a unique email/username and log in. Each profile includes a bio, profile picture, and the ability to follow/unfollow other user profiles.
* **JWT Authentication:** The backend issues JWT tokens upon login/registration. These tokens are used to authorize subsequent requests (included via an `Authorization: Token <jwt>` header).
* **Publishing Articles:** Logged-in users can create new blog posts (articles) with a title, description (teaser), and body content. Articles support **Markdown** formatting (converted to HTML on display) for rich text content.
* **Tags:** Users can add tags to their articles. The homepage includes a sidebar of popular tags to filter articles. The system tracks tag usage to display top tags.
* **Article Feed:** The home page shows a global feed of all recent articles and a personalized feed of articles from authors a user follows. Users can toggle between these feeds.
* **Favorite Articles:** Readers can favorite (like) an article they enjoy, and these favorites are saved to their profile. The article model tracks which users have favorited it. Favoriting an article updates its favorite count and is reflected in the UI (♥ icon).
* **Comments:** Users can comment on articles. Comments appear below each article in a discussion thread, showing the comment content and author. Article pages allow adding new comments (for logged-in users) or deleting one’s own comments.
* **Following Authors:** Users can follow other authors to see a personalized feed of articles. On an author’s profile, you can follow/unfollow them. The backend manages a many-to-many relationship for followers/following between User profiles, and the feed API returns articles by followed authors when the “Your Feed” is selected.

This project is not only a functional blog engine but also a demonstration of a modern **full-stack architecture** using separate frontend and backend layers. It adheres to a common API spec (similar to the RealWorld example app), making it a great reference for building full-stack applications with Angular and Django.

## Screenshots

*(Screenshots of the application’s user interface will be added here.)* Below are example screenshots illustrating the UI:

* **Home Page:** Shows the site banner and toggles between the global feed and your personalized feed, along with a list of popular tags. *(e.g. a banner with the text "DataDrift" and tagline "A place to share your knowledge.", a list of recent articles with pagination, and a sidebar of tags.)*
* **Article Page:** Displays a full article (title, author, body content formatted from Markdown), with favorite and follow buttons, and a comments section beneath.
* **Editor Page:** A form for writing a new article or editing an existing one (fields for title, description, tags, and a textarea for the content).
* **Profile Page:** Shows a user’s profile information (bio, avatar) and their articles or favorited articles, with a button to follow/unfollow (for other users’ profiles).
* **Authentication Pages:** Forms for signing up and signing in, each with input validation feedback.

*(*The repository does not include pre-generated screenshots. To see the interface, run the application locally as described below, and then you can capture screenshots of the UI.*)*

## Architecture

The application is organized into a **frontend client** and a **backend server**, plus a database. The components communicate primarily via HTTP using a RESTful API provided by the backend. The high-level architecture is as follows:

* **Frontend:** An Angular application (HTML/TypeScript) that provides the user interface. It is a single-page application (SPA) that runs in the browser. The UI is built with Angular **v18** and uses Bootstrap 5 for styling. The frontend is responsible for rendering pages (home, login, editor, etc.), handling user interactions, and making AJAX calls to the backend API. It uses the Angular Router for client-side routing and services to manage authentication tokens and data fetching. The app is packaged for production as static files (HTML/CSS/JavaScript) which can be served by any web server. In development, it runs on a local dev server (usually at `localhost:4200`). The Angular app knows how to reach the backend via a base API URL; in development it’s configured to use `http://localhost:8000/api`, and in production it’s set to `'/api'` (relative path) so that it works when served from the same domain as the backend (through a reverse proxy).

* **Backend:** A RESTful API server built with **Python Django** (using Django REST Framework). It implements all the core domain logic: user management, authentication, and article operations. The backend follows a modular design with Django apps such as `users` and `articles`. Key aspects of the backend:

  * *Django 5:* Built with Django **5.0** (as indicated by the migration files), ensuring the latest features and security updates.
  * *Django REST Framework:* Used to create API endpoints. The API returns JSON responses and handles authentication via JWT tokens (using the `djangorestframework-simplejwt` package). The default authentication class is JWT authentication.
  * *Models:* The data model is defined using Django’s ORM. For example, the `Article` model includes fields for title, description, body, tags (many-to-many), author (foreign key to User), and a many-to-many relation to track users who have favorited the article. The `User` model extends Django’s AbstractUser to use email as login and adds fields like bio, image, and a self-referential many-to-many for following other users. There are also `Comment` and `Tag` models for article comments and tagging functionality.
  * *Views & Serializers:* The backend exposes endpoints such as `POST /api/users` (registration), `POST /api/users/login` (authentication), `GET/PUT /api/user` (current user profile), `GET /api/profiles/:username` (public profile), `POST/DELETE /api/profiles/:username/follow` (follow/unfollow), `CRUD /api/articles` and nested `comments`, `GET /api/tags`, etc. These are implemented with Django REST Framework viewsets and class-based views. Serializers handle converting model instances to JSON and validating input.
  * *Business Logic:* The backend ensures security (e.g., only authors can edit or delete their own articles), and implements actions like adding/removing favorites or following/unfollowing profiles as custom endpoints.
  * *Database:* Uses **PostgreSQL** as the primary data store. Django’s ORM is configured to use a Postgres database (see the `ENGINE` setting pointing to `django.db.backends.postgresql`). Data such as user accounts, articles, comments, and tags are all stored in the PostgreSQL database.
  * *Static File Serving:* In production, the Django app can serve the compiled Angular files. The settings include the Angular build output as a static files directory, and any URL not starting with `/api/` is routed to serve the Angular app’s `index.html` (enabling client-side routing). In the development setup, static files are served by Angular’s dev server instead.

* **Communication:** The frontend and backend communicate via REST API calls over HTTP. For example, when the Angular app loads the home page, it will call GET `api/articles` to fetch the list of articles. All API endpoints are prefixed with `/api/` for clarity. The Angular HttpClient calls include the JWT token for authenticated requests, which the backend verifies for protected endpoints. Thanks to the proxy configuration, in production the frontend calls `/api/...` on the same host (proxied to Django), and in development it calls the Django server directly at `localhost:8000`. Cross-Origin Resource Sharing (CORS) is enabled via the `django-cors-headers` module for development (allowing the Angular dev server to call the API).

* **Infrastructure:** The project is set up with **Docker** and **Docker Compose** to simplify running all components:

  * A PostgreSQL database container (`postgres:16-alpine`) is used for data storage.
  * The Django backend is containerized (Python 3.10 slim base) and served via Gunicorn WSGI server. On startup it runs database migrations and creates a default superuser (with credentials from environment) automatically.
  * The Angular frontend is built and served using an Nginx container. The Docker build process compiles the Angular app, then an Nginx image serves the static files. Nginx is configured to forward API requests to the backend container on the internal network. The frontend container listens on port 80 internally, mapped to port 4200 on the host for convenience.
  * All services are connected via a Docker network so that the frontend can reach the backend by service name. Healthchecks are in place to ensure proper startup order (backend waits for database, frontend waits for backend).

This multi-tier architecture (Angular client, Django API, Postgres DB) and the use of Docker make the app easy to deploy and scale. Each component can be developed and tested in isolation, and the integration is handled via clear API contracts and docker-compose orchestration.

## Technologies Used

**Frontend:**

* **Angular** (TypeScript) – Angular CLI generated project, using Angular v18.x. Implements a responsive SPA with routing and services for state management.
* **Bootstrap 5** – For responsive CSS styling and layout (the UI uses a custom Bootstrap theme, giving a clean, Medium-like look). The HTML templates use Bootstrap classes (containers, rows, buttons, etc.) for layout.
* **RxJS** – Used with Angular for reactive programming (handling asynchronous calls and streams).
* **ngx-markdown** and **marked** – Libraries to parse and display Markdown in articles. Users write article content in Markdown; the frontend converts it to HTML for display.
* **TypeScript** – Strongly-typed language for the Angular codebase, enabling robust client-side development.

**Backend:**

* **Python 3 & Django 5** – The web framework powering the server, providing the ORM, migrations, and admin interface. The project uses Django 5.0+.
* **Django REST Framework** – Extensions on Django to easily create RESTful APIs (viewsets, serializers, etc.). Enables robust request handling and input validation for the API.
* **PostgreSQL** – Relational database for persistent storage of all data. The app is configured to use Postgres (with psycopg2 as the driver). This choice ensures reliable data integrity and the ability to use advanced SQL features if needed.
* **JWT (JSON Web Tokens)** – Authentication mechanism. Implemented via the `djangorestframework-simplejwt` library. The backend issues JWT tokens for user login which the client stores and sends with each request. Django REST Framework’s JWT authentication class verifies tokens on protected endpoints.
* **Corsheaders** – Django middleware to allow cross-origin requests in development.
* **django-filter** – Used for filtering querysets via URL parameters (e.g., filtering articles by tag or author).
* **Gunicorn** – WSGI HTTP server to run the Django application in production (as seen in the Docker setup).
* **Others:** Standard Django apps like the built-in admin, sessions, etc., and testing frameworks (Django’s test framework for backend, Karma/Jasmine for frontend testing).

**Infrastructure/DevOps:**

* **Docker & Docker Compose** – Used to containerize the application and manage multi-container deployment (DB, backend, frontend). This simplifies setup and ensures consistency across environments.
* **Nginx** – Used in the frontend container to serve the Angular static files and proxy API calls to the backend.
* **Node.js & npm** – Used for building the Angular application (Node 20.x is used in the build stage for the frontend Dockerfile). Developers will use Node and npm locally to run and test the frontend.
* **TypeORM** – *\[Not used]* Despite the mention of TypeORM in the prompt (which typically applies to a Node.js backend), this project’s backend is Django, which uses its own ORM. Therefore, TypeORM is not applicable here. Instead, Django’s ORM is used for database interactions.

Both the frontend and backend are accompanied by test suites. The Angular project includes unit tests (using Jasmine/Karma) for components and services. The backend includes a comprehensive set of API tests using Django’s `APITestCase` to verify endpoints (e.g., user registration, login, article CRUD, commenting, etc.). These tests help ensure the application’s reliability and help new contributors understand expected behaviors.

## Installation and Running Locally

You can run DataDrift Fullstack Blog locally either with Docker (quick setup) or by running the frontend and backend separately on your machine. Below are instructions for both methods.

### 1. Running with Docker (Quick Start)

The easiest way to get started is using **Docker Compose**, which will set up all three services (frontend, backend, and database) for you. Make sure you have Docker and Docker Compose installed, then follow these steps:

```bash
# Clone the repository
git clone https://github.com/Wojciech512/DataDrift-Fullstack-Blog.git
cd DataDrift-Fullstack-Blog

# (Optional) Edit the .env file to set any custom values (e.g., DB password, or Django admin credentials).
# By default, .env has development settings with a default superuser (username: superUser, password: Test123!).

# Start all services using Docker Compose
docker-compose up --build
```

Docker Compose will pull the necessary images (for Node, Nginx, Python, Postgres) and build the application. On first run, the backend will apply database migrations and create a superuser automatically (using credentials from `.env`).

Once the process completes, you should have:

* Angular frontend running at **[http://localhost:4200](http://localhost:4200)**.
* Django backend API running at **[http://localhost:8000/api](http://localhost:8000/api)** (accessible via the front-end or directly via tools like curl/Postman).
* The PostgreSQL database running locally on port 5432 (if you wish to connect to it directly, use the credentials in `.env`).

You can log into the application using the default credentials (or create a new account via the Sign Up page on the frontend). The default admin/superuser for the Django admin site (at `http://localhost:8000/admin/`) is `superUser` / `Test123!` by default.

To stop the containers, press `Ctrl+C` in the terminal where Docker is running, or run `docker-compose down` to gracefully shut them down.

### 2. Manual Setup (Running frontend & backend without Docker)

If you prefer to run the frontend and backend manually (for development or debugging purposes), follow these steps:

**Prerequisites:** Make sure you have **Node.js** (recommend v18 or later) with npm, **Python 3.10+**, and **PostgreSQL** installed on your system. You’ll also need **Angular CLI** globally (`npm install -g @angular/cli`) or you can use the Angular CLI that’s included in the project’s devDependencies via npm scripts.

**Backend Setup:**

1. **Clone the repo and set up Python environment:**

   ```bash
   git clone https://github.com/Wojciech512/DataDrift-Fullstack-Blog.git
   cd DataDrift-Fullstack-Blog/backend
   python -m venv venv          # Create a virtual environment (optional but recommended)
   source venv/bin/activate     # Activate the venv (Linux/macOS)
   # On Windows: venv\Scripts\activate
   ```

2. **Install backend dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

   This installs Django, Django REST Framework, and other necessary packages.

3. **Configure environment variables:** The backend reads settings from environment or `.env` file. You can copy the provided `.env` (in the project root) or set your own environment variables for the database connection and Django settings. Key variables:

   * `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT` – point these to your PostgreSQL database and credentials. (By default, `.env` expects a Postgres instance with user/password `postgres` on localhost.)
   * `SECRET_KEY` – for Django (you can use the default insecure key for dev or generate a new one).
   * `DEBUG` – set to `True` for local development to enable debug mode.
   * You can also set `DJANGO_SUPERUSER_USERNAME`, `DJANGO_SUPERUSER_PASSWORD`, `DJANGO_SUPERUSER_EMAIL` if you want the automatic superuser creation to use your values (when running migrations).

4. **Set up the database:** Ensure PostgreSQL is running and a database exists (matching `DB_NAME`). Create the database if needed. If using default `.env`, you might create a database named “postgres” (or change `DB_NAME` to an existing DB).

5. **Run database migrations:**

   ```bash
   python manage.py migrate
   ```

   This will create the necessary tables in the database.

6. **Create a superuser (admin account):**

   ```bash
   python manage.py createsuperuser
   ```

   You’ll be prompted for email/username and password. (This step can be skipped if you set `DJANGO_SUPERUSER_*` env vars and are okay with using `createsuperuser --noinput` as done in Docker, but interactive creation is fine for dev.)

7. **Run the backend server:**

   ```bash
   python manage.py runserver 8000
   ```

   This starts the Django development server on port 8000. You should see console output that it’s running at [http://127.0.0.1:8000/](http://127.0.0.1:8000/). You can test an API endpoint, e.g., GET [http://127.0.0.1:8000/api/tags](http://127.0.0.1:8000/api/tags), which should return an empty list (or tags if any exist).

**Frontend Setup:**

1. **Install dependencies:**

   ```bash
   cd ../frontend   # from the project root, move into the frontend directory
   npm install
   ```

   This will install Angular and all required node packages (as listed in `package.json`).

2. **Run the Angular dev server:**

   ```bash
   ng serve --open
   ```

   This will compile the Angular application and serve it locally on [http://localhost:4200/](http://localhost:4200/) (and `--open` will automatically open your web browser). The Angular CLI dev server will proxy API calls to the target defined in `environment.development.ts` (which is `http://localhost:8000/api` by default). Ensure the backend is running on port 8000 as per the above step.

   > **Tip:** The Angular dev server is set up such that any unknown requests (like `/api/...`) will be forwarded to the backend (this is configured by the dev environment setting). If you have issues with API calls, check that the backend is running and CORS is enabled (it is configured to allow localhost by default).

3. **Using the app:** Once `ng serve` is running, you can navigate to `http://localhost:4200/` in your browser. You should see the DataDrift home page. From here, you can register a new user or log in. Use the app and the network requests should go to your local Django API. The site will live-reload as you make changes in the code.

**Running Tests:**

* **Frontend tests:** You can run `npm test` (which invokes `ng test` via Karma) to execute the Angular unit tests. By default, this will run in headless Chrome and report the results in the console.
* **Backend tests:** To run the Django tests, make sure your virtualenv is active and run:

  ```bash
  python manage.py test
  ```

  This will run the test suites under `backend/users/tests/` and `backend/articles/tests/`. The tests will spin up a temporary SQLite database by default (since Django test runner configures that) and execute numerous API requests to verify correctness (user registration, login, article creation, etc.). All tests should pass if the setup is correct.

**Development Tips:**

* When both servers are running, you can develop the frontend and backend independently. If you change backend code, you might need to restart the Django dev server. If you change frontend code, Angular’s live reload will refresh the page.
* The Angular app is configured to automatically reload on file changes during `ng serve`. The Django server will need manual restart if code changes (unless you use an extension like `django-autoreload`, but by default runserver includes auto-reload for code changes).
* You can use the Django admin at `http://localhost:8000/admin/` to inspect the database (use the superuser account you created to login).

## Deployment

Deploying DataDrift Fullstack Blog to a production environment can be done using the provided Docker configuration or by manually deploying the front and back ends. Some deployment options:

* **Docker Compose (Production):** You can use a similar approach as local Docker, but on a server. For example, copy the code to a VM or cloud instance that has Docker installed, ensure the `.env` is configured for production (e.g., set `DEBUG=False`, use a strong `SECRET_KEY`, and proper domain in `ALLOWED_HOSTS`), and run `docker-compose up -d --build`. This will build and launch the three containers. It is generally recommended to use a reverse proxy (like Nginx or Caddy) in front of the frontend container for TLS/SSL termination and domain routing, but the provided Nginx (serving the Angular app) could also be configured for SSL if needed.
* **Docker Images:** The project includes Dockerfiles for the frontend and backend, so you can build these images individually and deploy to a container registry. For example, build the backend image: `docker build -t myregistry/datadrift-backend:latest ./backend` and similarly for the frontend. The backend image will run the Django app via Gunicorn, and the frontend image will serve the Angular app via Nginx. You could deploy these images to a container orchestration platform (Docker Swarm, Kubernetes, etc.) and provision a Postgres database (or use a managed database service). Make sure to supply environment variables in production for database credentials and other secrets.
* **Manual Deployment:** Alternatively, you might host the frontend on a static hosting service and the backend on a PaaS or VM:

  * **Frontend on static host:** Since the Angular app is a static bundle, you can deploy the contents of `frontend/dist/frontend/browser/` (produced by `ng build`) to any static file hosting (AWS S3+CloudFront, Vercel, Netlify, GitHub Pages, etc.). Ensure it’s served at the same domain as the API or configure CORS accordingly. The router is an SPA, so you need to enable fallback to `index.html` for unknown routes (most static hosts have an option for this).
  * **Backend on server/PaaS:** The Django app can be deployed on any platform that supports Python (Heroku, AWS Elastic Beanstalk, DigitalOcean App Platform, etc.). Use Gunicorn as the server and possibly Nginx as a reverse proxy. Remember to set environment variables for `SECRET_KEY`, database, etc. and run `manage.py migrate` and createsuperuser as needed. When deploying to Heroku or similar, you might use a Heroku Postgres addon for the database, and adjust `ALLOWED_HOSTS` and CORS settings to allow the frontend’s domain.
  * If frontend is on a different domain than backend, you must enable CORS for that domain in Django settings (the current config allows all origins or specifically configured ones via `.env`).

For a straightforward deployment scenario, using Docker Compose on a single host (VM or server) is recommended, as it closely mirrors the development setup and ensures consistency. You would just need to handle setting up HTTPS (for example, using Let’s Encrypt with a proxy in front of the Docker setup).

**Performance and Scalability:** The application is suitable for hobby/demo use out of the box. For higher load, you might consider scaling out:

* Running multiple Gunicorn workers or replicating the backend container for more throughput.
* Using a load balancer in front of multiple frontend containers (if not serving static files via CDN).
* Offloading static file serving to a CDN or the Nginx in frontend container (which is already in place).
* Using a managed Postgres with proper scaling and backups.

## Authors and Contact

**Author:** The project was implemented by [**Wojciech512**](https://github.com/Wojciech512). (This is the GitHub handle of the author of DataDrift. You can find the source code and contribution history on their GitHub profile.)

For any inquiries, suggestions, or contributions, you can reach out by:

* Opening an issue or discussion on the [GitHub repository](https://github.com/Wojciech512/DataDrift-Fullstack-Blog).
* Contacting the author via the contact info on their GitHub profile (if available) or the email associated with commits.
* You may also fork the repository and submit pull requests for any improvements.

**Contributions** are welcome. If you encounter any issues or have ideas for enhancements, feel free to create an issue. This project is open-source, so the community can learn from it or build upon it.

---

*Thank you for checking out DataDrift Fullstack Blog! We hope this README provides a clear guide to understanding and running the project. Enjoy sharing your knowledge on the platform!* 🚀
