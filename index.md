```php
<?php
include 'config.php';
session_start();

try {
    // Ambil lokasi unik dari ads
    $locations = [];
    $loc_stmt = $conn->query("SELECT DISTINCT location FROM ads WHERE location IS NOT NULL AND location != '' ORDER BY location ASC");
    $locations = $loc_stmt->fetchAll(PDO::FETCH_COLUMN);

    $where = [];
    $params = [];

    if (isset($_GET['title']) && $_GET['title'] != '') {
        $where[] = "ads.title LIKE :title";
        $params[':title'] = '%' . $_GET['title'] . '%';
    }
    if (isset($_GET['location']) && $_GET['location'] != '') {
        $where[] = "ads.location LIKE :location";
        $params[':location'] = '%' . $_GET['location'] . '%';
    }
    if (isset($_GET['category_id']) && $_GET['category_id'] != '') {
        $where[] = "ads.category_id = :category_id";
        $params[':category_id'] = intval($_GET['category_id']);
    }

    $where_sql = $where ? 'WHERE ' . implode(' AND ', $where) : '';

    $sql = "SELECT ads.*, categories.name AS category_name, 
            (SELECT image_path FROM ad_images WHERE ad_id = ads.id LIMIT 1) AS image_path
            FROM ads 
            JOIN categories ON ads.category_id = categories.id
            $where_sql
            ORDER BY ads.created_at DESC
            LIMIT 12";

    $stmt = $conn->prepare($sql);
    foreach ($params as $key => &$val) {
        if ($key === ':category_id') {
            $stmt->bindParam($key, $val, PDO::PARAM_INT);
        } else {
            $stmt->bindParam($key, $val);
        }
    }
    $stmt->execute();
    $ads = $stmt->fetchAll(PDO::FETCH_ASSOC);

    // Ambil kategori untuk menu
    $cat_stmt = $conn->query("SELECT * FROM categories");
    $categories = $cat_stmt->fetchAll(PDO::FETCH_ASSOC);

    // Ambil nama kategori jika ada filter
    $cat_name = '';
    if (isset($_GET['category_id']) && $_GET['category_id'] != '') {
        $cat_id = intval($_GET['category_id']);
        $cat_stmt = $conn->prepare("SELECT name FROM categories WHERE id = :id");
        $cat_stmt->bindParam(':id', $cat_id, PDO::PARAM_INT);
        $cat_stmt->execute();
        $cat_result = $cat_stmt->fetch(PDO::FETCH_ASSOC);
        $cat_name = $cat_result['name'] ?? '';
    }
} catch(PDOException $e) {
    die("Error: " . $e->getMessage());
}
?>

<!DOCTYPE html>
<html lang="id">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OLX Clone - Jual Beli Mudah</title>
    <link rel="stylesheet" href="assets/css/style.css">
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

    <!-- Search Bar -->
    <section class="search-section">
        <div class="container">
            <form method="GET" action="index.php" style="display: flex; gap: 10px; justify-content: center; flex-wrap: wrap;">
                <input type="text" name="title" value="<?= isset($_GET['title']) ? htmlspecialchars($_GET['title']) : '' ?>" placeholder="Judul barang...">
                <input type="text" name="location" value="<?= isset($_GET['location']) ? htmlspecialchars($_GET['location']) : '' ?>" placeholder="Lokasi...">
                <select name="location">
                    <option value="">Semua Lokasi</option>
                    <?php foreach ($locations as $loc): ?>
                        <option value="<?= htmlspecialchars($loc) ?>" <?= (isset($_GET['location']) && $_GET['location'] == $loc) ? 'selected' : '' ?>>
                            <?= htmlspecialchars($loc) ?>
                        </option>
                    <?php endforeach; ?>
                </select>
                <button type="submit">Cari</button>
            </form>
        </div>
    </section>

    <!-- Kategori -->
    <section class="category-section">
        <div class="container">
            <h2>Kategori Populer</h2>
            <div class="categories">
                <?php foreach ($categories as $cat): ?>
                    <a href="index.php?category_id=<?= $cat['id'] ?>"
                        class="category-item<?= (isset($_GET['category_id']) && $_GET['category_id'] == $cat['id']) ? ' active' : '' ?>"
                        style="text-align:center; text-decoration:none;">
                        <?php
                        $icon_path = 'assets/images/' . $cat['icon'];
                        if (!$cat['icon'] || !file_exists($icon_path)) {
                            $placeholder = 'https://placehold.co/48x48/2196f3/fff?text=' . urlencode(substr($cat['name'], 0, 2));
                            echo '<img src="' . $placeholder . '" alt="' . htmlspecialchars($cat['name']) . '">';
                        } else {
                            echo '<img src="' . $icon_path . '" alt="' . htmlspecialchars($cat['name']) . '">';
                        }
                        ?>
                        <span><?= htmlspecialchars($cat['name']) ?></span>
                    </a>
                <?php endforeach; ?>
            </div>
        </div>
    </section>

    <!-- Daftar Produk/Iklan -->
    <section class="product-section">
        <div class="container">
            <h2>Daftar Iklan Terbaru</h2>

            <?php
            $info = [];
            if (isset($_GET['title']) && $_GET['title'] != '') $info[] = "Judul: <b>" . htmlspecialchars($_GET['title']) . "</b>";
            if (isset($_GET['location']) && $_GET['location'] != '') $info[] = "Lokasi: <b>" . htmlspecialchars($_GET['location']) . "</b>";
            if (isset($_GET['category_id']) && $_GET['category_id'] != '') {
                $info[] = "Kategori: <b>" . htmlspecialchars($cat_name) . "</b>";
            }
            if ($info) echo "<p>Filter: " . implode(', ', $info) . "</p>";
            ?>

            <?php
            if (isset($_GET['category_id']) && $_GET['category_id'] != '') {
                echo "<p>Menampilkan iklan untuk kategori: <b>" . htmlspecialchars($cat_name) . "</b></p>";
            }
            ?>

            <div class="product-grid">
                <?php foreach ($ads as $row): ?>
                    <div class="product-card">
                        <a href="detail.php?id=<?= $row['id'] ?>">
                            <img src="<?= $row['image_path'] ? 'uploads/' . $row['image_path'] : 'assets/images/noimage.png' ?>" alt="Produk">
                            <h3><?= htmlspecialchars($row['title']) ?></h3>
                            <p class="price">Rp <?= number_format($row['price'], 0, ',', '.') ?></p>
                            <p class="location"><?= htmlspecialchars($row['location']) ?></p>
                        </a>
                    </div>
                <?php endforeach; ?>
            </div>
        </div>
    </section>

    <!-- Footer -->
    <footer>
        <div class="container">
            <p>&copy; 2025 OLX Clone. All rights reserved.</p>
        </div>
    </footer>
</body>

</html>
```
