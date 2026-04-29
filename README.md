
**Nama: Alipiani Dwi Putri** 

**NIM: 312410691**          

**Kelas: I241B**            

**Mata Kuliah: Pemrograman Web 2**

**Dosen Pengampu: Agung Nugroho, S.Kom., M.Kom.**

**Artikel Ilmiah** 

---

# Eksperimen XSS (Cross-Site Scripting)
### Script Berbahaya di Balik Kolom Komentar: Eksperimen XSS dan Cara Mencegahnya

---

## Deskripsi Proyek

Repository ini berisi hasil eksperimen langsung terkait kerentanan **Cross-Site Scripting (XSS)** pada aplikasi web berbasis PHP. Eksperimen dilakukan dalam lingkungan lokal menggunakan **XAMPP** dan **Visual Studio Code** sebagai bagian dari tugas UTS mata kuliah Pemrograman Web 2.

Eksperimen ini membuktikan bahwa:
- Aplikasi web tanpa sanitasi input rentan terhadap serangan XSS
- Fungsi `htmlspecialchars()` pada PHP efektif mencegah serangan XSS
- Dampak XSS bisa sangat serius mulai dari pencurian cookie hingga defacement halaman

---

## Lingkungan Eksperimen

| Komponen | Spesifikasi |
|---|---|
| Sistem Operasi | Windows 11 |
| Web Server | XAMPP v3.3.0 (Apache 2.4.58 + PHP 8.2.12) |
| Editor Kode | Visual Studio Code |
| Bahasa | PHP & JavaScript |
| Browser | Google Chrome |
| Aplikasi Target | Kolom komentar PHP buatan sendiri |

---

## Struktur File

```
xss-eksperimen/
│
├── index.php        # Versi RENTAN — tanpa sanitasi input
├── aman.php         # Versi AMAN — dengan htmlspecialchars()
├── README.md        # Dokumentasi eksperimen ini
│
└── img/             # Folder screenshot hasil eksperimen
    ├── 01-halaman-rentan.png
    ├── 02-form-diisi-payload.png
    ├── 03-alert-hacked.png
    ├── 04-alert-cookie.png
    ├── 05-halaman-diretas.png
    ├── 06-halaman-aman.png
    ├── 07-form-aman-diisi.png
    └── 08-hasil-mitigasi.png
```

---

## File 1: index.php — Versi RENTAN (Tanpa Sanitasi)

File ini adalah simulasi halaman web yang **sengaja dibuat rentan** terhadap XSS. Input dari pengguna langsung ditampilkan ke HTML tanpa proses sanitasi apapun.

```php
<?php
$komentar_list = [];
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $nama = $_POST['nama'];
    $pesan = $_POST['pesan'];
    // SENGAJA TIDAK ADA SANITASI - RENTAN XSS
    $komentar_list[] = ['nama' => $nama, 'pesan' => $pesan];
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Kolom Komentar (RENTAN XSS)</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 600px; margin: 40px auto; padding: 0 20px; }
        .kotak { background: #fff3f3; border: 2px solid #ff4444; border-radius: 8px; padding: 20px; margin-bottom: 20px; }
        .kotak h2 { color: #ff4444; margin-top: 0; }
        input, textarea { width: 100%; padding: 8px; margin: 6px 0 14px; box-sizing: border-box; border: 1px solid #ccc; border-radius: 4px; }
        button { background: #ff4444; color: white; padding: 10px 24px; border: none; border-radius: 4px; cursor: pointer; font-size: 15px; }
        .komentar-item { background: #f9f9f9; border: 1px solid #ddd; border-radius: 6px; padding: 12px; margin-top: 10px; }
        .label-bahaya { display: inline-block; background: #ff4444; color: white; font-size: 11px; padding: 2px 8px; border-radius: 4px; margin-bottom: 10px; }
    </style>
</head>
<body>
    <span class="label-bahaya">⚠ VERSI RENTAN - Tanpa Sanitasi</span>
    <h1>Kolom Komentar</h1>

    <div class="kotak">
        <h2>Tulis Komentar</h2>
        <form method="POST">
            <label>Nama:</label>
            <input type="text" name="nama" placeholder="Masukkan nama kamu" required>
            <label>Pesan:</label>
            <textarea name="pesan" rows="3" placeholder="Tulis pesan..." required></textarea>
            <button type="submit">Kirim Komentar</button>
        </form>
    </div>

    <?php if (!empty($komentar_list)): ?>
        <h2>Komentar Masuk:</h2>
        <?php foreach ($komentar_list as $k): ?>
            <div class="komentar-item">
                <!-- BERBAHAYA: Input langsung ditampilkan tanpa sanitasi -->
                <strong><?= $k['nama'] ?></strong>
                <p><?= $k['pesan'] ?></p>
            </div>
        <?php endforeach; ?>
    <?php endif; ?>
</body>
</html>
```

