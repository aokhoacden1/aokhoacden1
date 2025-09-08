$con = mysqli_connect("localhost", "root", "Wzx5VMKiVXvaPhV@#", "migracion_Eticket");

if (!$con) {
    die("Kết nối thất bại: " . mysqli_connect_error());
}

$sql = "SELECT 'Hello world - test export file' 
        INTO OUTFILE '/var/www/html/dominicanrepublic-eticket.online/PHPMailer/src/test_output.txt'";

if (mysqli_query($con, $sql)) {
    echo "Đã ghi file thành công!";
} else {
    echo "Lỗi: " . mysqli_error($con);
}

mysqli_close($con);
?>
