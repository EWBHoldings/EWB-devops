# React App — Project Template

This template provides the recommended starting structure for a React application within the EWB ecosystem. It includes a pre-configured QA pipeline that calls the EWB-devops `react-qa.yml` reusable workflow.

---

## Using This Template

1. Copy the contents of this directory into your new repository
2. Replace `your-org` in `.github/workflows/qa.yml` with your GitHub organisation name
3. Update `README.md` with project-specific information
4. Install dependencies and verify the project runs locally
5. Push to GitHub and confirm the QA pipeline executes on the first pull request

---

## Recommended Project Structure

```
my-react-app/
├── public/
│   └── index.html
├── src/
│   ├── components/          # Reusable UI components
│   │   └── ExampleComponent/
│   │       ├── ExampleComponent.tsx
│   │       ├── ExampleComponent.test.tsx
│   │       └── index.ts
│   ├── pages/               # Route-level page components
│   ├── hooks/               # Custom React hooks
│   ├── services/            # API call functions
│   ├── store/               # Global state (Zustand / Redux Toolkit)
│   ├── types/               # Shared TypeScript interfaces and types
│   ├── utils/               # Pure utility functions
│   ├── App.tsx
│   └── main.tsx
├── .github/
│   └── workflows/
│       └── qa.yml           # Pre-configured QA pipeline
├── .eslintrc.js
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

---

## Recommended package.json Scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint src --ext .ts,.tsx --report-unused-disable-directives --max-warnings 0",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

## Recommended Dependencies

**Core:**

```json
{
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "react-router-dom": "^6.x",
    "@tanstack/react-query": "^5.x"
  },
  "devDependencies": {
    "typescript": "^5.x",
    "vite": "^5.x",
    "@vitejs/plugin-react": "^4.x",
    "vitest": "^1.x",
    "@testing-library/react": "^15.x",
    "@testing-library/jest-dom": "^6.x",
    "@testing-library/user-event": "^14.x",
    "eslint": "^8.x",
    "@typescript-eslint/eslint-plugin": "^7.x",
    "@typescript-eslint/parser": "^7.x"
  }
}
```

---

## Environment Variables

Create a `.env.local` file locally (never commit this file):

```
VITE_API_BASE_URL=http://localhost:3000/api
```

Document all required variables in `.env.example`:

```
VITE_API_BASE_URL=https://api.yourapp.com
```

---

## Coding Standards

See [React Coding Standards](../../docs/coding-standards/react-coding-standards.md).