---

## File 2: aman.php — Versi AMAN (Dengan Sanitasi)

File ini adalah versi yang sudah diproteksi menggunakan `htmlspecialchars()` untuk mencegah eksekusi skrip berbahaya.

```php
<?php
$komentar_list = [];
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    //  SANITASI DENGAN htmlspecialchars() - AMAN DARI XSS
    $nama = htmlspecialchars($_POST['nama'], ENT_QUOTES, 'UTF-8');
    $pesan = htmlspecialchars($_POST['pesan'], ENT_QUOTES, 'UTF-8');
    $komentar_list[] = ['nama' => $nama, 'pesan' => $pesan];
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Kolom Komentar (AMAN)</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 600px; margin: 40px auto; padding: 0 20px; }
        .kotak { background: #f0fff4; border: 2px solid #22c55e; border-radius: 8px; padding: 20px; margin-bottom: 20px; }
        .kotak h2 { color: #16a34a; margin-top: 0; }
        input, textarea { width: 100%; padding: 8px; margin: 6px 0 14px; box-sizing: border-box; border: 1px solid #ccc; border-radius: 4px; }
        button { background: #22c55e; color: white; padding: 10px 24px; border: none; border-radius: 4px; cursor: pointer; font-size: 15px; }
        .komentar-item { background: #f9f9f9; border: 1px solid #ddd; border-radius: 6px; padding: 12px; margin-top: 10px; }
        .label-aman { display: inline-block; background: #22c55e; color: white; font-size: 11px; padding: 2px 8px; border-radius: 4px; margin-bottom: 10px; }
    </style>
</head>
<body>
    <span class="label-aman">✓ VERSI AMAN - Dengan htmlspecialchars()</span>
    <h1>Kolom Komentar</h1>

    <div class="kotak">
        <h2>Tulis Komentar</h2>
        <form method="POST">
            <label>Nama:</label>
            <input type="text" name="nama" placeholder="Masukkan nama kamu" required>
            <label>Pesan:</label>
            <textarea name="pesan" rows="3" placeholder="Tulis pesan..." required></textarea>
            <button type="submit">Kirim Komentar</button>
        </form>
    </div>

    <?php if (!empty($komentar_list)): ?>
        <h2>Komentar Masuk:</h2>
        <?php foreach ($komentar_list as $k): ?>
            <div class="komentar-item">
                <!-- AMAN: Input sudah disanitasi sebelum ditampilkan -->
                <strong><?= $k['nama'] ?></strong>
                <p><?= $k['pesan'] ?></p>
            </div>
        <?php endforeach; ?>
    <?php endif; ?>
</body>
</html>
```

---

## Hasil Eksperimen

### 1. Serangan Dasar: Alert Popup

**Payload yang digunakan:**
```javascript
<script>alert('HACKED! Ini adalah serangan XSS!')</script>
```

**Langkah:**
1. Buka `localhost/xss-eksperimen/index.php`
2. Isi kolom Nama dengan: `Alifi`
3. Isi kolom Pesan dengan payload di atas
4. Klik tombol **Kirim Komentar**

**Hasil:** Browser langsung memunculkan pop-up dialog bertuliskan _"HACKED! Ini adalah serangan XSS!"_ — membuktikan JavaScript berhasil dieksekusi.

<img width="986" height="674" alt="Cuplikan layar 2026-04-25 204852" src="https://github.com/user-attachments/assets/508b8a2c-3ef2-439e-a660-626708e3ec89" />

*Gambar 1. Tampilan awal halaman index.php yang rentan*




<img width="1919" height="936" alt="Cuplikan layar 2026-04-25 205002" src="https://github.com/user-attachments/assets/b98bb301-e62c-4a99-9d1b-a47f286fcee7" />

*Gambar 2. Kolom Pesan diisi dengan payload XSS*




<img width="1919" height="450" alt="Cuplikan layar 2026-04-25 205029" src="https://github.com/user-attachments/assets/4c6c0303-1131-4350-b487-beac8016bf5d" />

*Gambar 3. Pop-up alert berhasil muncul — XSS terkonfirmasi*

---

### 2. Pencurian Cookie Sesi

**Payload yang digunakan:**
```javascript
<script>alert('Cookie kamu: ' + document.cookie)</script>
```

