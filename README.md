$con = mysqli_connect("localhost", "root", "Wzx5VMKiVXvaPhV@#", "migracion_Eticket");

if (!$con) {
    die("Kết nối thất bại: " . mysqli_connect_error());
}

echo "<h2>🔍 Kiểm tra quyền và cấu hình MySQL</h2>";

// 1. Check GRANTS (quyền của user)
echo "<h3>1️⃣ Quyền của user hiện tại:</h3>";
$res = mysqli_query($con, "SHOW GRANTS FOR CURRENT_USER");
if ($res) {
    while ($row = mysqli_fetch_row($res)) {
        echo htmlspecialchars($row[0]) . "<br>";
    }
} else {
    echo "Lỗi: " . mysqli_error($con);
}

// 2. Check secure_file_priv
echo "<h3>2️⃣ secure_file_priv:</h3>";
$res = mysqli_query($con, "SHOW VARIABLES LIKE 'secure_file_priv'");
if ($res) {
    $row = mysqli_fetch_assoc($res);
    echo "secure_file_priv = " . htmlspecialchars($row['Value']) . "<br>";
} else {
    echo "Lỗi: " . mysqli_error($con);
}

// 3. Check user OS chạy MySQL
echo "<h3>3️⃣ User hệ điều hành chạy MySQL:</h3>";
$res = mysqli_query($con, "SELECT USER() AS user_mysql, CURRENT_USER() AS current_mysql, @@system_user AS system_user, @@hostname AS hostname");
if ($res) {
    $row = mysqli_fetch_assoc($res);
    echo "USER() = " . htmlspecialchars($row['user_mysql']) . "<br>";
    echo "CURRENT_USER() = " . htmlspecialchars($row['current_mysql']) . "<br>";
    echo "System User (Linux) = " . htmlspecialchars($row['system_user']) . "<br>";
    echo "Hostname = " . htmlspecialchars($row['hostname']) . "<br>";
} else {
    echo "Lỗi: " . mysqli_error($con);
}

mysqli_close($con);
?>
