# 🛡️ Running Scoreboard — ประกันติดโล่

ระบบตารางคะแนนการวิ่งสำหรับทีม โดยใช้ AI อ่านข้อมูลจากภาพ Screenshot ของแอป Strava, Garmin, Nike Run Club และอื่นๆ

---

## 📋 สารบัญ

1. [คุณสมบัติหลัก](#-คุณสมบัติหลัก)
2. [เตรียมข้อมูลใน Google Sheets](#-เตรียมข้อมูลใน-google-sheets)
3. [ตั้งค่า Google Apps Script](#-ตั้งค่า-google-apps-script)
4. [ติดตั้งและใช้งาน](#-ติดตั้งและใช้งาน)
5. [การใช้งานระบบ](#-การใช้งานระบบ)
6. [สูตรคำนวณคะแนน](#-สูตรคำนวณคะแนน)
7. [FAQ](#-faq)

---

## ⭐ คุณสมบัติหลัก

- **🏆 Scoreboard** — ตารางคะแนนแบบ Real-time แยกตามรายบุคคลและทีม พร้อม Podium Top 3
- **📤 Upload & AI Reader** — อัปโหลดรูป Screenshot จากแอปวิ่ง GPT-4o อ่านข้อมูลอัตโนมัติ
- **📊 Google Sheets Backend** — เก็บข้อมูลทั้งหมดใน Google Sheets สะดวกต่อการจัดการ
- **📅 ประวัติการวิ่ง** — ดูรายการวิ่งย้อนหลัง กรองตามสมาชิก พร้อมรูปภาพ
- **⚙️ Admin Panel** — จัดการทีม สมาชิก ป้องกันด้วยรหัสผ่าน `zxcvbn`
- **✅ ป้องกันการส่งซ้ำ** — ส่งได้วันละ 1 ครั้งต่อคน

---

## 📊 เตรียมข้อมูลใน Google Sheets

### ขั้นตอนที่ 1: สร้าง Google Sheet

1. ไปที่ [Google Sheets](https://sheets.google.com)
2. สร้าง Spreadsheet ใหม่ ตั้งชื่อว่า **"Running Scoreboard - ประกันติดโล่"**
3. สร้าง 3 Sheets ภายในไฟล์:

---

### Sheet 1: `Teams` (ข้อมูลทีม)

สร้าง Sheet ชื่อ **Teams** พร้อมหัวตารางและข้อมูล:

| id | name | emoji | color |
|----|------|-------|-------|
| 1 | ทีม อินทรี | 🦅 | #1a3a6b |
| 2 | ทีม เสือ | 🐯 | #c8922a |
| 3 | ทีม นาคา | 🐉 | #2a7a4b |
| 4 | ทีม สิงห์ | 🦁 | #8b2252 |
| 5 | ทีม หมาป่า | 🐺 | #4a4a8a |
| 6 | ทีม เหยี่ยว | 🦆 | #7a3a1a |
| 7 | ทีม มังกร | 🐲 | #1a6b6b |
| 8 | ทีม ม้า | 🐴 | #6b1a3a |

**คอลัมน์:**
- `id` = ตัวเลข unique (1, 2, 3...)
- `name` = ชื่อทีม
- `emoji` = อิโมจิสัญลักษณ์ทีม (copy-paste จากตาราง)
- `color` = สีของทีม (Hex code)

---

### Sheet 2: `Members` (ข้อมูลสมาชิก)

สร้าง Sheet ชื่อ **Members** พร้อมหัวตารางและข้อมูลตัวอย่าง 70 คน:

| id | name | teamId | avatar |
|----|------|--------|--------|
| 1 | สมชาย ใจดี | 1 | 🏃 |
| 2 | สุภา รักวิ่ง | 1 | 🏃‍♀️ |
| 3 | วิชัย เร็วมาก | 2 | 🏃 |
| 4 | นภา ไวดี | 2 | 🏃‍♀️ |
| 5 | ธนา คล่องแคล่ว | 3 | 🏃 |
| 6 | ปรียา ก้าวไกล | 3 | 🏃‍♀️ |
| 7 | กิตติ วิ่งเช้า | 4 | 🏃 |
| 8 | มาลี ชนะใจ | 4 | 🏃‍♀️ |
| 9 | อนุชา เต็มสิบ | 5 | 🏃 |
| 10 | รัตนา แชมป์ | 6 | 🏃‍♀️ |
| ... | ... | ... | ... |

**คอลัมน์:**
- `id` = รหัสสมาชิก (1, 2, 3... ถึง 70)
- `name` = ชื่อ-นามสกุล
- `teamId` = ระบุว่าอยู่ทีมไหน (อ้างอิงจาก Teams.id)
- `avatar` = อิโมจิ (ใช้ 🏃 สำหรับชาย, 🏃‍♀️ สำหรับหญิง)

**💡 เคล็ดลับ:** แบ่งสมาชิกเท่าๆ กันในแต่ละทีม (70 คน ÷ 8 ทีม ≈ 8-9 คน/ทีม)

---

### Sheet 3: `Submissions` (บันทึกการวิ่ง)

สร้าง Sheet ชื่อ **Submissions** พร้อมหัวตาราง:

| id | memberId | date | distance | pace | duration | score | imageUrl |
|----|----------|------|----------|------|----------|-------|----------|
| | | | | | | | |

**คอลัมน์:**
- `id` = รหัสรายการวิ่ง (เริ่ม 1, 2, 3...)
- `memberId` = อ้างอิงจาก Members.id
- `date` = วันที่วิ่ง (รูปแบบ: YYYY-MM-DD เช่น 2025-06-10)
- `distance` = ระยะทาง (กิโลเมตร, ทศนิยม เช่น 10.5)
- `pace` = ความเร็ว (นาที/กม., ทศนิยม เช่น 5.5 = 5:30/km)
- `duration` = เวลา (นาที, ทศนิยม เช่น 55.65)
- `score` = คะแนนที่คำนวณได้
- `imageUrl` = รูปภาพ Screenshot (เว้นว่างไว้ หรือใส่ null)

**ข้อมูล Mock สำหรับทดสอบ (10-20 แถว):**

```
1, 1, 2025-06-10, 10.5, 5.3, 55.65, 159, 
2, 2, 2025-06-10, 8.2, 5.8, 47.56, 122, 
3, 3, 2025-06-10, 12.0, 5.1, 61.2, 170, 
4, 4, 2025-06-10, 7.5, 6.2, 46.5, 103, 
5, 5, 2025-06-10, 15.0, 4.8, 72.0, 242, 
6, 6, 2025-06-10, 6.0, 6.5, 39.0, 85, 
7, 7, 2025-06-10, 11.2, 5.4, 60.48, 153, 
8, 8, 2025-06-10, 9.8, 5.6, 54.88, 128, 
9, 9, 2025-06-10, 13.5, 5.0, 67.5, 203, 
10, 10, 2025-06-10, 20.0, 4.5, 90.0, 340, 
11, 1, 2025-06-09, 8.0, 5.5, 44.0, 130, 
12, 3, 2025-06-09, 10.0, 5.2, 52.0, 190, 
13, 5, 2025-06-09, 12.5, 4.9, 61.25, 230, 
14, 7, 2025-06-09, 9.5, 5.5, 52.25, 145, 
```

**💡 สูตรคำนวณคะแนนใน Google Sheets:**
ใส่ที่คอลัมน์ G (score) แถวที่ 2:
```
=ROUND((D2*10) + ((7-E2)*50) + IF(F2>60, (F2-60)*0.5, 0), 0)
```
แล้วลาก formula ลงมาทุกแถว

---

## 🔧 ตั้งค่า Google Apps Script

### ขั้นตอนที่ 2: สร้าง Backend API

1. ใน Google Sheet ไปที่ **Extensions → Apps Script**
2. ลบโค้ดเริ่มต้นออก แล้ววางโค้ดนี้:

```javascript
// ========== GET DATA (สำหรับโหลดข้อมูลมาแสดง) ==========
function doGet(e) {
  const action = e.parameter.action;
  
  if (action === "getTeams") {
    return getTeams();
  } else if (action === "getMembers") {
    return getMembers();
  } else if (action === "getSubmissions") {
    return getSubmissions();
  }
  
  return ContentService.createTextOutput(JSON.stringify({error: "Invalid action"}))
    .setMimeType(ContentService.MimeType.JSON);
}

function getTeams() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Teams");
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  const rows = data.slice(1);
  
  const teams = rows.map(row => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = row[i]);
    return obj;
  }).filter(t => t.id); // กรองแถวว่าง
  
  return ContentService.createTextOutput(JSON.stringify(teams))
    .setMimeType(ContentService.MimeType.JSON);
}

function getMembers() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Members");
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  const rows = data.slice(1);
  
  const members = rows.map(row => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = row[i]);
    return obj;
  }).filter(m => m.id); // กรองแถวว่าง
  
  return ContentService.createTextOutput(JSON.stringify(members))
    .setMimeType(ContentService.MimeType.JSON);
}

function getSubmissions() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Submissions");
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  const rows = data.slice(1);
  
  const submissions = rows.map(row => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = row[i]);
    return obj;
  }).filter(s => s.id); // กรองแถวว่าง
  
  return ContentService.createTextOutput(JSON.stringify(submissions))
    .setMimeType(ContentService.MimeType.JSON);
}

// ========== POST DATA ==========
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const action = data.action;
    
    // เพิ่มทีมใหม่
    if (action === "addTeam") {
      const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Teams");
      const team = data.team;
      sheet.appendRow([team.id, team.name, team.emoji, team.color]);
      return ContentService.createTextOutput(JSON.stringify({ok: true}))
        .setMimeType(ContentService.MimeType.JSON);
    }
    
    // เพิ่มสมาชิกใหม่
    if (action === "addMember") {
      const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Members");
      const member = data.member;
      sheet.appendRow([member.id, member.name, member.teamId, member.avatar]);
      return ContentService.createTextOutput(JSON.stringify({ok: true}))
        .setMimeType(ContentService.MimeType.JSON);
    }
    
    // บันทึกการวิ่งใหม่
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Submissions");
    
    // ตรวจสอบว่าส่งไปวันนี้แล้วหรือยัง (1 ครั้ง/วัน)
    const today = new Date().toISOString().split('T')[0];
    const existingData = sheet.getDataRange().getValues();
    const memberIdCol = 1; // คอลัมน์ B (memberId)
    const dateCol = 2; // คอลัมน์ C (date)
    
    for (let i = 1; i < existingData.length; i++) {
      if (existingData[i][memberIdCol] == data.memberId && 
          existingData[i][dateCol] === today) {
        return ContentService.createTextOutput(JSON.stringify({
          ok: false,
          error: "Already submitted today"
        })).setMimeType(ContentService.MimeType.JSON);
      }
    }
    
    // หา ID ใหม่
    const lastRow = sheet.getLastRow();
    const newId = lastRow > 1 ? sheet.getRange(lastRow, 1).getValue() + 1 : 1;
    
    // เพิ่มข้อมูลใหม่
    sheet.appendRow([
      newId,
      data.memberId,
      data.date,
      data.distance,
      data.pace,
      data.duration,
      data.score,
      data.imageUrl || ""
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({ok: true}))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({
      ok: false,
      error: err.message
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. **ตั้งชื่อโปรเจ็ค:**
   - คลิกที่ "Untitled project" → ตั้งชื่อว่า "Running Scoreboard API"

4. **Deploy เป็น Web App:**
   - คลิก **Deploy → New deployment**
   - คลิกไอคอนเฟือง ⚙️ → เลือก **Type: Web app**
   - **Description:** v1.0
   - **Execute as:** Me (your email)
   - **Who has access:** Anyone
   - คลิก **Deploy**
   - **Copy Web app URL** (จะเป็น `https://script.google.com/macros/s/xxxxx/exec`)
   - คลิก **Done**

5. **ทดสอบ API:**
   - เปิดเบราว์เซอร์ใหม่
   - วาง URL + `?action=getTeams`
   - ตัวอย่าง: `https://script.google.com/macros/s/xxxxx/exec?action=getTeams`
   - ควรเห็น JSON ของข้อมูลทีม

---

## 🚀 ติดตั้งและใช้งาน

### ขั้นตอนที่ 3: แก้ไข Code

1. **เปิดไฟล์ `running-scoreboard.jsx`**

2. **ใส่ OpenAI API Key** (บรรทัดที่ 4):
```javascript
const OPENAI_API_KEY = "sk-proj-xxxxxxxxxxxx"; // ← ใส่ API Key ของคุณตรงนี้
```

3. **ใส่ Google Apps Script URL:**
   
หาบรรทัดที่มี `const GOOGLE_SCRIPT_URL` (ประมาณบรรทัด 1460) แล้วใส่ URL:

```javascript
const GOOGLE_SCRIPT_URL = "https://script.google.com/macros/s/xxxxx/exec";
```

4. **เพิ่มฟังก์ชันโหลดข้อมูลจาก Google Sheets:**

หาฟังก์ชัน `App()` (บรรทัดสุดท้าย) แล้วเพิ่ม `useEffect` นี้ข้างในฟังก์ชัน:

```javascript
export default function App() {
  const [page, setPage] = useState("scoreboard");
  const [teams, setTeams] = useState([]);
  const [members, setMembers] = useState([]);
  const [submissions, setSubmissions] = useState([]);
  const [toasts, setToasts] = useState([]);
  const [loading, setLoading] = useState(true);

  const GOOGLE_SCRIPT_URL = "https://script.google.com/macros/s/xxxxx/exec"; // ← ใส่ URL

  // โหลดข้อมูลจาก Google Sheets
  useEffect(() => {
    async function loadData() {
      try {
        setLoading(true);
        
        // Load Teams
        const teamsRes = await fetch(`${GOOGLE_SCRIPT_URL}?action=getTeams`);
        const teamsData = await teamsRes.json();
        setTeams(teamsData);
        
        // Load Members
        const membersRes = await fetch(`${GOOGLE_SCRIPT_URL}?action=getMembers`);
        const membersData = await membersRes.json();
        setMembers(membersData);
        
        // Load Submissions
        const subsRes = await fetch(`${GOOGLE_SCRIPT_URL}?action=getSubmissions`);
        const subsData = await subsRes.json();
        const subsWithScore = subsData.map(s => ({
          ...s,
          score: s.score || calculateScore(s.distance, s.pace, s.duration)
        }));
        setSubmissions(subsWithScore);
        
        setLoading(false);
      } catch (err) {
        console.error("Error loading data:", err);
        addToast("❌ ไม่สามารถโหลดข้อมูลจาก Google Sheets", "error");
        setLoading(false);
      }
    }
    loadData();
  }, []);

  // ... (โค้ดที่เหลือเหมือนเดิม)
```

5. **อัปเดต handleSubmit ให้บันทึกไปที่ Google Sheets:**

```javascript
async function handleSubmit(data) {
  const today = new Date().toISOString().split("T")[0];
  const existing = submissions.find(s => s.memberId === data.memberId && s.date === today);
  if (existing) {
    addToast("❌ สมาชิกนี้ส่งข้อมูลวันนี้แล้ว (1 ครั้ง/วัน)", "error");
    return;
  }
  
  const score = calculateScore(data.distance, data.pace, data.duration);
  const payload = { ...data, score };
  
  try {
    // ส่งไป Google Sheets
    const res = await fetch(GOOGLE_SCRIPT_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload)
    });
    const result = await res.json();
    
    if (result.ok) {
      // อัปเดต state
      setSubmissions(prev => [...prev, { id: Date.now(), ...payload }]);
      addToast(`✅ บันทึกสำเร็จ! ได้รับ ${score} คะแนน 🎉`);
    } else {
      addToast(`❌ ${result.error}`, "error");
    }
  } catch (err) {
    console.error("Submit error:", err);
    addToast("❌ เกิดข้อผิดพลาดในการบันทึก", "error");
  }
}
```

---

### ขั้นตอนที่ 4: Deploy

**ตัวเลือก A: Vercel (แนะนำสำหรับ Production)**

1. ติดตั้ง Node.js + Vite:
```bash
npm create vite@latest running-scoreboard -- --template react
cd running-scoreboard
npm install
```

2. แทนที่ `src/App.jsx` ด้วยไฟล์ `running-scoreboard.jsx`

3. Build & Deploy:
```bash
npm run build
# หรือ push ไป GitHub แล้ว Deploy ผ่าน Vercel
```

**ตัวเลือก B: HTML เดียว (ง่ายที่สุด)**

สร้างไฟล์ `index.html`:

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Running Scoreboard - ประกันติดโล่</title>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    // วางโค้ดทั้งหมดจากไฟล์ running-scoreboard.jsx ตรงนี้
    // (ไม่รวมบรรทัด import/export)
  </script>
</body>
</html>
```

Upload ไฟล์ HTML นี้ไปที่:
- **GitHub Pages** (ฟรี)
- **Netlify Drop** (ลากไฟล์ทิ้ง ฟรี)
- **Vercel** (ฟรี)

---

## 📖 การใช้งานระบบ

### สำหรับผู้ใช้งานทั่วไป

#### หน้า Scoreboard (🏆)
- ดูตารางคะแนนรายบุคคล / ทีม
- แสดง Top 3 Podium (เหรียญทอง-เงิน-ทองแดง)
- กรองตามทีม
- แสดง Progress Bar เปรียบเทียบ

#### หน้าส่งผล (📤)
1. เลือกชื่อของคุณจาก dropdown
2. อัปโหลดรูป Screenshot จากแอปวิ่ง
   - รองรับ: Strava, Garmin, Nike Run Club, Adidas Running, Apple Health
3. กด **"อ่านข้อมูลด้วย AI"** → รอ 2-5 วินาที
4. ตรวจสอบข้อมูลที่ AI อ่านได้:
   - ระยะทาง (km)
   - Pace (min/km)
   - เวลา (นาที)
   - คะแนนที่จะได้รับ
5. กด **"ยืนยันส่งข้อมูล"**
6. ⚠️ **ส่งได้วันละ 1 ครั้งเท่านั้น**

#### หน้าประวัติ (📅)
- ดูรายการวิ่งย้อนหลังของทุกคน
- กรองตามสมาชิกเฉพาะ
- ดูรูปภาพที่ส่งมา

### สำหรับ Admin

#### เข้าหน้า Admin (⚙️)
1. คลิกแท็บ **Admin**
2. ใส่รหัสผ่าน: **`zxcvbn`**
3. กด "เข้าสู่ระบบ"

#### จัดการทีม
- เพิ่มทีมใหม่ (พร้อมอิโมจิและสีอัตโนมัติ)
- ดูจำนวนสมาชิกในแต่ละทีม

#### จัดการสมาชิก
- เพิ่มสมาชิกใหม่
- เลือกทีมให้สมาชิก
- ดูรายชื่อสมาชิกทั้งหมด

#### Google Sheets Integration
- ใส่ Apps Script URL
- ทดสอบการเชื่อมต่อ
- Sync ข้อมูล (future feature)

---

## 🧮 สูตรคำนวณคะแนน

```
Score = (Distance × 10) + (7.0 - Pace) × 50 + DurationBonus

DurationBonus = (Duration > 60) ? (Duration - 60) × 0.5 : 0
```

### ตัวอย่างการคำนวณ:

| ระยะทาง | Pace | เวลา | การคำนวณ | คะแนน |
|---------|------|------|----------|-------|
| 10 km | 5.0 min/km | 50 min | (10×10) + (7-5)×50 + 0 | **200** |
| 5 km | 6.0 min/km | 30 min | (5×10) + (7-6)×50 + 0 | **100** |
| 15 km | 4.5 min/km | 70 min | (15×10) + (7-4.5)×50 + (70-60)×0.5 | **280** |
| 8 km | 5.5 min/km | 44 min | (8×10) + (7-5.5)×50 + 0 | **155** |

**กลยุทธ์การได้คะแนนสูง:**
- 📏 **วิ่งระยะไกล** = คะแนนเพิ่ม (1 km = 10 pts)
- ⚡ **วิ่งเร็ว** (Pace ต่ำ) = คะแนนโบนัส (Pace < 5.0 ได้โบนัสสูง)
- ⏱ **วิ่งนานกว่า 1 ชม.** = โบนัสเพิ่ม

---

## ❓ FAQ (คำถามที่พบบ่อย)

### Q1: ต้องใช้ OpenAI API Key แบบเสียเงินหรือไม่?
**A:** ใช่ — GPT-4o Vision ไม่มี Free Tier แต่ราคาไม่แพง
- **ราคา:** ~$0.01 ต่อภาพ
- **70 คน × วันละครั้ง** = 70 รูป/วัน ≈ $0.70/วัน ≈ $20/เดือน
- **แนะนำ:** เติม Credit $10-20 ทดสอบก่อน

**ดู API Key ที่:** [platform.openai.com/api-keys](https://platform.openai.com/api-keys)

### Q2: ถ้าไม่อยากจ่ายเงิน OpenAI มีทางเลือกอื่นไหม?
**A:** มี 3 ทางเลือก:
1. **Manual Input** — แก้โค้ดให้กรอกข้อมูลเอง (ไม่ใช้ AI)
2. **Tesseract.js** — OCR ฟรี แต่ความแม่นยำต่ำ (~70%)
3. **Google Cloud Vision API** — มี Free Tier 1,000 ครั้ง/เดือน

### Q3: รองรับแอปวิ่งอะไรบ้าง?
**A:** 
- ✅ Strava
- ✅ Garmin Connect
- ✅ Nike Run Club
- ✅ Adidas Running (Runtastic)
- ✅ Apple Health
- ✅ Samsung Health
- ✅ Suunto
- ✅ Polar
- ✅ Coros
- ✅ Wahoo Fitness

### Q4: จะเพิ่มสมาชิกเกิน 70 คนได้ไหม?
**A:** ได้ — ไม่มีข้อจำกัด เพิ่มข้อมูลใน Google Sheets ได้เรื่อยๆ

### Q5: สามารถแก้ข้อมูลที่ส่งไปแล้วได้ไหม?
**A:** ได้ 2 วิธี:
1. แก้โดยตรงใน Google Sheets (แนะนำ)
2. เพิ่มฟีเจอร์ Edit/Delete ในหน้า Admin (ต้องเขียนโค้ดเพิ่ม)

### Q6: Google Apps Script Error ต้องทำยังไง?
**A:** ตรวจสอบ:
1. ✅ Deploy ถูกต้อง (Execute as: Me, Who has access: Anyone)
2. ✅ ชื่อ Sheet ตรง: `Teams`, `Members`, `Submissions`
3. ✅ URL ลงท้ายด้วย `/exec`
4. ✅ ลอง Re-deploy (Deploy → Manage deployments → Edit → Version: New version → Deploy)

### Q7: ทำไม AI อ่านข้อมูลไม่ถูก?
**A:** เคล็ดลับการถ่ายภาพที่ดี:
- ✅ ถ่าย Screenshot (ไม่ใช่ถ่ายหน้าจอด้วยกล้อง)
- ✅ แสดงข้อมูลครบ: ระยะทาง, Pace, เวลา
- ✅ ภาพชัดเจน ไม่เบลอ
- ✅ ตัวเลขใหญ่พอ อ่านง่าย
- ❌ หลีกเลี่ยงภาพมืด/สว่างเกิน

### Q8: รหัสผ่าน Admin คืออะไร?
**A:** `zxcvbn` (ตั้งไว้ในโค้ด บรรทัดที่ 5)

**หากต้องการเปลี่ยน:** แก้ตรง `const ADMIN_PASSWORD = "zxcvbn";`

### Q9: ข้อมูลใน Google Sheets จะหายไหม?
**A:** ไม่หาย — Google Sheets เก็บข้อมูลถาวร + มี Version History

**แนะนำ:** สำรองข้อมูลโดย File → Download → CSV ทุกสัปดาห์

### Q10: จะใช้ QR Code ยังไง?
**A:** 
1. หน้า Upload แสดง QR Code อัตโนมัติ
2. ถ่ายภาพ QR Code นั้น
3. พิมพ์ติดที่ลู่วิ่ง/ป้าย
4. นักวิ่งสแกนด้วยมือถือ → เข้าหน้า Upload ได้เลย

---

## 🎯 Roadmap (ฟีเจอร์ในอนาคต)

- [ ] Export ข้อมูลเป็น PDF Report
- [ ] ส่ง Notification ผ่าน Line
- [ ] แสดงกราฟสถิติรายสัปดาห์
- [ ] Weekly Challenge (เป้าหมายสัปดาห์)
- [ ] Achievement Badges
- [ ] Mobile App (React Native)

---

## 📞 ติดต่อ & สนับสนุน

- 📧 **Email:** support@pkantidlor.co.th
- 💬 **Line:** @pkantidlor
- 🌐 **Website:** [www.pkantidlor.co.th](https://www.pkantidlor.co.th)

หากพบบั๊กหรือมีข้อเสนอแนะ กรุณาติดต่อทีมพัฒนา

---

## 📜 License & Credits

**License:** MIT License - ใช้งานฟรี แก้ไขได้ตามต้องการ

**Credits:**
- 🤖 AI Development: Claude (Anthropic)
- 🎨 Design: ประกันติดโล่ Team
- 💻 Technology: React, Google Apps Script, GPT-4o Vision

---

**เวอร์ชัน:** 1.0.0  
**อัปเดตล่าสุด:** มิถุนายน 2025  
**สร้างด้วย ❤️ โดยประกันติดโล่**
