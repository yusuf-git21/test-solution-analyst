## High Level Design Architecture (Mobile Loan App - PT XYZ)
+-------------------------+
|     Mobile Frontend     |
|  (Android / iOS App)    |
+-----------+-------------+
            |
            v
+-----------+-------------+
|      API Gateway        |
+-----------+-------------+
            |
            v
+-----------+-------------+            +--------------------------+
|        Backend API       |<--------->|  Third-party Integrations |
| (Node.js / Spring / .NET)|           | (Email, SMS, KTP, Payment)|
+-----------+-------------+            +--------------------------+
   |         |         |         |         |
   v         v         v         v         v
+------+  +--------+ +--------+ +--------+ +------------------+
| Auth |  |  Loan  | |  User  | | Notify | | Validation Engine |
| Svc  |  | Mgmt   | | Profile| |  Svc   | | (Loan eligibility)|
+------+  +--------+ +--------+ +--------+ +------------------+
   |
   v
+------------------+
|     Database     |
|     (MySQL)      |
+------------------+

# Key Components
- **Frontend (Mobile App)**:
Dibuat dengan Flutter/React Native.
Mendukung login/password & biometric (fingerprint/FaceID).

- **Backend API**:
Modular service-based: Auth, Loan, User Profile, Notification, Validation.
Database (MySQL):
Menyimpan data user, loan, repayment.

- **Third-party Integrations**:
KTP Verification API (eKYC)
Email & SMS gateway
Payment gateway

- **Authentication Service**:
Menangani login, logout, dan otentikasi biometric (via device).
Loan Management Service:
Pembuatan pinjaman, pengecekan status, pelunasan.

- **User Profile Service**:
Registrasi, upload dokumen KTP & foto selfie, update profil.

-**Notification Service**:
Mengirimkan notifikasi status pinjaman via email/SMS.

- **Validation Engine**:
Memastikan tidak ada pinjaman aktif sebelum pengajuan baru.

## Screen Flow & ERD
Screen Flow (Simplified):
- **Splash Screen**
- **Login / Register Screen**
Register (Name, Email, Phone, KTP, Selfie)
Login (Email + Password / Biometric)

-**Home Screen**
View Loan Status (Sisa hutang, Tagihan bulanan)
Button: Ajukan Pinjaman

-**Loan Application**
Form: Jumlah, Tenor
Submit → Proses Verifikasi

-**Loan Status Screen**
Menunggu, Ditolak, Diterima

-**otifikasi / Riwayat Pinjaman**

## ERD (Entity Relationship Diagram):
Users
- id (PK)
- name
- email
- phone
- password_hash
- ktp_photo
- selfie_photo
- created_at

Loans
- id (PK)
- user_id (FK)
- amount
- tenor
- status (pending/accepted/rejected)
- created_at
- approved_at

Repayments
- id (PK)
- loan_id (FK)
- due_date
- amount_due
- is_paid

Notifications
- id (PK)
- user_id (FK)
- type (email/sms)
- message
- sent_at

# Flowchart ERD Diagram
flowchart TD
    %% Entitas
    U[Users]
    L[Loans]
    R[Repayments]
    N[Notifications]

    %% Kolom Users
    U --> U1[id (PK)]
    U --> U2[name]
    U --> U3[email]
    U --> U4[phone]
    U --> U5[password_hash]
    U --> U6[ktp_photo]
    U --> U7[selfie_photo]
    U --> U8[created_at]

    %% Kolom Loans
    L --> L1[id (PK)]
    L --> L2[user_id (FK)]
    L --> L3[amount]
    L --> L4[tenor]
    L --> L5[status (pending/accepted/rejected)]
    L --> L6[created_at]
    L --> L7[approved_at]

    %% Kolom Repayments
    R --> R1[id (PK)]
    R --> R2[loan_id (FK)]
    R --> R3[due_date]
    R --> R4[amount_due]
    R --> R5[is_paid]

    %% Kolom Notifications
    N --> N1[id (PK)]
    N --> N2[user_id (FK)]
    N --> N3[type (email/sms)]
    N --> N4[message]
    N --> N5[sent_at]

    %% Relasi antar entitas
    U -->|has| L
    L -->|has| R
    U -->|receives| N

