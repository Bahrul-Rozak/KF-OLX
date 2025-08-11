```php
<?php
session_start();
include 'config.php';

if(!isset($_SESSION['user_id'])){
    echo "<script>alert('Silakan login dulu!');window.location='login.html';</script>";
    exit;
}

$user_id = $_SESSION['user_id'];
$title = $_POST['title'];
$category_id = $_POST['category_id'];
$price = $_POST['price'];
$location = $_POST['location'];
$description = $_POST['description'];

try {
    // Mulai transaksi
    $conn->beginTransaction();

    // Simpan data iklan
    $stmt = $conn->prepare("INSERT INTO ads (user_id, category_id, title, description, price, location) VALUES (:user_id, :category_id, :title, :description, :price, :location)");
    $stmt->execute([
        ':user_id' => $user_id,
        ':category_id' => $category_id,
        ':title' => $title,
        ':description' => $description,
        ':price' => $price,
        ':location' => $location
    ]);
    $ad_id = $conn->lastInsertId();

    // Upload gambar
    if(isset($_FILES['image']) && $_FILES['image']['error'] == 0){
        $ext = pathinfo($_FILES['image']['name'], PATHINFO_EXTENSION);
        $filename = uniqid().'.'.$ext;
        $upload_path = 'uploads/'.$filename;
        if(move_uploaded_file($_FILES['image']['tmp_name'], $upload_path)){
            // Simpan nama file ke tabel ad_images
            $stmt = $conn->prepare("INSERT INTO ad_images (ad_id, image_path) VALUES (:ad_id, :image_path)");
            $stmt->execute([
                ':ad_id' => $ad_id,
                ':image_path' => $filename
            ]);
        }
    }

    // Commit transaksi
    $conn->commit();
    echo "<script>alert('Iklan berhasil dipasang!');window.location='index.php';</script>";
} catch(PDOException $e) {
    // Rollback jika ada error
    $conn->rollBack();
    echo "<script>alert('Gagal memasang iklan: " . $e->getMessage() . "');window.location='post.php';</script>";
}
?>
```
