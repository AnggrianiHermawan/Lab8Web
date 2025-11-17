# Praktikum 8: PHP dan Database MySQL

> **Nama:** Anggriani Hermawan
> **NIM:** 312410175  
> **Mata Kuliah:** Pemrograman Web  
> **Dosen:** Agung Nugroho, S.Kom., M.Kom

---

## Tujuan Praktikum
Mahasiswa mampu memahami konsep dasar Database.
Mahasiswa mampu memahami konsep dasar CRUD menggunakan PHP.
Mahasaswa mampu membuat program CRUD sederhana menggunakan PHP.

## 1. Membuat Database & Tabel
### A. Akses phpMyAdmin
Buka browser: http://localhost/phpmyadmin/

### B. Buat Database Baru
CREATE DATABASE latihan1;

<img width="135" height="72" alt="Cuplikan layar 2025-11-17 205449" src="https://github.com/user-attachments/assets/c1b6fca9-567f-4f8f-9a29-7e87429c7353" />

### C. Buat Tabel data_barang
```
CREATE TABLE data_barang (
  id_barang INT(10) AUTO_INCREMENT PRIMARY KEY,
  kategori VARCHAR(30),
  nama VARCHAR(30),
  gambar VARCHAR(100),
  harga_beli DECIMAL(10,0),
  harga_jual DECIMAL(10,0),
  stok INT(4)
);
```
<img width="98" height="40" alt="Cuplikan layar 2025-11-17 205721" src="https://github.com/user-attachments/assets/2e1504e3-ea16-4b07-8b1d-4bca0107e8a0" />

### D. Tambah Data Awal
```
INSERT INTO data_barang (kategori, nama, gambar, harga_beli, harga_jual, stok)
VALUES
('Elektronik', 'HP Samsung Android', 'hp_samsung.jpg', 2000000, 2400000, 5),
('Elektronik', 'HP Xiaomi Android', 'hp_xiaomi.jpg', 1000000, 1400000, 5),
('Elektronik', 'HP OPPO Android', 'hp_oppo.jpg', 1800000, 2300000, 5);
```
<img width="505" height="73" alt="Cuplikan layar 2025-11-17 205518" src="https://github.com/user-attachments/assets/b893d925-7cfd-4873-9c16-c29b8f7fa404" />

## 2. Implementasi CRUD di PHP
### A. koneksi.php (Koneksi ke database)
```
<?php
$host = "localhost";
$user = "root";
$pass = "";
$db = "latihan1";

$conn = mysqli_connect($host, $user, $pass, $db);
if (!$conn) {
    echo "Koneksi ke server gagal: " . mysqli_connect_error();
    exit;
}
// echo "Koneksi berhasil";
?>
```
<img width="1920" height="1080" alt="Cuplikan layar 2025-11-17 193635" src="https://github.com/user-attachments/assets/5c992134-7b42-4a01-849c-dec16d301445" />

Penjelasan:
- mysqli_connect() → membuat koneksi MySQL.
- Kalau koneksi gagal → program dihentikan.
- File ini wajib di-include di semua file CRUD lain.

