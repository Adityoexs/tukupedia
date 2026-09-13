# Tukupedia Development Setup Guide

## 📋 Prerequisites

Before starting, ensure you have:

- **Docker & Docker Compose** (for database & services)
  - Download: https://www.docker.com/products/docker-desktop
  
- **Node.js 18+** (for frontend)
  - Download: https://nodejs.org/
  - Verify: `node --version && npm --version`
  
- **Git**
  - Download: https://git-scm.com/
  - Verify: `git --version`

- **.NET 8 SDK** (for backend, optional for development)
  - Download: https://dotnet.microsoft.com/download/dotnet/8.0
  - Verify: `dotnet --version`

- **PostgreSQL Client** (optional, for manual DB operations)
  - `psql` command line tool

## 🚀 Quick Setup (5 minutes)

### 1. Clone Repository

```bash
git clone https://github.com/Adityoexs/tukupedia.git
cd tukupedia
```

### 2. Setup Environment

```bash
# Copy environment template
cp .env.example .env

# Default values in .env:
# DATABASE_URL=postgresql://tukupedia:tukupedia@localhost:5432/tukupedia
# REDIS_URL=redis://localhost:6379
# JWT_SECRET=your-secret-key-here-change-in-production
# API_URL=http://localhost:5000
# FRONTEND_URL=http://localhost:3000
```

### 3. Start Services with Docker Compose

```bash
# Start PostgreSQL, Redis, and other services
docker-compose up -d

# Verify services are running
docker-compose ps

# Output should show:
# - postgres (postgresql)
# - redis (redis)
```

### 4. Setup Backend

```bash
cd backend

# Restore NuGet packages
dotnet restore

# Apply database migrations
dotnet ef database update --project Tukupedia.Data --startup-project Tukupedia.API

# Run seed script (optional, for demo data)
# Will create sample products, categories, users, etc.

cd ..
```

### 5. Setup Frontend

```bash
cd frontend

# Install dependencies
npm install

# Build PWA assets
npm run build:pwa

cd ..
```

### 6. Start Development Servers

**Terminal 1 - Backend API:**
```bash
cd backend
dotnet run --project Tukupedia.API
# Runs on http://localhost:5000
```

**Terminal 2 - Frontend:**
```bash
cd frontend
npm run dev
# Runs on http://localhost:3000
```

### 7. Verify Everything Works

- Frontend: http://localhost:3000
- Backend API: http://localhost:5000
- GraphQL Playground: http://localhost:5000/graphql
- PostgreSQL: localhost:5432
- Redis: localhost:6379

---

## 📦 Detailed Setup Instructions

### Backend Setup (.NET 8)

#### Project Structure
```
backend/
├── Tukupedia.API/           # Main API project (entry point)
│   ├── Controllers/         # REST controllers
│   ├── GraphQL/             # GraphQL schema & resolvers
│   ├── Services/            # Business logic
│   ├── Middleware/          # Custom middleware
│   ├── appsettings.json     # Configuration
│   └── Program.cs           # Startup configuration
│
├── Tukupedia.Core/          # Domain models & interfaces
│   ├── Entities/            # Database models
│   ├── DTOs/                # Data Transfer Objects
│   └── Interfaces/          # Service interfaces
│
├── Tukupedia.Data/          # Data access layer
│   ├── Contexts/            # DbContext
│   ├── Repositories/        # Repository pattern
│   ├── Migrations/          # EF Core migrations
│   └── Seeders/             # Database seeders
│
├── Tukupedia.Service/       # Service layer
│   ├── Services/            # Business logic
│   └── Interfaces/          # Service interfaces
│
└── Tukupedia.sln            # Solution file
```

#### Install .NET Dependencies

```bash
cd backend

# Restore all NuGet packages
dotnet restore

# Packages included:
# - EntityFrameworkCore (ORM)
# - HotChocolate (GraphQL)
# - Npgsql (PostgreSQL driver)
# - StackExchange.Redis (Redis client)
# - AspNetCore.Identity (Authentication)
# - Serilog (Logging)
```

#### Database Migration

