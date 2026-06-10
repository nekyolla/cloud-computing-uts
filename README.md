# Tugas Proyek UTS Cloud Computing — Kelompok 4 TI-C2

Proyek mata kuliah **Cloud Computing** yang mengimplementasikan tiga modul berbasis **API Contract Simple v1**: Presensi QR Dinamis, Accelerometer Telemetry, dan GPS Tracking + Peta.


---

## Anggota Kelompok

| No. | Nama                | NIM       | Role                   |
|-----|---------------------|-----------|------------------------|
| 1   | Panji Bachtiar      | 434231029 | QA & Swap Testing      |
| 2   | Helmi Said H        | 434231080 | Client Web             |
| 3   | Nouval Aryanta      | 434231071 | Backend / API          |
| 4   | Abdillah Muharrarul | 434231053 | Dokumentasi & Deploy   |

---

## Tech Stack

| Layer    | Teknologi                                 |
|----------|-------------------------------------------|
| Backend  | Google Apps Script (GAS) + Google Sheets  |
| Client   | TypeScript / Next.js (Vercel)             |
| Test     | Postman Collection                        |
| Storage  | Google Sheets (tokens, presence, accel, gps) |

---

## Base URL

```
{{BASE_URL}} = https://script.google.com/macros/s/<deployment-id>/exec
```

> Ganti `<deployment-id>` dengan ID deployment GAS.  
> Semua request menggunakan base URL ini dengan `pathInfo`.

---

## Format Response Standar

**Sukses:**
```json
{ "ok": true, "data": { } }
```

**Gagal:**
```json
{ "ok": false, "error": "pesan error singkat" }
```

Error umum: `token_expired`, `token_invalid`, `missing_field: user_id`, `device_not_found`

---

## Modul 1 — Presensi QR Dinamis

Sistem presensi berbasis QR token dinamis untuk mencegah titip absen.

### Alur
Dosen generate QR → tampil di kelas → mahasiswa scan → client kirim check-in → server validasi token → simpan presensi → cek status.

### Endpoints

#### 1. Generate QR Token
```
POST {{BASE_URL}}/presence/qr/generate
```
Request:
```json
{
  "course_id": "cloud-101",
  "session_id": "sesi-02",
  "ts": "2026-02-18T10:00:00Z"
}
```
Response:
```json
{
  "ok": true,
  "data": {
    "qr_token": "TKN-8F2A19",
    "expires_at": "2026-02-18T10:02:00Z"
  }
}
```

#### 2. Check-in
```
POST {{BASE_URL}}/presence/checkin
```
Request:
```json
{
  "user_id": "2023xxxx",
  "device_id": "dev-001",
  "course_id": "cloud-101",
  "session_id": "sesi-02",
  "qr_token": "TKN-8F2A19",
  "ts": "2026-02-18T10:01:10Z"
}
```
Response:
```json
{
  "ok": true,
  "data": {
    "presence_id": "PR-0001",
    "status": "checked_in"
  }
}
```

#### 3. Cek Status
```
GET {{BASE_URL}}/presence/status?user_id=2023xxxx&course_id=cloud-101&session_id=sesi-02
```
Response:
```json
{
  "ok": true,
  "data": {
    "user_id": "2023xxxx",
    "course_id": "cloud-101",
    "session_id": "sesi-02",
    "status": "checked_in",
    "last_ts": "2026-02-18T10:01:10Z"
  }
}
```

---

## Modul 2 — Accelerometer Telemetry

Pengiriman data sensor accelerometer dari smartphone secara batch ke cloud.

### Alur
Client baca sensor (x, y, z) → kumpulkan batch → POST ke server → server simpan ke Sheets → client GET latest untuk ditampilkan.

### Endpoints

#### 1. Kirim Data Batch
```
POST {{BASE_URL}}/telemetry/accel
```
Request:
```json
{
  "device_id": "dev-001",
  "ts": "2026-02-18T10:15:30Z",
  "samples": [
    { "t": "2026-02-18T10:15:29.100Z", "x": 0.12, "y": 0.01, "z": 9.70 },
    { "t": "2026-02-18T10:15:29.300Z", "x": 0.15, "y": 0.02, "z": 9.68 }
  ]
}
```
Response:
```json
{ "ok": true, "data": { "accepted": 2 } }
```

#### 2. Ambil Data Terbaru
```
GET {{BASE_URL}}/telemetry/accel/latest?device_id=dev-001
```
Response:
```json
{
  "ok": true,
  "data": { "t": "2026-02-18T10:15:29.300Z", "x": 0.15, "y": 0.02, "z": 9.68 }
}
```

---

## Modul 3 — GPS Tracking + Peta

Pengiriman lokasi GPS dari smartphone dan tampil di peta sebagai marker + polyline.

### Alur
Client minta izin GPS → baca lat/lng → POST ke server → client GET latest (marker) & history (polyline) → tampilkan di peta.

### Endpoints

#### 1. Log GPS Point
```
POST {{BASE_URL}}/telemetry/gps
```
Request:
```json
{
  "device_id": "dev-001",
  "ts": "2026-02-18T10:15:30Z",
  "lat": -7.2575,
  "lng": 112.7521,
  "accuracy_m": 12.5
}
```
Response:
```json
{ "ok": true, "data": { "accepted": true } }
```

#### 2. Ambil GPS Terbaru (Marker)
```
GET {{BASE_URL}}/telemetry/gps/latest?device_id=dev-001
```
Response:
```json
{
  "ok": true,
  "data": {
    "ts": "2026-02-18T10:15:30Z",
    "lat": -7.2575,
    "lng": 112.7521,
    "accuracy_m": 12.5
  }
}
```

#### 3. Ambil GPS History (Polyline)
```
GET {{BASE_URL}}/telemetry/gps/history?device_id=dev-001&limit=200
```
Response:
```json
{
  "ok": true,
  "data": {
    "device_id": "dev-001",
    "items": [
      { "ts": "2026-02-18T10:15:00Z", "lat": -7.2570, "lng": 112.7515 },
      { "ts": "2026-02-18T10:15:10Z", "lat": -7.2572, "lng": 112.7518 },
      { "ts": "2026-02-18T10:15:30Z", "lat": -7.2575, "lng": 112.7521 }
    ]
  }
}
```

---

## Cara Menjalankan

### Backend (Google Apps Script)

1. Buka [Google Apps Script](https://script.google.com) dan buat project baru
2. Copy semua file dari folder `backend/` ke editor GAS
3. Buat Google Sheets baru dengan sheet: `tokens`, `presence`, `accel`, `gps`
4. Sesuaikan `SPREADSHEET_ID` di konfigurasi script
5. Deploy sebagai **Web App** → Execute as: Me → Who has access: Anyone
6. Salin URL deployment (`/exec`) sebagai `BASE_URL`

### Client Web

```bash
cd client
npm install
# Buat file .env.local
echo "NEXT_PUBLIC_BASE_URL=https://script.google.com/macros/s/<id>/exec" > .env.local
npm run dev
```

Akses di `http://localhost:3000`

### Smoke Test (Postman)

1. Import file dari folder `collection/`
2. Set variable `BASE_URL` ke URL deployment GAS
3. Jalankan collection runner

---
