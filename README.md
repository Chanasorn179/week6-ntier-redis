# TaskBoard — N-Tier Architecture with Redis Caching + Nginx Load Balancing

> **ENGSE207 Software Architecture — Term Project Week 6**  
> มหาวิทยาลัยเทคโนโลยีราชมงคลล้านนา | นาย ชนสรณ์ บุตรถา 67543210025-2

---

## ภาพรวมโปรเจกต์

โปรเจกต์นี้เป็นการต่อยอด N-Tier Architecture จาก Week 6 เดิม โดยเพิ่ม **Redis Caching Layer** และ **Nginx Load Balancing** เพื่อให้ระบบรองรับ Traffic สูงได้ดีขึ้น มี High Availability และ Horizontal Scaling

---

## สถาปัตยกรรมระบบ

```
Browser (Client)
      │
   Port 80
      ▼
┌─────────────────────────────────────┐
│  TIER 1: Nginx (Load Balancer)      │
│  • Serve Static Frontend            │
│  • Round-Robin /api/* → App ×3      │
└────────┬──────────┬──────────┬──────┘
         ▼          ▼          ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐
    │ TIER 2  │ │ TIER 2  │ │ TIER 2  │
    │  App#1  │ │  App#2  │ │  App#3  │
    │Node.js  │ │Node.js  │ │Node.js  │
    └────┬────┘ └────┬────┘ └────┬────┘
         └───────────┼───────────┘
            ┌────────┴────────┐
            ▼                 ▼
    ┌──────────────┐  ┌──────────────────┐
    │   TIER 3a    │  │    TIER 3b       │
    │    Redis     │  │   PostgreSQL     │
    │  (Cache)     │  │  (Persistent DB) │
    │  TTL: 60s    │  │                  │
    └──────────────┘  └──────────────────┘
```

| Tier | Container | บทบาท |
|------|-----------|-------|
| Tier 1 | Nginx | Load Balancer + Serve Static Files |
| Tier 2 | Node.js ×3 | Backend API (Scalable) |
| Tier 3a | Redis | In-Memory Cache (Cache-Aside Pattern) |
| Tier 3b | PostgreSQL | Persistent Database |

---

## โครงสร้างโปรเจกต์

```
week6-ntier-redis/
├── api/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   └── src/
│       ├── config/
│       │   ├── database.js       # PostgreSQL connection pool
│       │   └── redis.js          # Redis client + Cache utilities
│       ├── controllers/
│       │   └── taskController.js
│       ├── services/
│       │   └── taskService.js    # Business Logic + Cache-Aside
│       ├── repositories/
│       │   └── taskRepository.js # DB Queries
│       ├── routes/
│       │   └── taskRoutes.js
│       └── middleware/
│           └── logger.js
├── nginx/
│   ├── nginx.conf                # Load Balancer config (Round-Robin)
│   └── conf.d/
│       └── default.conf
├── frontend/
│   ├── index.html                # Task Board UI
│   ├── css/style.css
│   └── js/app.js
├── database/
│   └── init.sql                  # DB Schema + Seed Data
├── docker-compose.yml
├── .env
└── .gitignore
```

---

## Tech Stack

| ส่วนประกอบ | เทคโนโลยี |
|-----------|-----------|
| Backend API | Node.js + Express |
| Cache | Redis 7 |
| Database | PostgreSQL 15 |
| Load Balancer | Nginx (Round-Robin) |
| Container | Docker + Docker Compose |
| Frontend | HTML / CSS / Vanilla JS |

---

## วิธีรันโปรเจกต์

### ข้อกำหนดเบื้องต้น

- Docker Desktop / Docker Engine
- Docker Compose v2+

### 1. Clone โปรเจกต์

```bash
git clone https://github.com/Chanasorn179/week6-ntier-redis.git
cd week6-ntier-redis
```

### 2. ตั้งค่า Environment

```bash
cp .env.example .env
# แก้ไขค่าตามต้องการ (หรือใช้ค่า default ได้เลย)
```

### 3. รัน 3 App Instances พร้อม Load Balancing

```bash
docker compose up --scale app=3 --build -d
```

### 4. ตรวจสอบสถานะ

```bash
docker compose ps
```

### 5. เปิด Browser

```
http://localhost
```

---

## API Endpoints

| Method | Endpoint | รายละเอียด | Cache |
|--------|----------|-----------|-------|
| `GET` | `/api/health` | Health check + Instance ID + Cache Stats | ❌ |
| `GET` | `/api/tasks` | ดึง Task ทั้งหมด | ✅ TTL 60s |
| `GET` | `/api/tasks/:id` | ดึง Task ตาม ID | ❌ |
| `POST` | `/api/tasks` | สร้าง Task ใหม่ (invalidates cache) | ❌ |
| `PUT` | `/api/tasks/:id` | อัปเดต Task (invalidates cache) | ❌ |
| `DELETE` | `/api/tasks/:id` | ลบ Task (invalidates cache) | ❌ |

