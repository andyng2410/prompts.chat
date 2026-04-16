# Deploy prompts.chat len Dokploy (Server: nocode.autos)

Huong dan tung buoc deploy du an **prompts.chat** len Dokploy voi subdomain tren server `nocode.autos`.

---

## Muc luc

1. [Yeu cau](#1-yeu-cau)
2. [Tao PostgreSQL Database tren Dokploy](#2-tao-postgresql-database-tren-dokploy)
3. [Tao Application tren Dokploy](#3-tao-application-tren-dokploy)
4. [Cau hinh Environment Variables](#4-cau-hinh-environment-variables)
5. [Cau hinh Dockerfile Path](#5-cau-hinh-dockerfile-path)
6. [Dat subdomain](#6-dat-subdomain)
7. [Deploy](#7-deploy)
8. [Kiem tra va xac nhan](#8-kiem-tra-va-xac-nhan)
9. [Seed du lieu (tuy chon)](#9-seed-du-lieu-tuy-chon)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Yeu cau

- Server da cai dat **Dokploy** (https://dokploy.com)
- Domain `nocode.autos` da tro DNS ve server (A record hoac wildcard `*.nocode.autos`)
- Truy cap Dokploy dashboard tai `https://dokploy.nocode.autos` (hoac IP server)

### DNS Setup

Truoc khi bat dau, dam bao ban da tao **DNS A Record** cho subdomain mong muon:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | `prompts` | `<IP_SERVER>` | 300 |
| A | `*.nocode.autos` (wildcard) | `<IP_SERVER>` | 300 |

> Neu da co wildcard DNS (`*.nocode.autos`) thi khong can tao rieng.

---

## 2. Tao PostgreSQL Database tren Dokploy

### Buoc 2.1: Tao Database Service

1. Vao **Dokploy Dashboard** > **Create Service** > **Database**
2. Chon **PostgreSQL**
3. Dat ten: `prompts-chat-db`
4. Cau hinh:
   - **Database Name:** `prompts_chat`
   - **Username:** `prompts_user`
   - **Password:** Dat mat khau manh (vi du: `Str0ng!P@ssw0rd2024`)
   - **Port:** `5432` (mac dinh)

### Buoc 2.2: Lay Connection String

Sau khi database duoc tao, lay connection string co dang:

```
postgresql://prompts_user:Str0ng!P@ssw0rd2024@prompts-chat-db:5432/prompts_chat?schema=public
```

> **Luu y:** Trong Dokploy, cac service co the giao tiep qua ten service (internal DNS).
> Neu database va app cung nam trong mot Dokploy project, dung ten service thay vi `localhost`.

---

## 3. Tao Application tren Dokploy

### Buoc 3.1: Tao Project

1. Vao **Dokploy Dashboard** > **Projects** > **Create Project**
2. Dat ten: `prompts-chat`

### Buoc 3.2: Tao Application

1. Trong project `prompts-chat` > **Create Service** > **Application**
2. Dat ten: `prompts-chat-app`
3. Chon **Source**: tuy chon mot trong cac cach sau:

#### Cach A: Tu GitHub Repository

1. Chon **GitHub**
2. Ket noi voi repository: `andyng2410/prompts.chat`
3. Chon branch: `main` (hoac branch mong muon)

#### Cach B: Tu Docker Image (tu build truoc)

Neu ban muon build image truoc roi push len registry:

```bash
# Build image tren may local
docker build -f docker/Dockerfile -t prompts-chat:latest .

# Tag va push len registry (Docker Hub hoac private registry)
docker tag prompts-chat:latest your-registry/prompts-chat:latest
docker push your-registry/prompts-chat:latest
```

Sau do trong Dokploy chon **Docker Image** va nhap image URL.

#### Cach C: Tu Git Repository (URL)

1. Chon **Git** > nhap URL repository
2. Nhap branch name

---

## 4. Cau hinh Environment Variables

Vao **Application** > **Environment** > them cac bien moi truong sau:

### Bat buoc

```env
# Database - dung connection string tu buoc 2
DATABASE_URL=postgresql://prompts_user:Str0ng!P@ssw0rd2024@prompts-chat-db:5432/prompts_chat?schema=public

# NextAuth
NEXTAUTH_URL=https://prompts.nocode.autos
NEXTAUTH_SECRET=<tao-bang-openssl-rand-base64-32>

# Cron Secret
CRON_SECRET=<random-secret-key>
```

> **Tao NEXTAUTH_SECRET:**
> ```bash
> openssl rand -base64 32
> ```

### Tuy chon

```env
# OAuth - GitHub (neu muon dang nhap bang GitHub)
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret

# OAuth - Google
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret

# AI Features (can OPENAI_API_KEY)
OPENAI_API_KEY=sk-xxx
OPENAI_BASE_URL=https://api.openai.com/v1
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_GENERATIVE_MODEL=gpt-4o-mini

# Sentry (monitoring)
SENTRY_AUTH_TOKEN=your_sentry_token

# Logging
LOG_LEVEL=info
```

---

## 5. Cau hinh Dockerfile Path

Vi Dockerfile nam trong thu muc `docker/`, can chi dinh dung duong dan:

1. Vao **Application** > **General** (hoac **Build**)
2. Cau hinh:
   - **Build Type:** `Dockerfile`
   - **Dockerfile Path:** `docker/Dockerfile`
   - **Build Context:** `.` (root cua repository)

---

## 6. Dat Subdomain

Day la buoc quan trong de truy cap ung dung qua subdomain tren `nocode.autos`.

### Buoc 6.1: Cau hinh Domain trong Dokploy

1. Vao **Application** > **Domains**
2. Click **Add Domain**
3. Nhap domain: `prompts.nocode.autos`
4. Cau hinh:
   - **Port:** `3000` (port cua Next.js)
   - **HTTPS:** Bat (Enable)
   - **Certificate:** Let's Encrypt (tu dong)

### Buoc 6.2: Vi du cac subdomain khac

Neu muon dung ten subdomain khac:

| Subdomain | Full Domain | Muc dich |
|-----------|-------------|----------|
| `prompts` | `prompts.nocode.autos` | Ung dung chinh |
| `ai-prompts` | `ai-prompts.nocode.autos` | Ten thay the |
| `chat` | `chat.nocode.autos` | Ten ngan gon |

### Buoc 6.3: Cau hinh Proxy (Advanced)

Trong tab **Advanced** cua domain, dam bao:

```
# Proxy settings
Proxy Port: 3000
Protocol: HTTP
```

Neu can cau hinh them Traefik labels (Advanced):

```yaml
# Traefik labels (Dokploy tu dong tao, chi can chinh neu can)
traefik.http.routers.prompts-chat.rule: Host(`prompts.nocode.autos`)
traefik.http.routers.prompts-chat.tls: true
traefik.http.routers.prompts-chat.tls.certresolver: letsencrypt
traefik.http.services.prompts-chat.loadbalancer.server.port: 3000
```

---

## 7. Deploy

### Buoc 7.1: Deploy ung dung

1. Vao **Application** > click **Deploy**
2. Dokploy se:
   - Clone repository
   - Build Docker image tu `docker/Dockerfile`
   - Chay container
   - `entrypoint.sh` se tu dong:
     - Doi database san sang
     - Chay Prisma migrations
     - Khoi dong Next.js server

### Buoc 7.2: Theo doi logs

1. Vao **Application** > **Logs** de xem tien trinh
2. Tim cac dong log thanh cong:

```
[prompts.chat] Waiting for database...
[prompts.chat] Running database migrations...
[prompts.chat] Migrations applied successfully.
[prompts.chat] Starting application on port 3000...
```

---

## 8. Kiem tra va xac nhan

### Health Check

```bash
curl https://prompts.nocode.autos/api/health
```

### Truy cap website

Mo trinh duyet va truy cap: **https://prompts.nocode.autos**

### Tao tai khoan Admin

Sau khi deploy thanh cong, dang ky tai khoan dau tien. Sau do nang cap len admin bang cach:

1. Truy cap vao container qua Dokploy **Terminal** (hoac SSH vao server)
2. Chay lenh:

```bash
# Vao container
docker exec -it <container_id> sh

# Chay script reset admin (neu co)
npx tsx prisma/reset-admin.ts
```

Hoac truc tiep cap nhat database:

```bash
# Ket noi vao PostgreSQL
docker exec -it <db_container_id> psql -U prompts_user -d prompts_chat

# Cap nhat role admin
UPDATE "User" SET role = 'ADMIN' WHERE email = 'your-email@example.com';
```

---

## 9. Seed du lieu (tuy chon)

Neu muon co du lieu mau:

```bash
# Vao container app
docker exec -it <container_id> sh

# Chay seed
npx prisma db seed
```

---

## 10. Troubleshooting

### Loi "Database not reachable"

- Kiem tra ten service database trong `DATABASE_URL` co dung khong
- Dam bao database va app cung nam trong mot Dokploy project (de co the dung internal DNS)
- Thu dung IP noi bo thay vi ten service

### Loi "Migration failed"

- Kiem tra `DATABASE_URL` co dung khong
- Dam bao database da duoc tao va user co quyen

### Loi SSL/HTTPS

- Kiem tra DNS A record da tro dung ve server
- Doi Let's Encrypt cap certificate (co the mat vai phut)
- Kiem tra port 80 va 443 da mo tren firewall

### Loi Build failed

- Kiem tra Dockerfile path la `docker/Dockerfile`
- Dam bao build context la `.` (root)
- Kiem tra logs build de xem loi cu the
- Du an yeu cau Node 24 - Dockerfile da cau hinh san

### Container bi restart lien tuc

- Kiem tra logs: co the database chua san sang
- Tang so lan retry trong `entrypoint.sh` neu can
- Kiem tra memory cua server (Next.js build can ~2GB RAM)

### Khong truy cap duoc website

- Kiem tra domain da duoc cau hinh dung trong Dokploy
- Kiem tra Traefik proxy dang chay
- Kiem tra `NEXTAUTH_URL` phai trung voi domain thuc te

---

## Tom tat cau hinh

| Muc | Gia tri |
|-----|---------|
| **App Name** | `prompts-chat-app` |
| **Database** | PostgreSQL (`prompts-chat-db`) |
| **Dockerfile** | `docker/Dockerfile` |
| **Build Context** | `.` |
| **Port** | `3000` |
| **Domain** | `prompts.nocode.autos` |
| **HTTPS** | Let's Encrypt (auto) |
| **Node Version** | 24 (trong Dockerfile) |
| **Output Mode** | Standalone (da cau hinh san) |
