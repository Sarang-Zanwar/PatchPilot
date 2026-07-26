# PatchPilot

**AI-generated release notes from any two Git commits.**

PatchPilot connects to your GitHub repositories, compares any base and head commit (no tags or GitHub Releases required), and uses an LLM to turn the raw diff into structured, categorized release notes — with every bullet point traceable back to the exact file and lines that produced it.

---

## ✨ Features

- **Sign in with GitHub** — OAuth2 login, exchanged for a first-party JWT session.
- **Repository management** — connect and manage repositories from your GitHub account.
- **Flexible comparisons** — pick any branch and any base/head commit pair.
- **AI release notes** — Google Gemini drafts categorized, human-readable release notes from the diff.
- **Evidence mapping** — every generated item links back to the file and line range that justifies it.
- **History** — every generation is saved and can be revisited or deleted later.

## 🧱 Tech Stack

**Frontend** (`/frontend`)
- [React 19](https://react.dev/) + [TanStack Start](https://tanstack.com/start) (file-based routing, SSR)
- TypeScript
- Tailwind CSS v4 + [shadcn/ui](https://ui.shadcn.com/) (Radix UI primitives)
- TanStack Query for server state, React Hook Form + Zod for forms
- Axios API client, Vite build tooling
- Deploys to Cloudflare Workers (via Nitro)

**Backend** (`/backend`)
- Spring Boot 4 (Java 21)
- Spring Security + OAuth2 (GitHub login) + JWT (jjwt) sessions
- Spring AI + Google Gemini for release-note generation
- MongoDB (Spring Data) for persistence
- springdoc-openapi for Swagger UI docs

## 📂 Project Structure

```
PatchPilot/
├── frontend/          # React + TanStack Start client
│   └── src/
│       ├── routes/       # File-based route definitions
│       ├── pages/        # Page components
│       ├── components/   # UI components (shadcn/ui + app-specific)
│       ├── hooks/        # React Query hooks per domain
│       ├── services/     # API wrapper functions
│       ├── lib/          # apiClient, session, utils
│       └── types/        # Shared TypeScript types
└── backend/            # Spring Boot REST API
    └── src/main/java/com/patchpilot/backend/
        ├── auth/          # Login/session handling
        ├── security/      # JWT + OAuth2 config
        ├── github/        # GitHub API integration
        ├── ai/            # Diff processing, prompting, Gemini provider
        ├── patchnotes/    # Patch note persistence & retrieval
        ├── repo/          # Connected repository management
        └── user/          # User profile
```

## 🚀 Getting Started

### Prerequisites

- **Node.js** 18+ and a package manager (npm, or [bun](https://bun.sh/) — a `bun.lock` is included)
- **Java 21** and Maven (or use the included `./mvnw` wrapper)
- **MongoDB** instance (local or hosted, e.g. MongoDB Atlas)
- A **GitHub OAuth App** ([create one here](https://github.com/settings/developers))
- A **Google Gemini API key** ([Google AI Studio](https://aistudio.google.com/))

### 1. Backend setup

```bash
cd backend
cp src/main/resources/application-local.example.yaml src/main/resources/application-local.yaml
```

Fill in `application-local.yaml` with your own values:

```yaml
spring:
  mongodb:
    uri: <your-mongodb-connection-string>

  ai:
    google:
      genai:
        api-key: <your-gemini-api-key>
        chat:
          options:
            model: gemini-2.5-flash
            temperature: 0.2

  security:
    oauth2:
      client:
        registration:
          github:
            client-id: <your-github-oauth-client-id>
            client-secret: <your-github-oauth-client-secret>
            scope:
              - read:user
              - user:email

jwt:
  secret: <a-long-random-secret>
  expiration: 86400000
```

Set your GitHub OAuth App's **Authorization callback URL** to:
```
http://localhost:8080/login/oauth2/code/github
```

Run the backend (defaults to the `local` Spring profile picking up the file above — activate it via `spring.profiles.active=local` or an environment variable):

```bash
./mvnw spring-boot:run
```

The API will start on `http://localhost:8080`. Swagger UI is available at `http://localhost:8080/swagger-ui.html`.

### 2. Frontend setup

```bash
cd frontend
npm install   # or: bun install
```

Create a `.env` file if your backend isn't running on the default URL:

```bash
VITE_API_BASE_URL=http://localhost:8080
```

Start the dev server:

```bash
npm run dev   # or: bun run dev
```

The app will be available at `http://localhost:5173`.

### 3. Log in

Open the app, click **Sign in with GitHub**, authorize the OAuth app, connect a repository, pick a base and head commit, and generate your first set of release notes.

## 🛠️ Available Scripts (frontend)

| Script | Description |
|---|---|
| `npm run dev` | Start the Vite dev server |
| `npm run build` | Production build (targets Cloudflare Workers via Nitro) |
| `npm run preview` | Preview the production build locally |
| `npm run lint` | Run ESLint |
| `npm run format` | Format the codebase with Prettier |

## 🔒 Security Notes

- The GitHub OAuth token used to call the GitHub API never touches the browser — it's stored server-side and used only by the backend.
- The frontend only ever holds a short-lived, first-party JWT, kept in `sessionStorage` (cleared on tab close, never written to `localStorage`).
- All backend endpoints except health checks and the OAuth/login/docs routes require a valid JWT.

## 📄 License

Add your preferred license here (e.g. MIT).

## 🤝 Contributing

Issues and pull requests are welcome. Please open an issue first to discuss significant changes.
