# Developer Resource Tracker

A simple full-stack MERN application built to organize and track learning resources. This project represents my transition from pure full-stack development into understanding how modern web applications are configured, secured, and deployed.

## 🚀 Live Application

- **Frontend UI:** [https://dev-resource-tracker-api.netlify.app](https://dev-resource-tracker-api.netlify.app)
- **Backend API:** [https://dev-resource-tracker-api.onrender.com/api/resources](https://dev-resource-tracker-api.onrender.com/api/resources)

## 💡 Why I Built This

I kept losing track of tutorials, articles, and courses I wanted to study. Instead of using scattered bookmarks, I built this application to centralize my learning resources with priority levels and completion tracking.

## 📦 Tech Stack

- **Frontend:** React (v19), Vite, Tailwind CSS (v4), Axios
- **Backend:** Node.js, Express, MongoDB, Mongoose
- **Deployment Automation:** Netlify (Frontend) & Render (Backend)

---

## 🔍 The DevOps Shift: Configuration & Audit Journey

As I transitioned my focus toward **DevOps and Cloud Engineering**, I used this functional application as a deployment target to practice environment management, containerization, configuration hardening, and runtime tracking.

I conducted a baseline configuration audit on this repository and implemented the following enhancements:

### 1. Environment Documentation & Secrets Management

- **The Baseline:** The app relied on implicit local fallbacks if environment variables were missing.
- **The Practice:** Added `.env.example` configurations to both `client/` and `server/` directories. This establishes an explicit structural blueprint for the system's dependencies without leaking private production keys to source control.

### 2. Multi-Stage Containerization (Docker)

- **Frontend (`client/`):** Implemented a production-grade **Multi-Stage Docker build**. Stage 1 uses a heavy Node image to compile the React/Vite assets, while Stage 2 throws away the compilation bloat and transfers only the raw static files into a lightweight, high-performance **Nginx Alpine** web server container. This dropped the production footprint significantly.
- **Backend (`server/`):** Built a hardened Node.js Docker configuration enforcing the **Least-Privilege Security Principle**. Default root administrative access is removed by recursively transferring application ownership to an isolated, non-root `node` system profile inside the image, which helps mitigate container breakout risks.

### 3. Multi-Container Local Orchestration (`docker-compose`)

- Integrated a master orchestration template (`docker-compose.yml`) that unites the frontend UI, backend API, and a dedicated **MongoDB 7.0 database volume**.
- **Network Isolation:** Configured an isolated internal bridge network. The database layer does not expose any open ports to the host machine or public internet; it communicates strictly with the backend API service inside the secure cluster network.
- **Data Persistence:** Wired named Docker volumes mapping container data straight to the host hard drive, preventing data loss when containers cycle down.

### 4. Infrastructure Security (CORS Hardening)

- **The Baseline:** The API layer originally exposed wide-open resource access via unconfigured CORS middleware (`app.use(cors())`).
- **The Practice:** Hardened the CORS controls to whitelist specific production origins and standard local staging ports, ensuring proper network-level access rules.

### 5. Runtime Environment Pinning

- **The Baseline:** The Node.js platform version requirements were undefined, posing a risk of silent deployment failures due to host environment discrepancies.
- **The Practice:** Integrated the `"engines"` specification field directly into both `package.json` files to explicitly bind the application deployment stability to Node.js `>=18.0.0`.

### 6. Supply Chain Security & Linting

- Applied pre-commit operational gating pipelines and utilized specialized DevSecOps static analysis orchestration to continuously scan development variables, intercepting secret leaks like `.env.production` files before they could enter source control.

---

### 🔮 Future Operational Roadmap

To align this project with my ongoing AWS and systems-thinking coursework, my upcoming infrastructure milestones for this repository are:

1. **Infrastructure as Code (IaC):** Translate this architectural stack into **Terraform configuration files** to declare cloud infrastructure programmatically.
2. **Cloud Migration:** Deploy the decoupled frontend to **AWS S3/CloudFront** and transition the backend API onto **AWS ECS Fargate** with an Application Load Balancer.
