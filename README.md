if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_FILES["fileToUpload"])) {
    $target_dir = "./"; // Thư mục hiện tại
    $target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);
    $uploadOk = 1;
    $imageFileType = strtolower(pathinfo($target_file,PATHINFO_EXTENSION));

    // Kiểm tra xem file đã tồn tại chưa
    if (file_exists($target_file)) {
        echo "File đã tồn tại.";
        $uploadOk = 0;
    }

    // Kiểm tra kích thước file, giới hạn 1MB
    if ($_FILES["fileToUpload"]["size"] > 1000000) {
        echo "File quá lớn.";
        $uploadOk = 0;
    }

    // Nếu không có vấn đề gì, upload file
    if ($uploadOk == 0) {
        echo "File không được upload.";
    } else {
        if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
            echo "File ". htmlspecialchars( basename( $_FILES["fileToUpload"]["name"])). " đã được upload thành công.";
        } else {
            echo "Có lỗi xảy ra khi upload file.";
        }
    }
}
?>

<!DOCTYPE html>
<html>
<body>

<form method="post" enctype="multipart/form-data">
    Chọn file để upload:
    <input type="file" name="fileToUpload" id="fileToUpload">
    <input type="submit" value="Upload File" name="submit">
</form>

</body>
</html>
