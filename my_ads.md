```php
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.html");
    exit;
}

include 'config.php';

try {
    $user_id = $_SESSION['user_id'];
    $stmt = $conn->prepare("SELECT ads.*, 
                          (SELECT image_path FROM ad_images WHERE ad_id=ads.id LIMIT 1) AS image_path, 
                          categories.name AS category_name 
                          FROM ads 
                          JOIN categories ON ads.category_id=categories.id 
                          WHERE ads.user_id=:user_id 
                          ORDER BY ads.created_at DESC");
    $stmt->bindParam(':user_id', $user_id, PDO::PARAM_INT);
    $stmt->execute();
    $ads = $stmt->fetchAll(PDO::FETCH_ASSOC);
} catch(PDOException $e) {
    die("Error: " . $e->getMessage());
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Iklan Saya - OLX Clone</title>
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
<section class="product-section">
    <div class="container">
        <h2>Iklan Saya</h2>
        <div class="product-grid">
            <?php foreach($ads as $row): ?>
            <div class="product-card">
                <a href="detail.php?id=<?= $row['id'] ?>">
                    <img src="<?= $row['image_path'] ? 'uploads/'.$row['image_path'] : 'assets/images/noimage.png' ?>" alt="Produk">
                    <h3><?= htmlspecialchars($row['title']) ?></h3>
                    <p class="price">Rp <?= number_format($row['price'],0,',','.') ?></p>
                    <p class="location"><?= htmlspecialchars($row['location']) ?></p>
                </a>
                <div style="margin-top:10px;">
                    <a href="edit_ad.php?id=<?= $row['id'] ?>" class="btn" style="padding:5px 12px;font-size:0.95em;">Edit</a>
                    <a href="delete_ad.php?id=<?= $row['id'] ?>" class="btn btn-outline" style="padding:5px 12px;font-size:0.95em;" onclick="return confirm('Yakin ingin menghapus iklan ini?')">Hapus</a>
                </div>
            </div>
            <?php endforeach; ?>
        </div>
    </div>
</section>
</body>
</html>
```