### B. index.php
```
<?php
include('koneksi.php');

$sql = 'SELECT * FROM data_barang ORDER BY id_barang DESC';
$result = mysqli_query($conn, $sql);
?>
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<link href="style.css" rel="stylesheet" type="text/css" />
<title>Data Barang</title>
</head>
<body>
<div class="container">
  <h1>Data Barang</h1>
  <div class="main">
    <p><a class="btn" href="tambah.php">+ Tambah Barang</a></p>

    <table>
      <tr>
        <th>Gambar</th>
        <th>Nama Barang</th>
        <th>Kategori</th>
        <th>Harga Beli</th>
        <th>Harga Jual</th>
        <th>Stok</th>
        <th>Aksi</th>
      </tr>

      <?php if ($result && mysqli_num_rows($result) > 0): ?>
        <?php while ($row = mysqli_fetch_assoc($result)): ?>
        <tr>
          <td style="width:100px;">
            <img src="img/<?= htmlspecialchars($row['gambar']); ?>" 
                 alt="<?= htmlspecialchars($row['nama']); ?>" 
                 style="width:80px; height:auto;">
          </td>

          <td><?= htmlspecialchars($row['nama']); ?></td>
          <td><?= htmlspecialchars($row['kategori']); ?></td>
          <td><?= number_format($row['harga_beli'], 0, ',', '.'); ?></td>
          <td><?= number_format($row['harga_jual'], 0, ',', '.'); ?></td>
          <td><?= (int)$row['stok']; ?></td>

          <td>
            <a href="ubah.php?id=<?= $row['id_barang']; ?>">Ubah</a> | 
            <a href="hapus.php?id=<?= $row['id_barang']; ?>" 
               onclick="return confirm('Yakin ingin hapus data ini?')">Hapus</a>
          </td>
        </tr>
        <?php endwhile; ?>

      <?php else: ?>
        <tr><td colspan="7">Belum ada data</td></tr>
      <?php endif; ?>

    </table>
  </div>
</div>
</body>
</html>
```
<img width="1920" height="1080" alt="Cuplikan layar 2025-11-17 204319" src="https://github.com/user-attachments/assets/976c0c76-9989-416c-b70e-37a8a9e4c662" />

Penjelasan:
File index.php berfungsi sebagai halaman utama untuk menampilkan semua data barang dari database, termasuk gambar, kategori, harga beli/jual, stok, dan tombol aksi.

### C. tambah.php
```
<?php
error_reporting(E_ALL);
include_once 'koneksi.php';

if (isset($_POST['submit'])) {
    $nama = mysqli_real_escape_string($conn, trim($_POST['nama']));
    $kategori = mysqli_real_escape_string($conn, trim($_POST['kategori']));
    $harga_jual = (int)$_POST['harga_jual'];
    $harga_beli = (int)$_POST['harga_beli'];
    $stok = (int)$_POST['stok'];

    $gambar = null;
    if (isset($_FILES['file_gambar']) && $_FILES['file_gambar']['error'] == 0) {
        // pastikan folder gambar/ ada dan writable
        if (!is_dir('gambar')) mkdir('gambar', 0755, true);
        $filename = time() . '_' . preg_replace('/[^A-Za-z0-9\._-]/', '_', $_FILES['file_gambar']['name']);
        $destination = __DIR__ . '/gambar/' . $filename;
        if (move_uploaded_file($_FILES['file_gambar']['tmp_name'], $destination)) {
            $gambar = 'gambar/' . $filename;
        }
    }

    $sql = "INSERT INTO data_barang (nama, kategori, harga_jual, harga_beli, stok, gambar)
            VALUES ('{$nama}', '{$kategori}', '{$harga_jual}', '{$harga_beli}', '{$stok}', " . ($gambar ? "'{$gambar}'" : "NULL") . ")";
    mysqli_query($conn, $sql);
    header('Location: index.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<link href="style.css" rel="stylesheet" type="text/css" />
<title>Tambah Barang</title>
</head>
<body>
<div class="container">
  <h1>Tambah Barang</h1>
  <div class="main">
    <form method="post" action="tambah.php" enctype="multipart/form-data">
      <div class="input"><label>Nama Barang</label><input type="text" name="nama" required></div>
      <div class="input"><label>Kategori</label>
        <select name="kategori">
          <option value="Komputer">Komputer</option>
          <option value="Elektronik">Elektronik</option>
          <option value="Hand Phone">Hand Phone</option>
        </select>
      </div>
      <div class="input"><label>Harga Jual</label><input type="number" name="harga_jual" required></div>
      <div class="input"><label>Harga Beli</label><input type="number" name="harga_beli" required></div>
      <div class="input"><label>Stok</label><input type="number" name="stok" required></div>
      <div class="input"><label>File Gambar</label><input type="file" name="file_gambar" accept="image/*"></div>
      <div class="submit"><input type="submit" name="submit" value="Simpan"></div>
    </form>
  </div>
</div>
</body>
</html>
```
<img width="1920" height="1080" alt="Cuplikan layar 2025-11-17 204331" src="https://github.com/user-attachments/assets/b1257be2-33ad-4957-acdb-914a729ed4fa" />

