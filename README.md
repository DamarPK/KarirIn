# KarirIn — Super Aggregator Platform Edukasi & Karier Indonesia

> Platform agregator terbesar untuk pendidikan, skill, sertifikasi, dan karier di Indonesia.

---

## 📁 Struktur Folder (Laravel)

```
karirin/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/
│   │   │   │   ├── GoogleAuthController.php
│   │   │   │   └── LoginController.php
│   │   │   ├── SearchController.php
│   │   │   ├── PlatformController.php
│   │   │   ├── RoadmapController.php
│   │   │   ├── JobController.php
│   │   │   ├── BookmarkController.php
│   │   │   ├── UserDashboardController.php
│   │   │   ├── CommunityController.php
│   │   │   └── ScholarshipController.php
│   │   ├── Middleware/
│   │   │   ├── PremiumAccess.php
│   │   │   └── TrackAnalytics.php
│   │   └── Requests/
│   │       └── SearchRequest.php
│   ├── Models/
│   │   ├── User.php
│   │   ├── Platform.php
│   │   ├── Roadmap.php
│   │   ├── RoadmapStep.php
│   │   ├── Job.php
│   │   ├── Course.php
│   │   ├── Bookmark.php
│   │   ├── SkillTracker.php
│   │   ├── Community.php
│   │   ├── Scholarship.php
│   │   └── SearchHistory.php
│   └── Services/
│       ├── AIRecommendationService.php
│       ├── SearchAggregatorService.php
│       └── AffiliateTrackingService.php
├── database/
│   ├── migrations/
│   │   ├── create_users_table.php
│   │   ├── create_platforms_table.php
│   │   ├── create_courses_table.php
│   │   ├── create_jobs_table.php
│   │   ├── create_roadmaps_table.php
│   │   ├── create_bookmarks_table.php
│   │   ├── create_skill_trackers_table.php
│   │   └── create_search_histories_table.php
│   └── seeders/
│       ├── PlatformSeeder.php
│       └── RoadmapSeeder.php
├── resources/views/
│   ├── layouts/app.blade.php
│   ├── pages/
│   │   ├── landing.blade.php
│   │   ├── dashboard.blade.php
│   │   ├── search.blade.php
│   │   ├── roadmap/index.blade.php
│   │   ├── roadmap/detail.blade.php
│   │   ├── jobs/index.blade.php
│   │   ├── community/index.blade.php
│   │   └── scholarship/index.blade.php
│   └── components/
│       ├── platform-card.blade.php
│       ├── job-card.blade.php
│       └── roadmap-card.blade.php
├── routes/
│   ├── web.php
│   └── api.php
├── public/
│   ├── index.html          ← Landing page (static HTML)
│   ├── dashboard.html      ← Dashboard
│   └── roadmap.html        ← Roadmap detail
└── .env
```

---

## 🗄️ Database Schema (MySQL)