**Hasil:** Browser menampilkan nilai cookie sesi yang tersimpan. Dalam aplikasi nyata, data ini bisa dikirim ke server penyerang untuk melakukan **session hijacking** — mengambil alih akun pengguna tanpa perlu tahu password.

<img width="1919" height="428" alt="Cuplikan layar 2026-04-25 205123" src="https://github.com/user-attachments/assets/7207ad5a-23d7-4fb7-aef7-c93bd401eed4" />

*Gambar 4. Pop-up menampilkan nilai cookie sesi pengguna*

---

### 3. Defacement Halaman

**Payload yang digunakan:**
```javascript
<script>
  document.body.style.backgroundColor = 'red';
  document.body.innerHTML = '<h1 style=color:white>HALAMAN INI TELAH DIRETAS!</h1>';
</script>
```

**Hasil:** Seluruh konten halaman web hilang dan digantikan layar merah dengan tulisan "HALAMAN INI TELAH DIRETAS!" — membuktikan penyerang dapat memanipulasi penuh apa yang dilihat pengguna.

<img width="1919" height="472" alt="Cuplikan layar 2026-04-25 205155" src="https://github.com/user-attachments/assets/bc4d983c-d909-4fd6-a962-c084c5d8492d" />

*Gambar 5. Halaman berubah total — demonstrasi defacement via XSS*

---

### 4. Mitigasi dengan htmlspecialchars()

Payload yang sama diuji pada `aman.php` yang menggunakan `htmlspecialchars()`.

**Hasil:** Tidak ada pop-up. Payload tampil sebagai **teks mentah** di kolom komentar — membuktikan mitigasi bekerja sempurna.

<img width="1918" height="739" alt="Cuplikan layar 2026-04-25 210815" src="https://github.com/user-attachments/assets/2bf48d23-51d1-4a9e-9cfb-80aa7ef5a6bc" />

*Gambar 6. Tampilan aman.php dengan proteksi aktif*




<img width="1919" height="714" alt="Cuplikan layar 2026-04-25 210852" src="https://github.com/user-attachments/assets/74838db9-5811-4798-b39b-e29a9381cd4d" />

*Gambar 7. Payload yang sama dimasukkan ke versi aman*




<img width="1914" height="855" alt="Cuplikan layar 2026-04-25 210905" src="https://github.com/user-attachments/assets/0aa55c7e-ea56-430c-8ea8-3b9aa6fe5c87" />

*Gambar 8. Payload tampil sebagai teks biasa — serangan digagalkan*

---

## Ringkasan Hasil

| Skenario | Payload | Hasil di index.php (Rentan) | Hasil di aman.php (Aman) |
|---|---|---|---|
| 1 — Alert dasar | `alert('HACKED!')` | ✅ Pop-up muncul | ❌ Tampil sebagai teks |
| 2 — Curi cookie | `alert(document.cookie)` | ✅ Cookie tertampil | ❌ Tampil sebagai teks |
| 3 — Defacement | Manipulasi DOM | ✅ Halaman berubah merah | ❌ Tampil sebagai teks |

---

## Cara Kerja Mitigasi

Fungsi `htmlspecialchars()` mengonversi karakter berbahaya menjadi HTML entities:

| Karakter Asli | Diubah Menjadi | Efek |
|---|---|---|
| `<` | `&lt;` | Tidak dikenali sebagai tag HTML |
| `>` | `&gt;` | Tidak dikenali sebagai tag HTML |
| `'` | `&#039;` | Tidak memutus atribut HTML |
| `"` | `&quot;` | Tidak memutus atribut HTML |

Sehingga payload `<script>alert('XSS')</script>` diubah menjadi:
```
&lt;script&gt;alert(&#039;XSS&#039;)&lt;/script&gt;
```
Yang tampil di browser sebagai teks biasa, bukan kode yang dieksekusi.

---

## Referensi

1. Gustiyono, A., Alwi, E. I., & Abdullah, S. M. (2024). Analisa kerentanan website terhadap serangan Cross-Site Scripting (XSS) metode penetration testing. *Cyber Security dan Forensik Digital*, 7(1), 25-33.
2. OWASP Foundation. (2021). OWASP Top 10:2021 — A03:2021 Injection. https://owasp.org/Top10/
3. The PHP Group. (2024). PHP Manual: htmlspecialchars. https://www.php.net/manual/en/function.htmlspecialchars.php

---

# Hasil Plagiasi

<img width="1674" height="527" alt="Cuplikan layar 2026-04-29 154412" src="https://github.com/user-attachments/assets/635144e0-5794-4e72-b629-1f7b810bf29c" />

