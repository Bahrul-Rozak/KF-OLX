```php
<?php
include 'config.php';
session_start();

$id = isset($_GET['id']) ? intval($_GET['id']) : 0;
if ($id <= 0) {
    echo "<script>alert('Iklan tidak ditemukan!');window.location='index.php';</script>";
    exit;
}

try {
    // Ambil data iklan menggunakan PDO
    $sql = "SELECT ads.*, categories.name AS category_name, users.name AS user_name, users.whatsapp
            FROM ads
            JOIN categories ON ads.category_id = categories.id
            JOIN users ON ads.user_id = users.id
            WHERE ads.id = :id";
    $stmt = $conn->prepare($sql);
    $stmt->bindParam(':id', $id, PDO::PARAM_INT);
    $stmt->execute();
    $ad = $stmt->fetch(PDO::FETCH_ASSOC);

    if (!$ad) {
        echo "<script>alert('Iklan tidak ditemukan!');window.location='index.php';</script>";
        exit;
    }

    // Ambil gambar iklan menggunakan PDO
    $img_stmt = $conn->prepare("SELECT image_path FROM ad_images WHERE ad_id = :ad_id LIMIT 1");
    $img_stmt->bindParam(':ad_id', $id, PDO::PARAM_INT);
    $img_stmt->execute();
    $img_row = $img_stmt->fetch(PDO::FETCH_ASSOC);
    $image_path = $img_row ? 'uploads/' . $img_row['image_path'] : 'assets/images/noimage.png';
} catch(PDOException $e) {
    die("Error: " . $e->getMessage());
}
?>
<!DOCTYPE html>
<html lang="id">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?= htmlspecialchars($ad['title']) ?> - OLX Clone</title>
    <link rel="stylesheet" href="assets/css/style.css">
    <style>
        /* New CSS styles to enhance the OLX look */
        :root {
            --olx-orange: #e67e22;
            --olx-dark: #2c3e50;
            --olx-light: #ecf0f1;
            --olx-gray: #95a5a6;
            --olx-success: #27ae60;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f5f5f5;
            color: #333;
        }

        header {
            background-color: white;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
            padding: 15px 0;
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .header-flex {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .logo {
            font-size: 24px;
            font-weight: bold;
            color: var(--olx-orange);
        }

        nav a {
            margin-left: 20px;
            text-decoration: none;
            color: var(--olx-dark);
            font-weight: 500;
        }

        nav a:hover {
            color: var(--olx-orange);
        }

        .btn {
            background-color: var(--olx-orange);
            color: white;
            padding: 8px 15px;
            border-radius: 4px;
            transition: background-color 0.3s;
        }

        .btn:hover {
            background-color: #d35400;
            color: white;
        }

        .container {
            width: 90%;
            max-width: 1200px;
            margin: 0 auto;
        }

        .detail-section {
            padding: 30px 0;
        }

        .detail-container {
            display: flex;
            gap: 30px;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
        }

        .detail-image {
            flex: 1;
            max-width: 500px;
        }

        .detail-image img {
            width: 100%;
            height: auto;
            border-radius: 8px;
            object-fit: cover;
            max-height: 500px;
        }

        .detail-info {
            flex: 1;
        }

        .detail-info h2 {
            font-size: 28px;
            margin-top: 0;
            color: var(--olx-dark);
        }

        .price {
            font-size: 24px;
            font-weight: bold;
            color: var(--olx-orange);
            margin: 15px 0;
        }

        .location {
            color: var(--olx-gray);
            margin: 15px 0;
            font-size: 14px;
        }

        .desc {
            line-height: 1.6;
            margin: 25px 0;
            color: #555;
        }

        @media (max-width: 768px) {
            .detail-container {
                flex-direction: column;
            }

            .detail-image {
                max-width: 100%;
            }
        }
    </style>
</head>

<body>
    <!-- Header -->
    <header>
        <div class="container header-flex">
            <div class="logo">OLX Clone</div>
            <nav>
                <a href="index.php">Beranda</a>
                <a href="post.php">Pasang Iklan</a>
                <a href="my_ads.php">Iklan Saya</a>
                <?php if (isset($_SESSION['user_id'])): ?>
                    <span>Halo, <?= htmlspecialchars($_SESSION['user_name']) ?></span>
                    <a href="logout.php" class="btn">Logout</a>
                <?php else: ?>
                    <a href="login.html" class="btn">Login</a>
                    <a href="register.html" class="btn btn-outline">Register</a>
                <?php endif; ?>
            </nav>
        </div>
    </header>
    <section class="detail-section">
        <div class="container detail-container">
            <div class="detail-image">
                <img src="<?= $image_path ?>" alt="Produk">
            </div>
            <div class="detail-info">
                <h2><?= htmlspecialchars($ad['title']) ?></h2>
                <p class="price">Rp <?= number_format($ad['price'], 0, ',', '.') ?></p>
                <p class="location"><?= htmlspecialchars($ad['location']) ?> | Kategori: <?= htmlspecialchars($ad['category_name']) ?></p>
                <p class="desc"><?= nl2br(htmlspecialchars($ad['description'])) ?></p>
                <p>Penjual: <?= htmlspecialchars($ad['user_name']) ?></p>
                <?php if (isset($_SESSION['user_id']) && $_SESSION['user_id'] != $ad['user_id']): // Tambahkan kondisi ini 
                ?>
                    <?php if ($ad['whatsapp']): ?>
                        <a href="https://wa.me/62<?= ltrim(preg_replace('/[^0-9]/', '', $ad['whatsapp']), '0') ?>?text=Halo%20saya%20tertarik%20dengan%20iklan%20Anda%20di%20OLX%20Clone"
                            target="_blank"
                            class="btn"
                            style="background:#25D366;margin-top:10px;">
                            Hubungi via WhatsApp
                        </a>
                    <?php endif; ?>
                <?php endif; ?>
            </div>
        </div>
    </section>
</body>

</html>
```
