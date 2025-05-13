# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This repository is a copy of the OWASP Juice Shop project, a deliberately insecure web application for security training. The project includes a Jenkinsfile for implementing security tools as part of the CI/CD pipeline.

## Build & Run Commands

### Installation
```
npm install
```

### Building
- Build frontend: `npm run build:frontend`
- Build server: `npm run build:server`
- Package application: `npm run package`

### Running the Application
- Start application (production): `npm start`
- Start application with frontend dev server: `npm run serve`
- Start application with hot reloading: `npm run serve:dev`

### Testing
- Run all tests: `npm test`
- Run frontend tests: `cd frontend && ng test`
- Run server tests: `npm run test:server`
- Run API tests: `npm run test:api`
- Run Cypress tests: `npm run cypress:run`
- Open Cypress test runner: `npm run cypress:open`

### Linting & Code Quality
- Lint all code: `npm run lint`
- Lint and fix code: `npm run lint:fix`
- Validate configuration schema: `npm run lint:config`

## Security Testing Tools
The project includes OWASP ZAP for security scanning:
- ZAP passive scanning configuration is available in the `passive.yaml` file
- ZAP reports are stored in the `zap/reports` directory

## Architecture Overview

### Backend
- Node.js/Express application written in TypeScript
- SQLite database with Sequelize ORM
- Socket.io for real-time communication
- Various security vulnerabilities deliberately implemented for training

### Frontend
- Angular application
- Material UI components
- i18n internationalization support
- Various security vulnerabilities deliberately implemented for training

### Key Directories
- `/routes`: Express API routes
- `/models`: Sequelize data models
- `/frontend/src/app`: Angular application code
- `/lib`: Utility functions and middleware
- `/data`: Data generation and storage utilities
- `/test`: Test files for backend and API

## Development Workflow
1. Install dependencies
2. Run the application in development mode
3. Make changes to the code
4. Run tests to ensure functionality
5. Check security implications using security testing tools
6. Update the Jenkinsfile with new security tools as needed