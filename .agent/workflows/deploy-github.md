---
description: วิธีอัปโหลด MkDocs Documentation ขึ้น GitHub และ Deploy GitHub Pages
---

# วิธีอัปโหลด MkDocs ขึ้น GitHub (DIY Project)

## ขั้นตอนที่ 1: เตรียม Git Repository

```bash
cd /media/devg/Micro-SV7/GitHub/GhostMicro/micro-electronic-project-diy

# ตรวจสอบสถานะ
git status

# เพิ่มไฟล์ทั้งหมด
git add .

# Commit
git commit -m "Update DIY documentation and add deployment flow"
```

## ขั้นตอนที่ 2: Push ขึ้น GitHub

```bash
# ถ้ายังไม่มี remote (เปลี่ยน YOUR_USERNAME เป็นชื่อของคุณ)
git remote add origin https://github.com/YOUR_USERNAME/micro-electronic-project-diy.git

# Push
git push -u origin main
```

## ขั้นตอนที่ 3: ตรวจสอบ GitHub Actions

ทุกครั้งที่คุณ `git push` ขึ้น branch `main` ระบบจะรัน GitHub Actions อัตโนมัติ (ดูได้ที่แท็บ **Actions** ในหน้า GitHub)

## ขั้นตอนที่ 4: เปิดใช้งาน GitHub Pages

1. ไปที่ GitHub Repository ของคุณ
2. ไปที่ **Settings** -> **Pages**
3. ภายใต้หัวข้อ **Build and deployment** -> **Branch**:
    - เลือก `gh-pages`
    - เลือก `/ (root)`
4. กด **Save**

## ขั้นตอนที่ 5: เข้าชมเว็บไซต์

เว็บไซต์ของคุณจะออนไลน์อยู่ที่:
`https://YOUR_USERNAME.github.io/micro-electronic-project-diy/`

---

## คำสั่งที่ใช้บ่อย

### อัปเดตเอกสารเพิ่ม
```bash
git add .
git commit -m "Update content"
git push
```

### ทดสอบดูในเครื่อง (Local)
```bash
mkdocs serve
```
