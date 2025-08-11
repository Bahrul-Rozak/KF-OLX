```php
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    echo "<script>alert('Silakan login dulu!');window.location='login.html';</script>";
    exit;
}

include 'config.php';

try {
    // Mengambil data kategori menggunakan PDO
    $stmt = $conn->query("SELECT * FROM categories");
    $categories = $stmt->fetchAll(PDO::FETCH_ASSOC);
} catch(PDOException $e) {
    die("Error: " . $e->getMessage());
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pasang Iklan - OLX Clone</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
    <header>
        <div class="container header-flex">
            <div class="logo">OLX Clone</div>
            <nav>
                <a href="index.php">Beranda</a>
                <a href="post.php">Pasang Iklan</a>
                <a href="logout.php" class="btn">Logout</a>
            </nav>
        </div>
    </header>
    <section class="form-section">
        <div class="container form-container">
            <h2>Pasang Iklan Baru</h2>
            <form action="post_process.php" method="POST" enctype="multipart/form-data">
                <label>Judul Iklan</label>
                <input type="text" name="title" placeholder="Judul iklan" required>
                <label>Kategori</label>
                <select name="category_id" required>
                    <option value="">Pilih Kategori</option>
                    <?php foreach($categories as $cat): ?>
                        <option value="<?= $cat['id'] ?>"><?= htmlspecialchars($cat['name']) ?></option>
                    <?php endforeach; ?>
                </select>
                <label>Harga</label>
                <input type="number" name="price" placeholder="Harga" required>
                <label>Lokasi</label>
                <input type="text" name="location" placeholder="Lokasi" required>
                <label>Deskripsi</label>
                <textarea name="description" placeholder="Deskripsi iklan" required></textarea>
                <label>Foto Produk</label>
                <input type="file" name="image" accept="image/*" required>
                <button type="submit">Pasang Iklan</button>
            </form>
        </div>
    </section>
</body>
</html>
```