```bash
cd backend

# Create initial migration (if needed)
dotnet ef migrations add InitialCreate \
  --project Tukupedia.Data \
  --startup-project Tukupedia.API \
  --output-dir Migrations

# Apply migrations to database
dotnet ef database update \
  --project Tukupedia.Data \
  --startup-project Tukupedia.API

# Verify database created
# Use pgAdmin or psql to connect to PostgreSQL
psql -h localhost -U tukupedia -d tukupedia -c "\dt"
```

#### Seed Sample Data

```bash
cd backend

# Run seeder to populate sample data
# Script will create:
# - 5 categories
# - 20 sample products
# - 3 demo users
# - 50 product reviews

dotnet run --project Tukupedia.API -- --seed
```

#### Run Backend Server

```bash
cd backend

# Development mode (with hot reload)
dotnet watch run --project Tukupedia.API

# Or standard run
dotnet run --project Tukupedia.API

# Server starts at:
# - API: http://localhost:5000/api
# - GraphQL: http://localhost:5000/graphql
# - Health: http://localhost:5000/health
```

#### Test Backend

```bash
cd backend

# Run all tests
dotnet test

# Run specific test project
dotnet test Tukupedia.Tests.Unit

# With coverage
dotnet test /p:CollectCoverage=true
```

---

### Frontend Setup (Next.js)

#### Project Structure
```
frontend/
├── app/
│   ├── (auth)/              # Auth pages (login, register)
│   ├── (shop)/              # Shop pages (products, cart, checkout)
│   ├── (account)/           # User account pages
│   ├── admin/               # Admin dashboard
│   └── layout.tsx           # Root layout
│
├── components/              # Reusable components
│   ├── ui/                  # UI components (button, card, etc)
│   ├── product/             # Product components
│   ├── cart/                # Cart components
│   └── common/              # Common components
│
├── lib/
│   ├── graphql/             # GraphQL client & queries
│   ├── hooks/               # Custom React hooks
│   ├── utils/               # Utility functions
│   └── store.ts             # Zustand state management
│
├── public/
│   ├── manifest.json        # PWA manifest
│   └── service-worker.js    # Service worker
│
├── styles/
│   └── globals.css          # Global styles
│
├── next.config.js           # Next.js config + PWA
├── tailwind.config.js       # Tailwind config
├── tsconfig.json            # TypeScript config
└── package.json
```

#### Install Node Dependencies

```bash
cd frontend

# Install all dependencies
npm install

# Dependencies include:
# - next (React framework)
# - tailwindcss (Styling)
# - @apollo/client (GraphQL client)
# - zustand (State management)
# - react-query (Data fetching)
# - next-pwa (PWA support)
# - react-hook-form (Form handling)
# - zod (Validation)
```

#### PWA Setup

```bash
cd frontend

# Build PWA assets
npm run build:pwa

# Generates:
# - public/manifest.json
# - public/icons/
# - public/service-worker.js
```

#### Environment Variables

```bash
# frontend/.env.local
NEXT_PUBLIC_API_URL=http://localhost:5000
NEXT_PUBLIC_GRAPHQL_URL=http://localhost:5000/graphql
NEXT_PUBLIC_APP_NAME=Tukupedia
NEXT_PUBLIC_APP_VERSION=1.0.0
```

#### Run Frontend Development Server

```bash
cd frontend

# Development mode with hot reload
npm run dev

# Server starts at http://localhost:3000

# Additional commands:
npm run build          # Production build
npm run start          # Production server
npm run lint           # ESLint
npm run type-check     # TypeScript check
npm run test           # Run tests (Jest)
```

#### Test Frontend

```bash
cd frontend

# Run all tests
npm run test

# Watch mode
npm run test:watch

# Coverage report
npm run test:coverage
```

---

### Database Setup

#### PostgreSQL Connection

```bash
# Connect to PostgreSQL
psql -h localhost -p 5432 -U tukupedia -d tukupedia

# Common commands:
\dt                 # List all tables
\d table_name       # Describe table
SELECT * FROM products LIMIT 10;  # Query
\q                  # Quit
```

#### pgAdmin Web UI (Optional)

```bash
# pgAdmin is included in docker-compose.yml
# Access at: http://localhost:5050

# Default credentials (in docker-compose.yml):
# Email: admin@admin.com
# Password: admin
```

