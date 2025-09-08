$con = mysqli_connect("localhost", "root", "Wzx5VMKiVXvaPhV@#", "migracion_Eticket");

// Kiểm tra kết nối
if (!$con) {
    die("Kết nối thất bại: " . mysqli_connect_error());
}

// Truy vấn version
$sql = "SELECT VERSION() AS mysql_version";
$result = mysqli_query($con, $sql);

if ($result) {
    $row = mysqli_fetch_assoc($result);
    echo "MySQL Version: " . $row['mysql_version'];
} else {
    echo "Lỗi khi thực thi truy vấn: " . mysqli_error($con);
}

// Đóng kết nối
mysqli_close($con);
?>