Penjelasan:
- Mengisi formulir barang baru
- Mengupload gambar barang
- Menyimpan semua data ke tabel data_barang
Kode ini sudah aman dan rapi, menggunakan mysqli_real_escape_string + validasi upload.

### D. ubah.php
```
<?php
error_reporting(E_ALL);
include_once 'koneksi.php';

function is_select($var, $val) {
    return ($var == $val) ? 'selected="selected"' : '';
}

if (isset($_POST['submit'])) {
    $id = (int)$_POST['id'];
    $nama = mysqli_real_escape_string($conn, trim($_POST['nama']));
    $kategori = mysqli_real_escape_string($conn, trim($_POST['kategori']));
    $harga_jual = (int)$_POST['harga_jual'];
    $harga_beli = (int)$_POST['harga_beli'];
    $stok = (int)$_POST['stok'];

    // ambil data lama untuk gambarnya
    $old = mysqli_fetch_assoc(mysqli_query($conn, "SELECT gambar FROM data_barang WHERE id_barang = {$id}"));
    $oldGambar = $old ? $old['gambar'] : null;

    $gambar = $oldGambar;
    if (isset($_FILES['file_gambar']) && $_FILES['file_gambar']['error'] == 0) {
        if (!is_dir('gambar')) mkdir('gambar', 0755, true);
        $filename = time() . '_' . preg_replace('/[^A-Za-z0-9\._-]/', '_', $_FILES['file_gambar']['name']);
        $destination = __DIR__ . '/gambar/' . $filename;
        if (move_uploaded_file($_FILES['file_gambar']['tmp_name'], $destination)) {
            $gambar = 'gambar/' . $filename;
            // optional: hapus file lama jika ada
            if (!empty($oldGambar) && file_exists($oldGambar)) {
                @unlink($oldGambar);
            }
        }
    }

    $sql = "UPDATE data_barang SET
            nama = '{$nama}',
            kategori = '{$kategori}',
            harga_jual = '{$harga_jual}',
            harga_beli = '{$harga_beli}',
            stok = '{$stok}',
            gambar = " . ($gambar ? "'{$gambar}'" : "NULL") . "
            WHERE id_barang = '{$id}'";
    mysqli_query($conn, $sql);
    header('Location: index.php');
    exit;
}

$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
$sql = "SELECT * FROM data_barang WHERE id_barang = '{$id}'";
$result = mysqli_query($conn, $sql);
if (!$result || mysqli_num_rows($result) == 0) die('Error: Data tidak tersedia');
$data = mysqli_fetch_assoc($result);
?>
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<link href="style.css" rel="stylesheet" type="text/css" />
<title>Ubah Barang</title>
</head>
<body>
<div class="container">
  <h1>Ubah Barang</h1>
  <div class="main">
    <form method="post" action="ubah.php" enctype="multipart/form-data">
      <div class="input"><label>Nama Barang</label><input type="text" name="nama" value="<?= htmlspecialchars($data['nama']); ?>" required></div>
      <div class="input"><label>Kategori</label>
        <select name="kategori">
          <option <?= is_select('Komputer', $data['kategori']); ?> value="Komputer">Komputer</option>
          <option <?= is_select('Elektronik', $data['kategori']); ?> value="Elektronik">Elektronik</option>
          <option <?= is_select('Hand Phone', $data['kategori']); ?> value="Hand Phone">Hand Phone</option>
        </select>
      </div>
      <div class="input"><label>Harga Jual</label><input type="number" name="harga_jual" value="<?= (int)$data['harga_jual']; ?>" required></div>
      <div class="input"><label>Harga Beli</label><input type="number" name="harga_beli" value="<?= (int)$data['harga_beli']; ?>" required></div>
      <div class="input"><label>Stok</label><input type="number" name="stok" value="<?= (int)$data['stok']; ?>" required></div>
      <div class="input"><label>Gambar Saat Ini</label>
        <?php if(!empty($data['gambar']) && file_exists($data['gambar'])): ?>
          <div><img src="<?= htmlspecialchars($data['gambar']); ?>" style="max-width:120px"></div>
        <?php else: ?>
          <div>-</div>
        <?php endif; ?>
      </div>
      <div class="input"><label>Ganti File Gambar (opsional)</label><input type="file" name="file_gambar" accept="image/*"></div>
      <div class="submit">
        <input type="hidden" name="id" value="<?= (int)$data['id_barang']; ?>">
        <input type="submit" name="submit" value="Simpan">
      </div>
    </form>
  </div>
</div>
</body>
</html>
```
<img width="1920" height="1080" alt="Cuplikan layar 2025-11-17 204345" src="https://github.com/user-attachments/assets/3747e7ed-51fa-4a8c-abc7-0d2d3431f37c" />