---

## การทดสอบ

### ทดสอบ Load Balancing (Round-Robin)

```bash
for i in $(seq 1 6); do
    RESPONSE=$(curl -s http://localhost/api/health | grep -o '"instanceId":"[^"]*"')
    echo "Request $i: $RESPONSE"
    sleep 0.5
done
```

ผลลัพธ์ที่คาดหวัง: `instanceId` สลับกันทุก request

### ทดสอบ Redis Cache (HIT/MISS)

```bash
# Request 1 → CACHE MISS (ดึงจาก DB)
curl -s http://localhost/api/tasks | python3 -m json.tool | head -5

# Request 2 → CACHE HIT (ได้จาก Redis)
curl -s http://localhost/api/tasks | python3 -m json.tool | head -5

# ดู Cache Stats
curl -s http://localhost/api/health | python3 -m json.tool
```

### Load Test (100 requests)

```bash
START=$(date +%s%N)
for i in $(seq 1 100); do
    curl -s -o /dev/null http://localhost/api/tasks
done
END=$(date +%s%N)
echo "100 requests: $(( (END - START) / 1000000 ))ms"
```

### ดู Redis Keys

```bash
docker exec taskboard-redis redis-cli KEYS "tasks:*"
docker exec taskboard-redis redis-cli INFO stats | grep -E "keyspace|hit|miss"
```

---

## Redis Cache Strategy

โปรเจกต์นี้ใช้ **Cache-Aside Pattern (Lazy Loading)**:

```
1. App Request มา
      │
      ▼
   Check Redis
      │
   ┌──┴──┐
   │     │
  HIT   MISS
   │     │
   │     ▼
   │  Query PostgreSQL
   │     │
   │     ▼
   │  Store in Redis (TTL 60s)
   │     │
   └──►  Return Data
```

- **Cache HIT**: ~2–5ms (ได้จาก Redis)
- **Cache MISS**: ~50ms (ดึงจาก PostgreSQL แล้วเก็บ Cache)
- **Cache Invalidation**: เมื่อมีการ POST / PUT / DELETE จะลบ Cache ทันที

---

## เปรียบเทียบกับ Week 6 เดิม

| Feature | Week 6 เดิม | Term Project Week 6 |
|---------|------------|---------------------|
| App Instances | 1 | 3 (scalable) |
| Cache | ❌ | ✅ Redis TTL-based |
| Load Balancing | ❌ | ✅ Nginx Round-Robin |
| Fault Tolerance | ❌ | ✅ ยังมี instance อื่นทำงาน |
| Horizontal Scaling | ❌ | ✅ `--scale app=N` |
| Performance | ★★☆☆☆ | ★★★★☆ |
| Availability | ★★☆☆☆ | ★★★★☆ |

---

## Challenges (ทำต่อเองได้)

| ระดับ | Challenge |
|:-----:|-----------|
| ⭐ | เพิ่ม Cache สำหรับ `GET /tasks/:id` |
| ⭐⭐ | เพิ่ม `X-Cache-Status: HIT/MISS` header ใน Response |
| ⭐⭐⭐ | เปลี่ยน Load Balancing เป็น Least Connections (`least_conn`) |

---

## Deliverables Checklist

- [ ] `docker-compose.yml` รัน `--scale app=3` ได้
- [ ] Redis Caching ทำงาน (เห็น HIT/MISS ใน logs)
- [ ] Load Balancing ทำงาน (Instance ID สลับกัน)
- [ ] Frontend แสดง Task Board + Instance Info
- [ ] Health Check endpoint แสดง Cache Stats
- [ ] Git commit พร้อม message อธิบาย

---

## วัตถุประสงค์การเรียนรู้ (CLO)

- อธิบายความแตกต่างระหว่าง **Tier** (Physical) กับ **Layer** (Logical) ได้
- ติดตั้งและใช้งาน **Redis** เป็น Caching Layer ใน Docker ได้
- ตั้งค่า **Nginx** เป็น Load Balancer แบบ Round-Robin สำหรับ Multiple Instances ได้
- ใช้ `docker compose up --scale` เพื่อ **Horizontal Scaling** ได้
- ทดสอบ **Cache Hit/Miss** และวัดผลกระทบต่อ Performance ได้

---

*ENGSE207 Software Architecture — Term Project Week 6*