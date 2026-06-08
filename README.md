# ห้องเรียนออนไลน์ครูโอม 🎓
**Aomsensei Classroom** — เว็บไซต์ห้องเรียนออนไลน์ฟรีสำหรับการศึกษา

---

## 📁 โครงสร้างไฟล์
```
aomsensei-classroom/
├── index.html        ← หน้าหลัก (สาธารณะ)
├── student.html      ← พอร์ทัลนักเรียน (ล็อกอินด้วยรหัสห้อง)
├── admin.html        ← แผงควบคุมครู (ล็อกอินด้วย Email/Password)
├── quiz.html         ← หน้าทำข้อสอบออนไลน์
├── js/
│   └── config.js     ← ⚙️ ใส่ Firebase Config ที่นี่
└── README.md
```

---

## 🚀 ขั้นตอนการตั้งค่า (ทำครั้งเดียว ~15 นาที)

### ขั้นที่ 1: สร้าง Firebase Project

1. ไปที่ [https://console.firebase.google.com](https://console.firebase.google.com)
2. คลิก **"Add project"** → ตั้งชื่อ เช่น `aomsensei-classroom`
3. ปิด Google Analytics (ไม่จำเป็น) → **Create project**

### ขั้นที่ 2: ตั้งค่า Authentication

1. ในเมนูซ้าย เลือก **Build → Authentication**
2. คลิก **"Get started"**
3. แท็บ **Sign-in method** → เปิดใช้ **Email/Password**
4. แท็บ **Users** → คลิก **"Add user"**
   - Email: `kruaom@school.com` (หรืออีเมลตามต้องการ)
   - Password: ตั้งรหัสผ่านที่แข็งแกร่ง
   - นี่คือบัญชีครูสำหรับล็อกอิน Admin

### ขั้นที่ 3: ตั้งค่า Firestore Database

1. เมนูซ้าย เลือก **Build → Firestore Database**
2. คลิก **"Create database"**
3. เลือก **"Start in production mode"** ⚠️ (ไม่ใช่ test mode) → Next → Enable
4. **ตั้งค่ากฎความปลอดภัย** — คลิกแท็บ **Rules** → วางโค้ดนี้แทนที่โค้ดเดิมทั้งหมด:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ===== เนื้อหาการเรียน: ทุกคนอ่านได้, แก้ไขได้เฉพาะครู (authenticated) =====
    match /classrooms/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /announcements/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /worksheets/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /quizzes/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /videos/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }

    // ===== ผลสอบ: นักเรียนสร้างได้, อ่านได้, ครูแก้ไข/ลบได้ =====
    match /quiz_results/{id} {
      allow read: if true;
      allow create: if true;
      allow update, delete: if request.auth != null;
    }

    // ===== ระบบเก็บคะแนน: ครูเท่านั้น =====
    match /score_system/{id} {
      allow read, write: if request.auth != null;
    }

    // ===== แชท: นักเรียนส่งได้, อ่านได้, ครูลบได้ =====
    match /chats/{roomId}/messages/{msgId} {
      allow read: if true;
      allow create: if true;
      allow delete: if request.auth != null;
    }

    // ===== ข้อมูลนักเรียน: สร้างได้, ครูแก้ไขได้ =====
    match /students/{id} {
      allow read: if true;
      allow create: if true;
      allow update, delete: if request.auth != null;
    }

    // ===== ตั้งค่าเว็บไซต์ (โลโก้/โปรไฟล์): ทุกคนอ่านได้, ครูเขียนได้ =====
    match /settings/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

5. คลิก **Publish**

> **⚠️ หมายเหตุ:** Production mode ปลอดภัยกว่า Test mode มาก เพราะ Test mode เปิดให้ใครก็ได้อ่าน-เขียนฐานข้อมูลได้ทั้งหมด ซึ่งไม่ปลอดภัยสำหรับข้อมูลนักเรียน

### ขั้นที่ 4: คัดลอก Firebase Config

