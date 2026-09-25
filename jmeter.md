Berikut adalah file **`README.md`** terlengkap yang merangkum seluruh materi, proyek praktik API DummyJSON, skrip Groovy JSR223, teknik parameterisasi CSV, *troubleshooting log*, hingga persiapan *live coding interview*.

Anda dapat langsung menyalin (*copy-paste*) seluruh isi di bawah ini ke dalam file **`README.md`** di repositori atau laptop Anda.

---

```markdown
# 🚀 Master Guide: Apache JMeter API Performance Testing & Live Coding Handbook

Dokumentasi ini dirancang sebagai panduan komprehensif, repositori proyek praktik, dan *cheat sheet* persiapan *technical test / live coding* untuk posisi **Performance Test Engineer / QA Automation**.

---

## 📌 1. Konsep Dasar & Hirarki JMeter

JMeter mengeksekusi komponen berdasarkan urutan prioritas internal (*Execution Order*), **BUKAN** berdasarkan urutan posisi visual dari atas ke bawah.

### Urutan Eksekusi Elemen:
1. **Configuration Elements** (misal: `HTTP Header Manager`, `CSV Data Set Config`)
2. **Pre-Processors** (misal: `JSR223 PreProcessor`)
3. **Timers** (misal: `Gaussian Random Timer`)
4. **Sampler** (misal: `HTTP Request`)
5. **Post-Processors** (misal: `JSON Extractor`, `JSR223 PostProcessor`)
6. **Assertions** (misal: `Response Assertion`)
7. **Listeners** (misal: `View Results Tree`, `Aggregate Report`)

---

## 💡 2. Scope Variabel: `vars` vs `props`

Memahami perbedaan ini adalah pertanyaan favorit dalam *interview* dan kunci desain *Test Plan* yang benar.

| Fitur | `vars` (JMeter Variables) | `props` (JMeter Properties) |
|---|---|---|
| **Cakupan Scope** | **Thread-Local** (Hanya untuk 1 Virtual User). | **Global** (Seluruh Thread Group & Test Plan). |
| **Fungsi Utama** | Korelasi data antar-request dalam 1 alur user. | Membagikan data/token global antar Thread Group. |
| **Cara Menulis** | `vars.put("key", "value")` | `props.put("key", "value")` |
| **Cara Memanggil (JMeter)** | `${key}` | `${__P(key)}` atau `${__property(key)}` |
| **Cara Memanggil (Groovy)**| `vars.get("key")` | `props.get("key")` |

---

## 🧪 3. Studi Kasus API Real-World (DummyJSON API)

Pengujian menggunakan Public API terproteksi dari **DummyJSON** (`dummyjson.com`).

### Spesifikasi Endpoint:
1. **POST Login:** `https://dummyjson.com/auth/login`
   * **Header:** `Content-Type: application/json`
   * **Payload Body:**
     ```json
     {
       "username": "${username}",
       "password": "${password}"
     }
     ```
   * **Response Key:** `accessToken`

2. **GET User Profile (Terproteksi):** `https://dummyjson.com/auth/me`
   * **Header:** `Authorization: Bearer <accessToken>`
   * **Response:** Profile data user (Status `200 OK`).

---

## 🛠️ 4. Step-by-Step Test Plan Construction

Ada 2 pendekatan arsitektur pengujian di JMeter:

### Pendekatan A: Multi-User Flow dalam 1 Thread Group (Best Practice Load Test)
Menggunakan `vars` (Thread-Local) sehingga setiap thread login dengan kredensialnya sendiri dan menggunakan tokennya sendiri secara terisolasi.

```text
Test Plan
└── 📁 Thread Group (Threads: 3, Ramp-Up: 3)
    ├── 📄 CSV Data Set Config (File: users.csv)
    ├── 🌐 HTTP Request: POST Login
    │   ├── ⚙️ HTTP Header Manager (Content-Type: application/json)
    │   └── 🧩 JSON Extractor (Var: localToken | Path: $.accessToken)
    ├── 🌐 HTTP Request: GET Profile Me
    │   └── ⚙️ HTTP Header Manager (Authorization: Bearer ${localToken})
    └── 📊 View Results Tree

```

### Pendekatan B: Multi Thread Group dengan Token Global (`props`)

Digunakan jika Thread Group 1 bertugas sebagai *Auth Setup* (1 user login admin) lalu melempar token ke Thread Group 2 (banyak user).

* **Test Plan Level:** Centang **Run Thread Groups consecutively (run one at a time)**.
* **Thread Group 1 (Auth):** Ekstrak token via JSON Extractor (`accessToken`), lalu jalankan **JSR223 PostProcessor**:
```groovy
props.put("globalToken", vars.get("accessToken"))

```


* **Thread Group 2 (Transactions):** Pada `HTTP Header Manager`, panggil menggunakan:
```text
Authorization: Bearer ${__P(globalToken)}

```



---

## 📄 5. Data Parameterization (CSV Data Set Config)

### 1. Buat File Data (`users.csv`)

> ⚠️ **PENTING:** Jangan mengetik spasi setelah koma agar data tidak terbaca korup!

```csv
username,password
emilys,emilyspass
michaelw,michaelwpass
sophiab,sophiabpass

```

### 2. Konfigurasi `CSV Data Set Config`

* **Filename:** `users.csv` *(atau relative path: `data/users.csv`)*
* **File Encoding:** `UTF-8`
* **Variable Names:** `username,password`
* **Delimiter:** `,`
* **Ignore First Line:** `True`
* **Recycle on EOF?:** `True`
* **Stop Thread on EOF?:** `False`
* **Sharing Mode:** `All threads`

---

## 📜 6. JSR223 Groovy Scripting & String Manipulation