Penjelasan:
File ubah.php digunakan untuk mengambil data barang, menampilkan form edit, dan meng-update data termasuk upload gambar baru.

### D. hapus.php
```
<?php
include_once 'koneksi.php';
$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;

// ambil nama file gambar untuk dihapus dari disk (opsional)
$res = mysqli_query($conn, "SELECT gambar FROM data_barang WHERE id_barang = {$id}");
if ($res && mysqli_num_rows($res) > 0) {
    $row = mysqli_fetch_assoc($res);
    if (!empty($row['gambar']) && file_exists($row['gambar'])) {
        @unlink($row['gambar']);
    }
}

$sql = "DELETE FROM data_barang WHERE id_barang = '{$id}'";
mysqli_query($conn, $sql);
header('Location: index.php');
exit;
?>
```
<img width="1920" height="1080" alt="Cuplikan layar 2025-11-17 204400" src="https://github.com/user-attachments/assets/6808758e-62c0-415d-bb48-16f0e7bfba24" />

Penjelasan:
File hapus.php berfungsi untuk menghapus data barang dari database, sekaligus menghapus file gambar yang terkait (jika ada).

### E. Menambahkan style.css (supaya tampilan rapih)
```
/* Reset */
body, h1, table, tr, td, th, div, form, input, select, label {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: Arial, sans-serif;
}

/* Container utama */
.container {
    width: 80%;
    margin: 20px auto;
}

/* Judul */
h1 {
    text-align: center;
    margin-bottom: 20px;
}

/* Tabel */
table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
}

table th {
    background: #db3477;
    color: #fff;
    padding: 10px;
    text-align: left;
    border: 1px solid #ddd;
}

table td {
    padding: 10px;
    border: 1px solid #ddd;
}

/* Gambar barang (termasuk PNG) */
table img {
    width: 120px;            /* ukuran gambar */
    height: auto;
    object-fit: contain;     /* supaya PNG tidak melebar */
    background: transparent; /* transparansi PNG tetap bagus */
    padding: 5px;
    border-radius: 8px;
    border: 1px solid #ddd;
}

/* Form */
.main form {
    width: 100%;
    padding: 20px;
    border: 1px solid #ddd;
    background: #f7f7f7;
    border-radius: 8px;
}

.input {
    margin-bottom: 15px;
}

.input label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
}

.input input[type="text"],
.input input[type="file"],
.input select {
    width: 100%;
    padding: 8px;
    border: 1px solid #aaa;
    border-radius: 5px;
}

/* Tombol submit */
.submit input {
    background: #db3477;
    color: #fff;
    padding: 10px 15px;
    border: none;
    cursor: pointer;
    margin-top: 10px;
    border-radius: 5px;
    font-size: 15px;
}

.submit input:hover {
    background: #db3477;
}
.foto-barang {
    width: 100px;
    height: auto;
    object-fit: contain;
    border: 1px solid #ccc;
    padding: 5px;
    border-radius: 8px;
    background: #fff;
}
```
