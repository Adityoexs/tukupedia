# Tukupedia - E-Commerce Platform

Tukupedia adalah platform e-commerce modern yang terinspirasi dari Tokopedia dan Matahari, dibangun dengan teknologi stack terkini untuk memberikan pengalaman belanja online yang seamless dan scalable.

## 🎯 Visi & Misi

- **Visi**: Menjadi platform e-commerce terpercaya dengan teknologi modern dan user experience terbaik
- **Misi**: Menghubungkan penjual dan pembeli dalam ekosistem digital yang aman dan efisien

## 🏗️ Tech Stack

### Frontend
- **Framework**: Next.js 14+ (React, App Router)
- **Styling**: Tailwind CSS
- **State Management**: TanStack Query + Zustand
- **PWA**: next-pwa, Service Workers, Offline Support
- **Features**: SSR/ISR untuk SEO, Dark Mode Ready

### Backend
- **Runtime**: .NET 8 (ASP.NET Core)
- **API**: GraphQL (HotChocolate) + REST (hybrid)
- **Language**: C#

### Database & Cache
- **Database**: PostgreSQL (Relasional + JSONB untuk atribut dinamis)
- **Search**: SQLite FTS (MVP), scalable ke Elasticsearch
- **Cache**: Redis (session, cart, query cache)
- **Storage**: S3-compatible (MinIO/AWS S3)

### Authentication
- **Auth Service**: ASP.NET Identity + JWT
- **Providers**: Email/Password, Google OAuth, Apple OAuth

### DevOps & Deployment
- **Containerization**: Docker
- **Orchestration**: Docker Compose (local dev), Kubernetes-ready
- **CI/CD**: GitHub Actions

## 📁 Project Structure

```
tukupedia/
├── frontend/                 # Next.js PWA Application
│   ├── app/
│   │   ├── (auth)/          # Login, Register, OAuth
│   │   ├── (shop)/          # Katalog, Produk, Cart, Checkout
│   │   ├── (account)/       # Profile, Orders, Wishlist
│   │   └── admin/           # Admin Dashboard
│   ├── components/          # Reusable components
│   ├── lib/                 # Utilities, API client
│   ├── public/              # Static assets
│   ├── styles/              # Global styles
│   ├── next.config.js       # Next.js config + PWA
│   ├── tailwind.config.js   # Tailwind config
│   └── package.json
│
├── backend/                  # .NET 8 API Server
│   ├── Tukupedia.API/       # Main API project
│   │   ├── Controllers/     # REST endpoints
│   │   ├── GraphQL/         # GraphQL resolvers & types
│   │   ├── Services/        # Business logic
│   │   ├── Models/          # DTOs & ViewModels
│   │   ├── Program.cs       # Startup configuration
│   │   └── appsettings.json # Configuration
│   │
│   ├── Tukupedia.Core/      # Domain models & interfaces
│   ├── Tukupedia.Data/      # Database context & repositories
│   ├── Tukupedia.Service/   # Service layer
│   └── Tukupedia.sln        # Solution file
│
├── database/                # Database scripts
│   ├── migrations/          # DB migration files
│   └── seeders/             # Seed data scripts
│
├── docker-compose.yml       # Local dev environment
├── Dockerfile               # Backend container config
├── .dockerignore             # Docker ignore rules
├── .env.example              # Environment variables template
│
├── docs/                    # Documentation
│   ├── ARCHITECTURE.md      # System design & architecture
│   ├── SETUP.md             # Development setup guide
│   ├── API.md               # API documentation
│   └── DATABASE.md          # Database schema docs
│
└── .github/
    └── workflows/           # CI/CD pipelines
        └── ci.yml           # GitHub Actions workflow
```

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 18+ (for development)
- .NET 8 SDK (for development)
- PostgreSQL 15+ (via Docker)
- Redis (via Docker)

### Setup Development Environment

```bash
# 1. Clone repository
git clone https://github.com/Adityoexs/tukupedia.git
cd tukupedia

# 2. Copy environment file
cp .env.example .env

# 3. Start services (PostgreSQL, Redis)
docker-compose up -d

# 4. Setup backend
cd backend
dotnet restore
dotnet ef database update
cd ..

# 5. Setup frontend
cd frontend
npm install
npm run dev
cd ..

# Services akan tersedia di:
# Frontend: http://localhost:3000
# Backend GraphQL: http://localhost:5000/graphql
# Backend REST: http://localhost:5000/api
```

## 📚 Dokumentasi

- [ARCHITECTURE.md](docs/ARCHITECTURE.md) - Arsitektur sistem & design decisions
- [SETUP.md](docs/SETUP.md) - Panduan setup development environment
- [DATABASE.md](docs/DATABASE.md) - Schema database & relasi
- [API.md](docs/API.md) - GraphQL & REST API documentation

## 🎯 Fitur Utama (MVP)

### User Features
- ✅ Katalog produk dengan kategori hierarki
- ✅ Pencarian & filter produk
- ✅ Keranjang & wishlist
- ✅ Checkout & berbagai metode pembayaran
- ✅ Tracking pesanan
- ✅ Rating & review produk
- ✅ Program loyalti & poin

### Merchant Features
- ✅ Buka toko (user menjadi merchant)
- ✅ Kelola produk & stok
- ✅ Kelola pesanan & pengiriman
- ✅ Melihat laporan penjualan

### Admin Features
- ✅ Dashboard analytics
- ✅ Manajemen kategori & produk
- ✅ Manajemen promo & voucher
- ✅ Moderasi review

### Progressive Web App
- ✅ Installable app
- ✅ Service Worker & offline support
- ✅ Push notifications
- ✅ Background sync
- ✅ Responsive design

## 🔄 Development Workflow

1. Create feature branch: `git checkout -b feature/your-feature`
2. Commit changes: `git commit -m "feat: your feature"`
3. Push to branch: `git push origin feature/your-feature`
4. Create Pull Request ke `main`

## 📋 Commit Convention

```
feat: add new feature
fix: fix bug
docs: update documentation
style: code style changes
refactor: refactor code
test: add tests
chore: maintenance tasks
```

## 🤝 Contributing

Kontribusi sangat diterima! Silakan buka issue atau PR untuk saran & perbaikan.

## 📄 License

MIT License - lihat [LICENSE](LICENSE) file untuk detail.

## 📞 Support

Untuk bantuan, silakan buka issue di repository ini atau hubungi tim development.

---

**Built with ❤️ by Tukupedia Team**
