# 🚀 Panduan Ringkas Penggunaan Apache JMeter

Panduan praktis dan cepat untuk penyusunan skrip pengujian beban API, korelasi data, skrip Groovy, dan eksekusi pengujian.

---

## 📌 1. Alur Pengujian Utama (Standard API Flow)

```text
[CSV Data Set Config]
        ↓
1. POST /login ──► Body: {"username": "${username}", "password": "${password}"}
        ↓
2. JSON Extractor ──► Variable: localToken | JSONPath: $.accessToken
        ↓
3. GET /profile ──► Header: Authorization: Bearer ${localToken}

```

---

## 💡 2. Bedah Variabel: `vars` vs `props`

| Jenis | Scope | Kegunaan | Cara Panggil |
| --- | --- | --- | --- |
| **`vars`** | **Thread-Local** (1 User) | Korelasi data unik antar-request dalam 1 Thread Group. | `${localToken}` |
| **`props`** | **Global** (Seluruh Test Plan) | Membagikan data/token umum antar Thread Group berbeda. | `${__P(globalToken)}` |

---

## 📄 3. Quick Setup CSV Data Set Config

1. **File `users.csv**` (tanpa spasi setelah koma):
```csv
username,password
emilys,emilyspass
michaelw,michaelwpass

```


2. **Konfigurasi Elemen:**
* **Filename:** `users.csv`
* **Variable Names:** `username,password`
* **Delimiter:** `,`
* **Ignore first line:** `True`
* **Recycle on EOF?:** `True`



---

## 📜 4. Groovy Cheat Sheet (JSR223 PostProcessor)

### Menyiapkan Global Property dari Variable Lokal:

```groovy
String token = vars.get("localToken")

if (token != null && !token.isEmpty() && !token.equals("NO_TOKEN")) {
    // String Manipulation: Bersihkan token
    String cleanToken = token.replace("Bearer ", "").replace("\"", "").trim()
    
    // Simpan ke Property Global
    props.put("globalToken", cleanToken)
} else {
    log.error("Gagal mendapatkan token!")
}

```

---

## 🏃‍♂️ 5. Perintah Eksekusi CLI (Non-GUI Mode)

> ⚠️ **Aturan Utama:** Mode GUI hanya untuk membuat/debug skrip. Load test wajib menggunakan Non-GUI CLI.

```bash
jmeter -n -t TestPlan.jmx -l result.jtl -e -o ./HTMLReport

```

* `-n` : Run Mode Non-GUI / CLI.
* `-t` : Path file skrip `.jmx`.
* `-l` : Path log file hasil pengujian `.jtl`.
* `-e -o` : Generate Dashboard HTML Report otomatis ke folder tujuan.

---

## 🚨 6. Checklist Debugging Cepat

* **Status 200 OK Tapi Sampler Merah:** Cek sintaks Groovy di JSR223 (misal: penulisan `var` padahal harus `vars`, atau nama variabel tanpa tanda kutip). Buka **Log Viewer** ($\Delta$ kanan atas).
* **Error 401 Unauthorized:** Cek tab *Request Headers* di View Results Tree, pastikan header `Authorization` terisi token asli, bukan teks `${localToken}`.
* **Out of Memory Error:** Matikan (*disable*) listener `View Results Tree` sebelum menjalankan tes dengan jumlah *user/thread* besar.
