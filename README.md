# ⚡ WEEK6 - N-Tier Architecture with Redis

โปรเจกต์นี้เป็นการพัฒนาแอปพลิเคชันโดยใช้แนวคิด **N-Tier Architecture**  
ร่วมกับ **Redis** เพื่อเพิ่มประสิทธิภาพของระบบในด้านความเร็วและการจัดการข้อมูล

---

## 📌 แนวคิดของระบบ

ระบบนี้ออกแบบโดยใช้ **N-Tier Architecture** ซึ่งแบ่งโครงสร้างออกเป็นหลายชั้น ได้แก่

- 🌐 Presentation Layer (Controller / API)
- ⚙️ Business Logic Layer (Service)
- 🗄️ Data Access Layer (Repository / Database)
- ⚡ Cache Layer (Redis)

โดย Redis ถูกใช้เป็น **In-memory cache** ซึ่งช่วยให้เข้าถึงข้อมูลได้รวดเร็วมาก  
และลดภาระของ Database ลงอย่างมีนัยสำคัญ  [oai_citation:0‡GitHub](https://github.com/redis/redis?utm_source=chatgpt.com)  

---

## 🧱 Architecture Overview

โครงสร้างของระบบประกอบด้วย:

- Backend API (Application Server)
- Database (เช่น MySQL / PostgreSQL)
- Redis (Cache Layer)
- Docker (สำหรับจัดการหลาย service)

### 🔄 Flow การทำงาน

1. Client ส่ง request มาที่ API
2. Server ตรวจสอบข้อมูลใน Redis
3. ถ้ามีข้อมูล → ส่งกลับทันที ⚡
4. ถ้าไม่มี → ไป query Database
5. เก็บข้อมูลลง Redis เพื่อใช้ครั้งถัดไป

---

## ⚙️ Features

- ⚡ ใช้ Redis สำหรับ Caching
- 🚀 เพิ่มความเร็วในการตอบสนองของระบบ
- 🧠 ลดการ query Database ซ้ำ
- 🏗️ โครงสร้างแบบ N-Tier ชัดเจน
- 🐳 รองรับ Docker (Multi-container)

---

## 👥 สมาชิกในทีม

- นายเบญจครายุทธ น้อยอุบล  
- นายชนะสรณ์ บุตรถา  
- นายธาวัน ทิพคุณ  
- นายอดิ โรจน์ กุหลั่น  

---

## 🚀 วิธีใช้งานโปรเจกต์

### 1. Clone Repository
```bash
git clone https://github.com/Chanasorn179/week6-ntier-redis.git
```

### 2. เข้าไปที่โฟลเดอร์โปรเจกต์
```bash
cd week6-ntier-redis
```

---

## ▶️ วิธีรันระบบ

### 🐳 ใช้ Docker (แนะนำ)
```bash
docker-compose up --build
```

---

### 💻 รันแบบปกติ
```bash
# ตัวอย่าง (ปรับตามภาษา/เฟรมเวิร์ก)
npm install
npm start
```

---

## 🛠️ เทคโนโลยีที่ใช้

- Redis ⚡
- Docker 🐳
- N-Tier Architecture
- Backend Framework (Node.js / Java / etc.)
- Database (MySQL / PostgreSQL)

---

## 📦 จุดเด่นของโปรเจกต์

- เพิ่ม Performance ด้วย Redis Cache
- ลดภาระของ Database
- แยก Layer ชัดเจน (Maintain ง่าย)
- รองรับการขยายระบบ (Scalable)
- ใช้งานร่วมกับ Docker ได้สะดวก

---

## 📖 สิ่งที่ได้เรียนรู้

- การใช้ Redis สำหรับ Cache
- การออกแบบระบบแบบ N-Tier
- การเชื่อมต่อ Redis กับ Backend
- การจัดการหลาย Service ด้วย Docker

---

## 📎 Repository
https://github.com/Chanasorn179/week6-ntier-redis