Gunakan selalu bahasa **groovy** pada JSR223 Elements.

### A. Skrip Aman Ekstraksi Token ke Property Global

```groovy
// 1. Ambil nilai token lokal
String token = vars.get("localToken")

// 2. Defensive Check (mencegah crash/NullPointerException)
if (token != null && !token.isEmpty() && !token.equals("NO_TOKEN")) {
    
    // 3. String Manipulation: Rapikan token dari spasi / kutip berlebih
    String cleanToken = token.replace("Bearer ", "").replace("\"", "").trim()
    
    // 4. Simpan ke Global Property
    props.put("globalToken", cleanToken)
    log.info(">>> SUCCESS: Global Token set to: " + cleanToken)
} else {
    log.error(">>> ERROR: Failed to extract accessToken!")
}

```

### B. Cheatsheet String Manipulation & Dynamic Data di Groovy

```groovy
// --- Manipulasi String ---
String text = " Bearer eyJhbGci... "
String clean = text.trim().replace("Bearer ", "") // Menghapus prefix & spasi
String lower = text.toLowerCase()                 // Mengubah ke huruf kecil
String sub   = text.substring(0, 10)              // Ambil 10 karakter pertama
String[] parts = text.split("\\.")               // Split JWT Token berdasarkan titik

// --- Membuat Data Dinamis ---
import java.util.UUID

String randomUUID = UUID.randomUUID().toString()               // Generate UUID
long timestamp    = System.currentTimeMillis()                // Epoch Timestamp
String randomEmail = "qa_user_" + timestamp + "@test.com"     // Dynamic Email

// Simpan ke JMeter Variables
vars.put("dynamic_uuid", randomUUID)
vars.put("dynamic_email", randomEmail)

```

---

## 🚨 7. Panduan Troubleshooting & Debugging Error

Berikut adalah daftar masalah nyata yang sering terjadi dan cara menyelesaikannya:

### 1. `POST Login` Berwarna MERAH padahal Status Code `200 OK`

* **Penyebab:** Ada kesalahan sintaksis pada skrip **JSR223 PostProcessor** di bawah request tersebut.
* **Kesalahan Umum Sintaks:**
* Menulis `var.get()` bukannya `vars.get()` (kurang huruf **s**).
* Menulis `vars.get(token)` tanpa tanda kutip `vars.get("token")`.


* **Solusi:** Buka **Log Viewer** (Ikon Segitiga Kuning-Merah di kanan atas) untuk melihat pesan error kodenya, lalu perbaiki sintaks Groovy.

### 2. Pesan `Invalid credentials` pada Request Login

* **Penyebab:**
* Password DummyJSON salah (misal `emilyspassword`, padahal yang benar `emilyspass`).
* Ada spasi tak terlihat di file CSV (misal `emilys, emilyspass`).
* JSON payload diketik di tab *Parameters* (seharusnya di tab *Body Data*).


* **Solusi:** Rapikan CSV tanpa spasi dan pastikan payload berada di tab **Body Data** dengan `Content-Type: application/json`.

### 3. Request `GET Profile` Mengembalikan `401 Unauthorized`

* **Penyebab:** Authorization header mengirim token kosong atau variabel tidak terurai (misal terkirim teks `${localToken}`).
* **Solusi:** Cek **View Results Tree** $\rightarrow$ Request `GET Profile` $\rightarrow$ Tab **Request Headers**. Pastikan header terisi `Authorization: Bearer eyJhbGci...`.

### 4. Hanya User Terakhir yang Terbaca di Thread Group 2

* **Penyebab:** Menggunakan `props.put()` dalam loop multi-thread. Thread terakhir akan menimpa (*overwrite*) nilai property user sebelumnya.
* **Solusi:** Jika butuh pengujian multi-user sejati, gunakan **1 Thread Group** dan manfaatkan variabel lokal `${localToken}` (`vars`).

---

## 🏃‍♂️ 8. Mode Eksekusi (GUI vs Non-GUI CLI) & Heap Tuning

### ⚠️ Rules Eksekusi

* **Mode GUI:** HANYA untuk pembuatan, penyusunan, dan *debugging* skrip (1–5 virtual users).
* **Mode Non-GUI (CLI):** WAJIB digunakan untuk *Actual Load / Stress Testing*.

### 1. Tuning Heap Memory JMeter

Sebelum tes beban tinggi, tingkatkan alokasi memori JVM pada file `bin/jmeter.bat` (Windows) atau `bin/jmeter` (macOS/Linux):

```bash
# Ubah alokasi HEAP default (misal 2GB Min - 4GB Max)
set HEAP=-Xms2g -Xmx4g -XX:MaxMetaspaceSize=512m

```

### 2. Command Line Interface (CLI) Execution

```bash
jmeter -n -t [path_script.jmx] -l [path_result.jtl] -e -o [path_html_report_folder]

```

**Contoh Perintah:**

```bash
jmeter -n -t ./scripts/DummyJSON_TestPlan.jmx -l ./results/run_01.jtl -e -o ./reports/Dashboard_Run_01

```

---

## 📊 9. Metrik Performa & Analisis SLA

Saat mengevaluasi hasil pengujian melalui **Aggregate Report** atau **Dashboard HTML Report**:

1. **Throughput (TPS - Transactions Per Second):** Total transaksi yang mampu diproses server per detik. Semakin tinggi semakin baik.
2. **90th / 95th Percentile Response Time:** Batas waktu respons maksimum yang dirasakan oleh 90% atau 95% pengguna. Standard SLA API yang baik: **< 2000 ms (2 detik)**.
3. **Error Rate (%):** Persentase kegagalan permintaan (HTTP 4xx/5xx). Target ideal untuk *Load Test* normal: **0.00%**.

---


```

```
