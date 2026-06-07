# Developer Resource Tracker

A simple full-stack MERN application built to organize and track learning resources. This project represents my transition from pure full-stack development into understanding how modern web applications are configured, secured, and deployed.

## 🚀 Live Application
* **Frontend UI:** [https://dev-resource-tracker-api.netlify.app](https://dev-resource-tracker-api.netlify.app)
* **Backend API:** [https://dev-resource-tracker-api.onrender.com/api/resources](https://dev-resource-tracker-api.onrender.com/api/resources)

## 💡 Why I Built This
I kept losing track of tutorials, articles, and courses I wanted to study. Instead of using scattered bookmarks, I built this application to centralize my learning resources with priority levels and completion tracking.

## 📦 Tech Stack
* **Frontend:** React (v19), Vite, Tailwind CSS (v4), Axios
* **Backend:** Node.js, Express, MongoDB, Mongoose
* **Deployment Automation:** Netlify (Frontend) & Render (Backend)

---

## 🔍 The DevOps Shift: Configuration & Audit Journey

As I transitioned my focus toward **DevOps and Cloud Engineering**, I used this functional application as a deployment target to practice environment management, configuration hardening, and runtime tracking. 

I conducted a baseline configuration audit on this repository and implemented the following enhancements:

### 1. Environment Documentation & Secrets Management
* **The Baseline:** The app relied on implicit local fallbacks if environment variables were missing.
* **The Practice:** Added `.env.example` configurations to both `client/` and `server/` directories. This establishes an explicit structural blueprint for the system's dependencies without leaking private production keys to source control.

### 2. Infrastructure Security (CORS Hardening)
* **The Baseline:** The API layer originally exposed wide-open resource access via unconfigured CORS middleware (`app.use(cors())`).
* **The Practice:** Hardened the CORS controls to whitelist specific production origins and standard local staging ports, ensuring proper network-level access rules.

### 3. Runtime Environment Pinning
* **The Baseline:** The Node.js platform version requirements were undefined, posing a risk of silent deployment failures due to host environment discrepancies.
* **The Practice:** Integrated the `"engines"` specification field directly into both `package.json` files to explicitly bind the application deployment stability to Node.js `>=18.0.0`.

---

### 🔮 Future Operational Roadmap
To align this project with my ongoing AWS and systems-thinking coursework, my upcoming infrastructure milestones for this repository are:
1. **Containerization:** Write optimized multi-stage `Dockerfiles` for the decoupled frontend and backend layouts.
2. **Local Orchestration:** Implement a `docker-compose.yml` file to spin up the entire application stack and isolated database services cleanly with a single command.
