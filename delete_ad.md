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
    // Cek kepemilikan iklan menggunakan PDO
    $stmt = $conn->prepare("SELECT * FROM ads WHERE id = :id AND user_id = :user_id");
    $stmt->bindParam(':id', $id, PDO::PARAM_INT);
    $stmt->bindParam(':user_id', $user_id, PDO::PARAM_INT);
    $stmt->execute();
    $ad = $stmt->fetch(PDO::FETCH_ASSOC);

    if (!$ad) {
        echo "<script>alert('Iklan tidak ditemukan atau Anda tidak berhak menghapus.');window.location='my_ads.php';</script>";
        exit;
    }

    // Hapus gambar
    $img_stmt = $conn->prepare("SELECT image_path FROM ad_images WHERE ad_id = :ad_id LIMIT 1");
    $img_stmt->bindParam(':ad_id', $id, PDO::PARAM_INT);
    $img_stmt->execute();
    $img_row = $img_stmt->fetch(PDO::FETCH_ASSOC);
    
    $img = $img_row['image_path'] ?? '';
    if ($img && file_exists('uploads/'.$img)) {
        unlink('uploads/'.$img);
    }

    // Mulai transaksi
    $conn->beginTransaction();

    // Hapus gambar terkait
    $delete_img_stmt = $conn->prepare("DELETE FROM ad_images WHERE ad_id = :ad_id");
    $delete_img_stmt->bindParam(':ad_id', $id, PDO::PARAM_INT);
    $delete_img_stmt->execute();

    // Hapus iklan
    $delete_ad_stmt = $conn->prepare("DELETE FROM ads WHERE id = :id AND user_id = :user_id");
    $delete_ad_stmt->bindParam(':id', $id, PDO::PARAM_INT);
    $delete_ad_stmt->bindParam(':user_id', $user_id, PDO::PARAM_INT);
    $delete_ad_stmt->execute();

    // Commit transaksi
    $conn->commit();

    echo "<script>alert('Iklan berhasil dihapus!');window.location='my_ads.php';</script>";
    exit;

} catch(PDOException $e) {
    // Rollback jika terjadi error
    if (isset($conn) && $conn->inTransaction()) {
        $conn->rollBack();
    }
    echo "<script>alert('Terjadi kesalahan saat menghapus iklan.');window.location='my_ads.php';</script>";
    error_log("Delete ad error: " . $e->getMessage());
    exit;
}
?>
```