## Detail API Design

- **User Registration**
POST /api/register
Request:
{
  "name": "Maulana",
  "email": "maulana@mail.com",
  "phone": "08123456789",
  "password": "securePass",
  "ktp_photo": "base64image",
  "selfie_photo": "base64image"
}
Response:
{ "status": "success", "user_id": "1" }

- **Login API**
POST /api/login

- **Loan Request**
POST /api/loans
Request:
{
  "user_id": 1,
  "amount": 8000000,
  "tenor": 12
}

Response
{ "status": "pending", "loan_id": 101 }

- **Loan Status**
GET /api/loans/:user_id
Request:

Method: GET
Endpoint: /api/loans/{user_id}
Headers:

Authorization: Bearer <token>
Content-Type: application/json

Response - Success (Loan Found) :

{
  "status": "success",
  "data": {
    "loan_id": 101,
    "user_id": 1,
    "amount": 8000000,
    "tenor": 12,
    "monthly_installment": 720000,
    "remaining_amount": 5760000,
    "status": "active",
    "created_at": "2025-03-01T10:00:00Z",
    "due_date": "2026-03-01",
    "repayments": [
      {
        "month": 1,
        "due_date": "2025-04-01",
        "amount_due": 720000,
        "is_paid": true
      },
      {
        "month": 2,
        "due_date": "2025-05-01",
        "amount_due": 720000,
        "is_paid": false
      }
    ]
  }
}

 Response - No Active Loan :

 {
  "status": "success",
  "data": null,
  "message": "Tidak ada pinjaman aktif untuk user ini."
}

Response - Error (User Not Found) :
{
  "status": "error",
  "message": "User tidak ditemukan."
}

- **Notifikasi**
POST /api/notifications/send

Request
Method: POST
Endpoint: /api/notifications/send
Headers:

Authorization: Bearer <token>
Content-Type: application/json


Body:
{
  "user_id": 1,
  "type": "email", // or "sms"
  "subject": "Pengajuan Pinjaman Diterima",
  "message": "Selamat! Pengajuan pinjaman Anda sebesar Rp8.000.000 telah disetujui.",
  "metadata": {
    "loan_id": 101,
    "amount": 8000000,
    "status": "accepted"
  }
}

 Response - Success:
 {
  "status": "success",
  "message": "Notifikasi berhasil dikirim ke user.",
  "data": {
    "notification_id": 202,
    "sent_at": "2025-04-05T14:15:00Z"
  }
}

Response - Error (User Not Found)

{
  "status": "error",
  "message": "User dengan ID tersebut tidak ditemukan."
}

## Screen Behavior
# Screen Behavior (UX/UI Notes)

| **Screen**           | **Behavior**                                                                                                                                  |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| **Login / Register** | - Validasi format email dan nomor telepon secara realtime  <br> - Upload gambar KTP & selfie dengan preview sebelum submit                   |
| **Home**             | - Menampilkan sisa hutang dan tagihan aktif jika ada <br> - Tombol pengajuan pinjaman akan **disable** jika masih ada pinjaman aktif         |
| **Loan Application** | - Input terbatas: jumlah pinjaman maksimal **Rp12.000.000**, tenor maksimal **12 bulan** <br> - Validasi dilakukan di **frontend & backend** |
| **Loan Status**      | - Indikator status pinjaman: <br> &nbsp;&nbsp;• Pending: warna **kuning**  <br> &nbsp;&nbsp;• Ditolak: warna **merah** <br> &nbsp;&nbsp;• Diterima: warna **hijau** |
| **Notifikasi**       | - Menampilkan daftar riwayat notifikasi terkait status pinjaman <br> - Menampilkan waktu notifikasi dikirim                                   |