#### Run SQL Scripts Manually

```bash
# Apply migration script
psql -h localhost -U tukupedia -d tukupedia < database/migrations/001_initial_schema.sql

# Seed data
psql -h localhost -U tukupedia -d tukupedia < database/seeders/seed_categories.sql
psql -h localhost -U tukupedia -d tukupedia < database/seeders/seed_products.sql
```

---

### Docker Compose Services

#### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f postgres
docker-compose logs -f redis

# Follow backend logs
docker-compose logs -f backend
```

#### Stop/Start Services

```bash
# Stop all services
docker-compose down

# Stop specific service
docker-compose stop postgres

# Restart services
docker-compose restart

# Remove volumes (careful! deletes data)
docker-compose down -v
```

#### Access Service Shells

```bash
# PostgreSQL shell
docker-compose exec postgres psql -U tukupedia -d tukupedia

# Redis CLI
docker-compose exec redis redis-cli

# Backend container
docker-compose exec backend bash
```

---

## 🧪 Testing & Verification

### Frontend Testing

```bash
cd frontend

# Run tests
npm run test

# Test specific file
npm run test -- product.test.tsx

# Watch mode
npm run test:watch

# Coverage report
npm run test:coverage
```

### Backend Testing

```bash
cd backend

# Run all tests
dotnet test

# Run specific test class
dotnet test --filter ClassName=ProductServiceTests

# Verbose output
dotnet test -v detailed
```

### API Testing (Postman/cURL)

```bash
# Test REST API
curl -X GET http://localhost:5000/api/products

# Test GraphQL
curl -X POST http://localhost:5000/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ products { id name price } }"}'

# Health check
curl http://localhost:5000/health
```

---

## 🐛 Troubleshooting

### Port Already in Use

```bash
# Backend (5000)
lsof -i :5000
kill -9 <PID>

# Frontend (3000)
lsof -i :3000
kill -9 <PID>

# PostgreSQL (5432)
lsof -i :5432

# Redis (6379)
lsof -i :6379
```

### Database Connection Error

```bash
# Verify PostgreSQL is running
docker-compose ps

# Check PostgreSQL logs
docker-compose logs postgres

# Manually connect
psql -h localhost -U tukupedia -d tukupedia

# If container error, rebuild
docker-compose down
docker-compose up -d postgres --build
```

### Redis Connection Error

```bash
# Test Redis connection
redis-cli -h localhost -p 6379 ping

# Should return: PONG

# If error, rebuild
docker-compose down
docker-compose up -d redis --build
```

### Node Modules Issues

```bash
# Clear cache & reinstall
cd frontend
rm -rf node_modules package-lock.json
npm install
npm run build:pwa
```

### .NET Build Issues

```bash
# Clean build
cd backend
dotnet clean
dotnet restore
dotnet build

# If migration issues
dotnet ef database drop --force
dotnet ef database update
```

---

## 📚 Additional Resources

### Documentation
- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [DATABASE.md](DATABASE.md) - Database schema
- [API.md](API.md) - API documentation

### Official Docs
- [Next.js Docs](https://nextjs.org/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [ASP.NET Core](https://docs.microsoft.com/aspnet/core)
- [PostgreSQL](https://www.postgresql.org/docs/)
- [Redis](https://redis.io/documentation)
- [HotChocolate GraphQL](https://chillicream.com/docs/hotchocolate)

### Useful Tools
- [GraphQL Playground](http://localhost:5000/graphql) - GraphQL IDE
- [pgAdmin](http://localhost:5050) - PostgreSQL Web UI
- [Docker Desktop](https://www.docker.com/products/docker-desktop) - Container management

---

## ✅ Setup Checklist

- [ ] Clone repository
- [ ] Copy `.env.example` to `.env`
- [ ] Run `docker-compose up -d`
- [ ] Setup backend (migrations, seed data)
- [ ] Setup frontend (npm install)
- [ ] Start backend server (dotnet run)
- [ ] Start frontend server (npm run dev)
- [ ] Verify all services running
- [ ] Test frontend at http://localhost:3000
- [ ] Test backend at http://localhost:5000/graphql
- [ ] Ready to develop! 🚀

---

**Last Updated**: 2024
