### 1. String Manipulation di Java

Dalam Java, objek `String` bersifat *immutable* (nilainya tidak bisa diubah setelah dibuat), sehingga setiap metode manipulasi String akan mengembalikan objek `String` baru.

```java
public class JavaStringDemo {
    public static void main(String[] args) {
        String text = "  Bearer eyJhbGciOiJIUzI1NiI.payload.signature  ";

        // 1. Menghapus Spasi di Awal & Akhir
        String trimmed = text.trim(); 
        // Hasil: "Bearer eyJhbGciOiJIUzI1NiI.payload.signature"

        // 2. Mengganti / Menghapus Teks
        String cleanToken = trimmed.replace("Bearer ", ""); 
        // Hasil: "eyJhbGciOiJIUzI1NiI.payload.signature"

        // 3. Substring (Mengambil Potongan Karakter berdasarkan Index)
        String firstTenChars = cleanToken.substring(0, 10); 
        // Hasil: 10 karakter pertama

        // 4. Split (Memecah String menjadi Array)
        // Catatan: Di Java, titik (.) adalah karakter khusus Regex, sehingga harus di-escape dengan "\\."
        String[] jwtParts = cleanToken.split("\\."); 
        // jwtParts[0] = Header, jwtParts[1] = Payload, jwtParts[2] = Signature

        // 5. Mengubah Kapitalisasi
        String upper = cleanToken.toUpperCase();
        String lower = cleanToken.toLowerCase();

        // 6. Pengecekan Kandungan Teks (Contains & StartsWith)
        boolean hasBearer = text.contains("Bearer");   // true
        boolean isBearer = trimmed.startsWith("Bearer"); // true

        // 7. Regex Replacement (Misal: Mengambil Angka Saja)
        String numbersOnly = "Order #99823".replaceAll("[^0-9]", ""); 
        // Hasil: "99823"

        // 8. String Formatting
        String formatted = String.format("User ID: %d, Name: %s", 101, "Carlos");
        // Hasil: "User ID: 101, Name: Carlos"
    }
}

```

---

### 2. String Manipulation di JavaScript

JavaScript memiliki sintaks yang mirip dengan Java, namun lebih fleksibel dengan dukungan *Template Literals* (backtick) untuk interpolasi string.

```javascript
const text = "  Bearer eyJhbGciOiJIUzI1NiI.payload.signature  ";

// 1. Menghapus Spasi di Awal & Akhir
const trimmed = text.trim(); 
// Hasil: "Bearer eyJhbGciOiJIUzI1NiI.payload.signature"

// 2. Mengganti / Menghapus Teks
const cleanToken = trimmed.replace("Bearer ", ""); 
// Hasil: "eyJhbGciOiJIUzI1NiI.payload.signature"
// Untuk mengganti semua kemunculan kata: text.replaceAll("target", "replacement")

// 3. Slice / Substring (Mengambil Potongan Karakter)
const firstTenChars = cleanToken.slice(0, 10); // Lebih direkomendasikan dibanding .substring()

// 4. Split (Memecah String menjadi Array)
const jwtParts = cleanToken.split("."); 
// jwtParts[0] = Header, jwtParts[1] = Payload, jwtParts[2] = Signature

// 5. Mengubah Kapitalisasi
const upper = cleanToken.toUpperCase();
const lower = cleanToken.toLowerCase();

// 6. Pengecekan Kandungan Teks (Menggunakan includes, BUKAN contains)
const hasBearer = text.includes("Bearer");     // true
const isBearer = trimmed.startsWith("Bearer"); // true

// 7. Regex Replacement (Menggunakan Flag /g)
const numbersOnly = "Order #99823".replace(/[^0-9]/g, ""); 
// Hasil: "99823"

// 8. Template Literals (String Formatting Modern)
const userId = 101;
const name = "Carlos";
const formatted = `User ID: ${userId}, Name:${name}`; 
// Hasil: "User ID: 101, Name: Carlos"

```

---

### 📊 Tabel Perbandingan Metode (Quick Cheat Sheet)

| Operasi | Java | JavaScript |
| --- | --- | --- |
| **Hapus Spasi Awal/Akhir** | `str.trim()` | `str.trim()` |
| **Ganti Teks (1x)** | `str.replace("a", "b")` | `str.replace("a", "b")` |
| **Ganti Semua Teks** | `str.replaceAll("a", "b")` | `str.replaceAll("a", "b")` / `str.replace(/a/g, "b")` |
| **Potong String** | `str.substring(start, end)` | `str.slice(start, end)` |
| **Memecah ke Array** | `str.split("\\.")` *(wajib escape regex)* | `str.split(".")` |
| **Cek Kandungan Teks** | `str.contains("abc")` | `str.includes("abc")` |
| **Cek Panjang String** | `str.length()` | `str.length` |
| **Huruf Besar / Kecil** | `str.toUpperCase()` / `.toLowerCase()` | `str.toUpperCase()` / `.toLowerCase()` |
| **Format Variable** | `String.format("Hi %s", name)` | ``Hi ${name}`` *(Template Literal)* |
