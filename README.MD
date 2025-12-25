# Canva Clone

🎨 Build a Full Canva Clone with Next.js, Node.js Microservices & Fabric.js

This repository is a full-stack Canva-style design editor built as a learning project and demo. It demonstrates a modern, scalable architecture combining a Next.js front-end with a Node.js API gateway and several microservices (Upload, Design, Subscription), using Fabric.js for a powerful canvas editing experience and MongoDB for persistence.

## Table of contents

- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [Architecture & Tech Stack](#architecture--tech-stack)
- [Repository Layout](#repository-layout)
- [Prerequisites](#prerequisites)
- [Environment Variables](#environment-variables)
- [Local Development](#local-development)
- [Services Details](#services-details)
- [Canvas Editor Features](#canvas-editor-features)
- [AI & Upload Integrations](#ai--upload-integrations)
- [Billing & Subscription Flow](#billing--subscription-flow)
- [Exporting & Storage](#exporting--storage)
- [Testing](#testing)
- [Deployment Suggestions](#deployment-suggestions)
- [Contributing](#contributing)
- [License (MIT)](#license-mit)

---

## Project Overview

This project reproduces a simplified but feature-rich Canva-like web application with a focus on:

- A responsive Next.js app (client) using TailwindCSS and Shadcn UI components for polished UI.
- A microservice backend (Node.js) with an API Gateway that routes requests to dedicated services:
  - Upload Service: handles image uploads and Cloudinary integration.
  - Design Service: persists user designs, autosave, history and export metadata.
  - Subscription Service: handles payments and subscription state (PayPal integration demo).
- Authentication via NextAuth (Auth.js) with token forwarding to backend services via middleware.
- A canvas editor built on Fabric.js supporting shapes, text, drawing, image placement, export, and auto-save.

This is a learning/demo project intended to show how to structure a modern full-stack editor app using microservices and a React/Next frontend.

## Key Features

- Next.js client with Shadcn UI + TailwindCSS for responsive and accessible UI.
- API Gateway forwarding requests to microservices (Upload, Design, Subscription).
- Auth.js (NextAuth v5) integration with token forwarding via middleware.
- Fabric.js based canvas editor with:
  - Add/manipulate shapes & text
  - Freehand drawing
  - Image uploads & placement
  - Lock/unlock canvas editing mode
  - Custom property editor and smart shape factory
  - Auto-save, load, and design history
- AI image generation panel (external API integration) with gating for non-premium users.
- Cloud uploads (Cloudinary) with preview and easy canvas integration.
- Export designs as PNG, JPG, SVG or JSON project files.
- Premium subscription flow (PayPal integration demo), gating features and limiting free users (limit: 5 designs by default).

## Architecture & Tech Stack

- Frontend: Next.js (app router) + React + TailwindCSS + Shadcn UI
- Canvas: Fabric.js
- State Management: Zustand (client-side store)
- Backend: Node.js microservices (Express style) behind an API Gateway
- Database: MongoDB (one DB per service or single DB with separate collections depending on deployment)
- Storage: Cloudinary for media uploads
- Auth: NextAuth (Auth.js) on Next.js, forwarding tokens to services
- Payments: PayPal (server-side subscription flow)
- AI: External AI image generation API (pluggable)

## Repository Layout

Root contains two top-level folders: `client` and `server`.

- `client/` — Next.js app and UI components

  - `app/` — Next.js app router pages and API adapters
  - `components/` — UI and editor components (editor, header, sidebar, etc.)
  - `fabric/` — Fabric.js utilities, shape factory and definitions
  - `services/` — client-side API wrappers (design-service, upload-service, subscription-service)
  - `store/` — Zustand store configuration

- `server/` — microservices + API gateway
  - `api-gateway/` — gateway application that implements routing, auth-forwarding, and request composition
  - `design-service/` — design persistence (controllers, models, routes)
  - `upload-service/` — uploads and Cloudinary integration
  - `subscription-service/` — billing and subscription management (PayPal related)

## Prerequisites

- Node.js (v18+ recommended)
- npm or pnpm
- MongoDB (local or Atlas)
- Cloudinary account (for uploads) — or mock uploads during local development
- PayPal developer account for subscription integration (optional for local/testing)
- An API key for your chosen AI image generation provider (if using AI panel)

## Environment Variables

Create `.env` files for the client and each service. Below is a suggested `.env.example` split by component.

Client (`client/.env.local`)

```
NEXT_PUBLIC_API_GATEWAY_URL=http://localhost:8080
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your_nextauth_secret
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your_cloud_name
NEXT_PUBLIC_AI_PROVIDER_BASE_URL=https://api.example.com
NEXT_PUBLIC_AI_PROVIDER_KEY=your_ai_key
```

API Gateway / Services (each service gets a `.env` file):

Common for services

```
PORT= (service-specific port)
MONGODB_URI=mongodb://localhost:27017/canva_clone_<service>
API_GATEWAY_KEY=an_internal_key_used_by_gateway
JWT_SECRET=your_jwt_secret (if using JWTs)
```

Upload Service (`server/upload-service/.env`)

```
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

Subscription Service (`server/subscription-service/.env`)

```
PAYPAL_CLIENT_ID=your_paypal_client_id
PAYPAL_CLIENT_SECRET=your_paypal_secret
WEBHOOK_SECRET=your_paypal_webhook_secret
```

AI Integration (if separate):

```
AI_API_KEY=your_ai_api_key
AI_API_URL=https://api.example.com/generate
```

## Local Development

General strategy: run MongoDB, start each microservice, start the API gateway, then start the Next.js client.

Example quick steps (run each in its own terminal):

1. Install dependencies

```bash
cd client
npm install

cd ../server/api-gateway
npm install

cd ../design-service
npm install

cd ../upload-service
npm install

cd ../subscription-service
npm install
```

2. Start services (example ports shown)

```bash
# start MongoDB (local) or ensure Atlas connection

# design service
cd server/design-service
npm run dev # expect port e.g. 4001

# upload service
cd ../upload-service
npm run dev # expect port e.g. 4002

# subscription service
cd ../subscription-service
npm run dev # expect port e.g. 4003

# api-gateway
cd ../api-gateway
npm run dev # expect port e.g. 8080

# client (Next.js)
cd ../../../client
npm run dev # runs on http://localhost:3000
```

Notes

- Ports shown above are examples — check each service `package.json` for exact `dev` scripts or configure them using environment variables.
- If you use Docker or docker-compose, you can orchestrate services and MongoDB together (not included here but a recommended next step).

## Services Details

API Gateway

- Routes incoming client requests to microservices, performs authentication token forwarding, basic rate-limiting and request validation.
- Exposes a consolidated API surface to the client (e.g. `/api/design`, `/api/upload`, `/api/subscription`).

Design Service

- Stores designs in MongoDB: a design document includes metadata, owner, thumbnails, exported file references, and a JSON snapshot of the Fabric.js canvas state.
- Supports: create, update (autosave), list, delete, export, and design history endpoints.

Upload Service

- Responsible for uploading files to Cloudinary. Validates file types/sizes, returns hosted URLs and thumbnails.

Subscription Service

- Manages customer subscriptions and billing with PayPal SDK.
- Stores subscription status and gates premium-only features (AI generation, unlimited designs, etc.).

## Canvas Editor Features

- Fabric.js-based editor with modular utilities in `client/fabric/`:
  - `fabric-utils.js` — helpers for object transforms, serialization/deserialization, export helpers.
  - `shape-factory.js` & `shape-definitions.js` — central shape creation for consistent styling and property hooks.
- Sidebar panels for Shapes, Uploads, Text, Draw, and AI panel.
- Custom property editor supporting typed inputs (color, fonts, size, spacing) and custom controls.
- Canvas Lock/Unlock mode to restrict accidental moves/edits while fine-tuning layout.
- Auto-save: periodically POSTs design snapshots to the Design Service and updates UI save state.

## AI & Upload Integrations

- AI Image Generation: the client provides an AI panel to generate imagery using an external API. Responses can be inserted directly onto the canvas.
- AI gating: free users are limited from using AI features unless they upgrade.
- Upload flow: files uploaded using the Upload Service are returned as URLs and optional cloud IDs; images can be fetched and inserted onto the canvas.

## Billing & Subscription Flow

- The subscription service uses PayPal SDK to create and verify subscriptions.
- The client opens upgrade dialogs and redirects to the subscription flow; on success the subscription service updates the user's subscription state.
- The Design Service and UI consult subscription state to enforce free-user limits (e.g., 5 designs) and feature gating.

## Exporting & Storage

- The editor supports exporting current canvas into PNG, JPG, SVG and JSON (Fabric.js serialized state).
- Exported files can be uploaded to Cloudinary (optional) for persistence or downloaded directly by the user.

## Testing

- Unit and integration tests are recommended per-service; this repo focuses on architecture and example integrations.
- If you add tests, follow the same patterns across services (Jest + Supertest for Node services, React Testing Library for React components).

## Deployment Suggestions

- Frontend: deploy to Vercel or Netlify (Next.js optimized) — ensure `NEXTAUTH_URL` and API gateway URL are set for production.
- Services: each microservice can be deployed on a container platform (Heroku, AWS ECS/Fargate, DigitalOcean App Platform, or Kubernetes).
- Database: use MongoDB Atlas for production and configure strong authentication and network rules.
- Secrets: keep `CLOUDINARY_API_SECRET`, `PAYPAL_CLIENT_SECRET`, and other secrets in a secret manager (AWS Secrets Manager, Azure Key Vault, or environment variables managed by your hosting provider).

## Contributing

Thanks for your interest in contributing! Contributions are welcome — please follow these guidelines:

1. Fork the repository and create a feature branch: `git checkout -b feat/your-feature`.
2. Open a pull request against `main` with a clear description of your change.
3. Keep changes focused and small; split large work into multiple PRs if possible.
4. Follow existing code style and naming conventions. Use Prettier/Eslint if available.
5. Write tests for new functionality when practical.

If you're planning to add major features (new service, breaking API change), open an issue first to discuss the design.

## Code of Conduct

Please be respectful and collaborative. If you want a formal code of conduct file, consider adding one (e.g. Contributor Covenant).

## License (MIT)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