### users
```sql
CREATE TABLE users (
  id            BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  name          VARCHAR(255) NOT NULL,
  email         VARCHAR(255) UNIQUE NOT NULL,
  google_id     VARCHAR(255) NULL,
  avatar        VARCHAR(500) NULL,
  plan          ENUM('free','premium') DEFAULT 'free',
  career_target VARCHAR(255) NULL,
  bio           TEXT NULL,
  created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### platforms
```sql
CREATE TABLE platforms (
  id          BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  name        VARCHAR(255) NOT NULL,
  slug        VARCHAR(255) UNIQUE NOT NULL,
  url         VARCHAR(500) NOT NULL,
  description TEXT,
  category    ENUM('kursus','bootcamp','loker','sertifikasi','kampus','komunitas'),
  pricing     ENUM('free','paid','freemium'),
  logo_emoji  VARCHAR(10),
  logo_color  VARCHAR(10),
  is_verified BOOLEAN DEFAULT TRUE,
  affiliate_url VARCHAR(500) NULL,
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### jobs (cached dari API eksternal)
```sql
CREATE TABLE jobs (
  id           BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  title        VARCHAR(255) NOT NULL,
  company      VARCHAR(255) NOT NULL,
  location     VARCHAR(255),
  salary_min   INT UNSIGNED NULL,
  salary_max   INT UNSIGNED NULL,
  is_remote    BOOLEAN DEFAULT FALSE,
  tags         JSON,
  source       VARCHAR(100),
  source_url   VARCHAR(500),
  source_id    VARCHAR(255),
  posted_at    TIMESTAMP NULL,
  expires_at   TIMESTAMP NULL,
  created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_title (title),
  INDEX idx_posted (posted_at)
);
```

### bookmarks
```sql
CREATE TABLE bookmarks (
  id           BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  user_id      BIGINT UNSIGNED NOT NULL,
  item_type    ENUM('course','job','platform','roadmap','scholarship'),
  item_id      BIGINT UNSIGNED NULL,
  item_url     VARCHAR(500) NULL,
  item_title   VARCHAR(255),
  item_meta    JSON,
  created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  UNIQUE KEY unique_bookmark (user_id, item_type, item_id)
);
```

### skill_trackers
```sql
CREATE TABLE skill_trackers (
  id           BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  user_id      BIGINT UNSIGNED NOT NULL,
  skill_name   VARCHAR(255) NOT NULL,
  progress     TINYINT UNSIGNED DEFAULT 0,   -- 0-100
  source_name  VARCHAR(255) NULL,
  notes        TEXT NULL,
  updated_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### search_histories
```sql
CREATE TABLE search_histories (
  id         BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  user_id    BIGINT UNSIGNED NULL,
  query      VARCHAR(500) NOT NULL,
  filters    JSON NULL,
  result_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user (user_id),
  INDEX idx_query (query(100))
);
```

---

## 🔌 API Structure (Laravel Routes)

### Public API
```
GET  /api/search?q=&category=&pricing=&is_remote=
GET  /api/platforms?category=
GET  /api/jobs?query=&location=&remote=&page=
GET  /api/roadmaps
GET  /api/roadmaps/{slug}
GET  /api/scholarships
GET  /api/community/forums
```

### Authenticated API
```
POST /api/auth/google
POST /api/auth/logout

GET  /api/user/dashboard
GET  /api/user/bookmarks
POST /api/user/bookmarks
DELETE /api/user/bookmarks/{id}

GET  /api/user/skills
PUT  /api/user/skills/{id}
POST /api/user/skills

GET  /api/user/search-history
GET  /api/ai/recommendations  ?goal=&skills=
```

---

## 🤖 AI Recommendation Service

```php
// app/Services/AIRecommendationService.php

class AIRecommendationService {
    public function getRecommendations(string $userGoal, array $currentSkills): array {
        // 1. Kirim ke OpenAI / Claude API
        $prompt = "User goal: $userGoal. Current skills: " . implode(', ', $currentSkills);
        $prompt .= "\n\nReturn JSON: {roadmap: [], courses: [], jobs: [], certifications: []}";

        $response = Http::withToken(config('services.ai.key'))
            ->post('https://api.openai.com/v1/chat/completions', [
                'model' => 'gpt-4o-mini',
                'messages' => [['role' => 'user', 'content' => $prompt]],
                'response_format' => ['type' => 'json_object'],
            ]);

        return json_decode($response['choices'][0]['message']['content'], true);
    }
}
```

---

## 💰 Monetisasi

### 1. Affiliate System
- Setiap klik ke platform partner (Dicoding, Coursera, RevoU, dll.) dilacak via affiliate link
- Implementasi: `AffiliateTrackingService.php` + UTM parameters
- Estimasi: Rp 50rb–500rb per konversi

### 2. Premium Membership (Rp 49.000/bulan)
- Fitur gratis: Search, bookmark 10 item, roadmap dasar
- Fitur premium: AI recommendations, unlimited bookmark, skill tracker penuh, akses mentor, analytics karier

### 3. Job Posting (Rp 500rb–2jt/listing)
- Perusahaan pasang loker langsung di KarirIn
- Featured listing: Muncul di atas hasil pencarian

### 4. Partnership Kampus & Bootcamp
- Sponsored placement di roadmap dan search results
- Paket: Rp 5–30 juta/bulan

### 5. Display Advertising
- Google AdSense integration untuk halaman search
- Native ads (label "Sponsored") di platform card

---

## 🚀 Deployment Guide

### Requirements
- PHP 8.2+
- Laravel 11
- MySQL 8.0+
- Redis (untuk caching & queue)
- Node.js 18+ (untuk asset build)

### 1. Clone & Setup
```bash
git clone https://github.com/yourorg/karirin.git
cd karirin
composer install
npm install && npm run build
cp .env.example .env
php artisan key:generate
```

### 2. Environment Variables (.env)
```env
APP_NAME=KarirIn
APP_URL=https://karirin.id
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=karirin_db
DB_USERNAME=karirin_user
DB_PASSWORD=strongpassword

GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx
GOOGLE_REDIRECT_URI=https://karirin.id/auth/google/callback

OPENAI_API_KEY=sk-xxx
REDIS_HOST=127.0.0.1

AFFILIATE_DICODING=affiliate_code_here
AFFILIATE_COURSERA=affiliate_code_here
```

### 3. Database
```bash
php artisan migrate
php artisan db:seed --class=PlatformSeeder
php artisan db:seed --class=RoadmapSeeder
```

### 4. Production (Vercel / Railway / VPS)

**Vercel (Frontend Static):**
```bash
vercel --prod
```

**Railway (Laravel Backend):**
```bash
railway up
railway run php artisan migrate
```

**VPS (Ubuntu 22.04 + Nginx):**
```nginx
server {
    listen 80;
    server_name karirin.id;
    root /var/www/karirin/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 5. Caching (Redis)
```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
# Job fetching setiap 30 menit
php artisan schedule:work
```

---

## 📊 Analytics Dashboard

Integrasikan dengan:
- **Google Analytics 4** — traffic & user behavior
- **Mixpanel** — event tracking (search, bookmark, click)
- **Custom Admin Panel** — `laravel-nova` atau `filament`

---

## 🌐 SEO Strategy

- Server-side rendering untuk semua halaman utama
- Sitemap otomatis: `/sitemap.xml`
- Schema markup: `JobPosting`, `Course`, `Organization`
- Meta tags dinamis per roadmap & platform
- Target keyword: "kursus online Indonesia", "bootcamp programming", "roadmap data scientist"

---

*KarirIn © 2025 — Made with ❤️ in Indonesia*
