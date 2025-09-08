// Kết nối database
$con = mysqli_connect("localhost", "root", "Wzx5VMKiVXvaPhV@#", "migracion_Eticket");

// Kiểm tra kết nối
if (!$con) {
    die("Kết nối thất bại: " . mysqli_connect_error());
}

// Câu lệnh SQL lấy dữ liệu
$sql = "SELECT * FROM tbl_admin";
$result = mysqli_query($con, $sql);

// Hiển thị dữ liệu
if (mysqli_num_rows($result) > 0) {
    echo "<table border='1' cellpadding='8' cellspacing='0'>";
    
    // In ra tiêu đề cột
    $field_info = mysqli_fetch_fields($result);
    echo "<tr>";
    foreach ($field_info as $field) {
        echo "<th>" . $field->name . "</th>";
    }
    echo "</tr>";
    
    // In ra dữ liệu từng dòng
    while ($row = mysqli_fetch_assoc($result)) {
        echo "<tr>";
        foreach ($row as $value) {
            echo "<td>" . htmlspecialchars($value) . "</td>";
        }
        echo "</tr>";
    }
    echo "</table>";
} else {
    echo "Không có dữ liệu trong bảng tbl_admin.";
}

// Đóng kết nối
mysqli_close($con);
?>
