<?php
if(isset($_POST['submit'])){
    $targetDir = "uploads/"; // Thư mục đích để lưu trữ tệp tin được tải lên
    $targetFile = $targetDir . basename($_FILES["file"]["name"]); // Đường dẫn đầy đủ đến tệp tin được tải lên

    // Kiểm tra xem tệp tin đã tồn tại hay chưa
    if(file_exists($targetFile)){
        echo "Tệp tin đã tồn tại.";
    } else {
        // Di chuyển tệp tin tải lên vào thư mục đích
        if(move_uploaded_file($_FILES["file"]["tmp_name"], $targetFile)){
            echo "Tệp tin đã được tải lên thành công.";
        } else {
            echo "Có lỗi xảy ra khi tải lên tệp tin.";
        }
    }
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Upload File</title>
</head>
<body>
    <form action="" method="POST" enctype="multipart/form-data">
        <input type="file" name="file" />
        <input type="submit" name="submit" value="Upload" />
    </form>
</body>
</html>