1. เมนูซ้าย → คลิกไอคอน **⚙️ Project settings**
2. เลื่อนลงมาที่ **Your apps** → คลิกไอคอน **</>** (Web)
3. ตั้งชื่อ app เช่น `classroom-web` → **Register app**
4. คัดลอก `firebaseConfig` object ที่แสดงขึ้นมา

### ขั้นที่ 5: ใส่ Config ในไฟล์

เปิดไฟล์ `js/config.js` แล้วแทนที่ค่า:

```javascript
export const FIREBASE_CONFIG = {
  apiKey: "AIzaSy...",           // ← ใส่ค่าจาก Firebase
  authDomain: "aomsensei-classroom.firebaseapp.com",
  projectId: "aomsensei-classroom",
  storageBucket: "aomsensei-classroom.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc..."
};
```

### ขั้นที่ 6: อัพโหลดขึ้น GitHub Pages

1. สร้างบัญชี [GitHub](https://github.com) (ถ้ายังไม่มี)
2. คลิก **"New repository"** → ตั้งชื่อ: `aomsensei-classroom`
3. อัพโหลดไฟล์ทั้งหมดในโฟลเดอร์นี้ (ลาก & วางหรือใช้ GitHub Desktop)
4. ไปที่ **Settings → Pages**
5. Source: **Deploy from a branch** → Branch: `main` → Folder: `/(root)` → **Save**
6. รอ 2-3 นาที เว็บจะพร้อมใช้ที่: `https://[ชื่อ-github].github.io/aomsensei-classroom`

---

## 🎮 วิธีใช้งาน

### ครู (Admin)
1. เข้า `[url]/admin.html`
2. ล็อกอินด้วย Email/Password ที่ตั้งไว้ใน Firebase Auth
3. **เพิ่มห้องเรียน** → ตั้งชื่อและรหัสห้อง (เช่น `M301`)
4. เพิ่มใบงาน / ข้อสอบ / วิดีโอ / ประกาศได้ทันที

### นักเรียน
1. เข้า `[url]/index.html` หรือ `[url]/student.html`
2. กรอกรหัสห้องเรียนที่ครูให้ + ชื่อตัวเอง
3. ดูใบงาน ทำข้อสอบ ดูวิดีโอ ได้เลย

---

## 📊 ฟีเจอร์ทั้งหมด

| ฟีเจอร์ | ครู | นักเรียน |
|---------|-----|---------|
| จัดการห้องเรียน | ✅ เพิ่ม/ลบ/แก้ไข | ✅ เข้าด้วยรหัส |
| ประกาศ | ✅ CRUD + ปักหมุด | ✅ อ่าน |
| ใบงาน | ✅ CRUD (PDF/HTML/Link) | ✅ ดาวน์โหลด |
| ข้อสอบ | ✅ สร้างคำถาม/เฉลย | ✅ ทำ + ดูคะแนน |
| วิดีโอ YouTube | ✅ CRUD | ✅ ดูได้ทันที |
| สถิติ | ✅ Dashboard + กราฟ | ✅ ผลคะแนนตนเอง |
| รายชื่อนักเรียน | ✅ ดูและลบ | - |

---

## 🔗 ลิงก์ไฟล์ HTML เดิม

นำ URL ของไฟล์ HTML ที่ Claude สร้างไว้ก่อนหน้า มาใส่ในช่อง **URL/ลิงก์ไฟล์** เมื่อเพิ่มใบงาน โดยเลือกประเภทเป็น **HTML**

---

## 💡 เคล็ดลับ

- **รหัสห้อง**: ใช้ตัวพิมพ์ใหญ่ เช่น `M301`, `ENG6-1`
- **ไฟล์ PDF**: อัพโหลดที่ Google Drive แล้วเปลี่ยน link เป็น direct download
- **YouTube**: วางลิงก์ปกติ ระบบจะแปลงเป็น embed อัตโนมัติ

