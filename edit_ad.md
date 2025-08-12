```php
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.html");
    exit;
}

include 'config.php';

$user_id = $_SESSION['user_id'];
$id = intval($_GET['id'] ?? 0);

try {
    // Ambil data iklan menggunakan PDO
    $stmt = $conn->prepare("SELECT * FROM ads WHERE id = :id AND user_id = :user_id");
    $stmt->bindParam(':id', $id, PDO::PARAM_INT);
    $stmt->bindParam(':user_id', $user_id, PDO::PARAM_INT);
    $stmt->execute();
    $ad = $stmt->fetch(PDO::FETCH_ASSOC);

    if (!$ad) {
        echo "<script>alert('Iklan tidak ditemukan atau Anda tidak berhak mengedit.');window.location='my_ads.php';</script>";
        exit;
    }

    // Proses update jika form disubmit
    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
        $title = $_POST['title'];
        $category_id = $_POST['category_id'];
        $price = $_POST['price'];
        $location = $_POST['location'];
        $description = $_POST['description'];

        // Mulai transaksi
        $conn->beginTransaction();

        try {
            // Update data iklan
            $update_stmt = $conn->prepare("UPDATE ads SET title = :title, category_id = :category_id, 
                                         price = :price, location = :location, description = :description 
                                         WHERE id = :id AND user_id = :user_id");
            
            $update_stmt->bindParam(':title', $title);
            $update_stmt->bindParam(':category_id', $category_id, PDO::PARAM_INT);
            $update_stmt->bindParam(':price', $price);
            $update_stmt->bindParam(':location', $location);
            $update_stmt->bindParam(':description', $description);
            $update_stmt->bindParam(':id', $id, PDO::PARAM_INT);
            $update_stmt->bindParam(':user_id', $user_id, PDO::PARAM_INT);
            $update_stmt->execute();

            // Jika ada upload gambar baru
            if(isset($_FILES['image']) && $_FILES['image']['error'] == 0) {
                $ext = pathinfo($_FILES['image']['name'], PATHINFO_EXTENSION);
                $filename = uniqid().'.'.$ext;
                $upload_path = 'uploads/'.$filename;
                
                if(move_uploaded_file($_FILES['image']['tmp_name'], $upload_path)) {
                    // Hapus gambar lama (opsional)
                    $img_stmt = $conn->prepare("SELECT image_path FROM ad_images WHERE ad_id = :ad_id LIMIT 1");
                    $img_stmt->bindParam(':ad_id', $id, PDO::PARAM_INT);
                    $img_stmt->execute();
                    $old_img = $img_stmt->fetch(PDO::FETCH_COLUMN);
                    
                    if($old_img && file_exists('uploads/'.$old_img)) {
                        unlink('uploads/'.$old_img);
                    }
                    
                    // Update gambar
                    $update_img_stmt = $conn->prepare("UPDATE ad_images SET image_path = :image_path WHERE ad_id = :ad_id");
                    $update_img_stmt->bindParam(':image_path', $filename);
                    $update_img_stmt->bindParam(':ad_id', $id, PDO::PARAM_INT);
                    $update_img_stmt->execute();
                }
            }

            // Commit transaksi
            $conn->commit();
            echo "<script>alert('Iklan berhasil diupdate!');window.location='my_ads.php';</script>";
            exit;

        } catch(PDOException $e) {
            // Rollback jika terjadi error
            $conn->rollBack();
            echo "<script>alert('Gagal mengupdate iklan: ".addslashes($e->getMessage())."');</script>";
        }
    }

    // Ambil kategori
    $cats_stmt = $conn->query("SELECT * FROM categories");
    $categories = $cats_stmt->fetchAll(PDO::FETCH_ASSOC);

} catch(PDOException $e) {
    echo "<script>alert('Terjadi kesalahan: ".addslashes($e->getMessage())."');</script>";
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Edit Iklan - OLX Clone</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
<header>
    <div class="container header-flex">
        <div class="logo">OLX Clone</div>
        <nav>
            <a href="index.php">Beranda</a>
            <a href="post.php">Pasang Iklan</a>
            <a href="my_ads.php">Iklan Saya</a>
            <a href="logout.php" class="btn">Logout</a>
        </nav>
    </div>
</header>
<section class="form-section">
    <div class="container form-container">
        <h2>Edit Iklan</h2>
        <form action="" method="POST" enctype="multipart/form-data">
            <label>Judul Iklan</label>
            <input type="text" name="title" value="<?= htmlspecialchars($ad['title']) ?>" required>
            <label>Kategori</label>
            <select name="category_id" required>
                <option value="">Pilih Kategori</option>
                <?php foreach($categories as $cat): ?>
                    <option value="<?= $cat['id'] ?>" <?= $ad['category_id'] == $cat['id'] ? 'selected' : '' ?>>
                        <?= htmlspecialchars($cat['name']) ?>
                    </option>
                <?php endforeach; ?>
            </select>
            <label>Harga</label>
            <input type="number" name="price" value="<?= htmlspecialchars($ad['price']) ?>" required>
            <label>Lokasi</label>
            <input type="text" name="location" value="<?= htmlspecialchars($ad['location']) ?>" required>
            <label>Deskripsi</label>
            <textarea name="description" required><?= htmlspecialchars($ad['description']) ?></textarea>
            <label>Ganti Foto Produk (opsional)</label>
            <input type="file" name="image" accept="image/*">
            <button type="submit">Update Iklan</button>
        </form>
    </div>
</section>
</body>
</html>
```
