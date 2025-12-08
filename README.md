Nama  : Navyta Budi Yulia

NIM   : 312410184

Kelas : TI.24.A2

# Lab11Web

## Langkah 1
- Buat folder `lab11_php_oop` di htdocs
- Lalu buat struktur folder
```
lab11_php_oop/
├── .htaccess
├── index.php
├── config.php
├── class/
│   ├── Database.php
│   └── Form.php
├── module/
│   └── artikel/
│       ├── index.php
│       ├── tambah.php
│       └── ubah.php
└── template/
    ├── header.php
    ├── sidebar.php (opsional)
    └── footer.php
```

## Langkah 2
- buat tabel artikel di dalam database praktkum sebelumnya
```
CREATE TABLE artikel (
  id INT AUTO_INCREMENT PRIMARY KEY,
  judul VARCHAR(255) NOT NULL,
  isi TEXT NOT NULL
);
```

<img width="1920" height="1008" alt="Image" src="https://github.com/user-attachments/assets/77287263-8a3b-4d98-a2dc-50f1468debac" />


## Langkah 3
- File class OOP yang digunakan
- Database.php
```
<?php
class Database {
    protected $conn;

    public function __construct() {
        // KONEKSI FIX LANGSUNG (TIDAK PAKAI CONFIG)
        $host = "127.0.0.1";
        $user = "root";
        $pass = "";
        $db   = "lab10web";
        $port = 3307;

        $this->conn = @new mysqli($host, $user, $pass, $db, $port);

        if ($this->conn->connect_error) {
            die("Koneksi database gagal: " . $this->conn->connect_error);
        }
    }

    public function insert($table, $data) {
        $cols = [];
        $vals = [];

        foreach ($data as $key => $val) {
            $cols[] = $key;
            $vals[] = "'$val'";
        }

        $cols = implode(",", $cols);
        $vals = implode(",", $vals);

        $sql = "INSERT INTO $table ($cols) VALUES ($vals)";
        return $this->conn->query($sql);
    }

    public function query($sql) {
        return $this->conn->query($sql);
    }
}
?>
```
- Form.php
```
<?php
class Form {
    private $fields = array();
    private $action;
    private $submit = "Submit Form";
    private $jumField = 0;

    public function __construct($action, $submit) {
        $this->action = $action;
        $this->submit = $submit;
    }

    public function addField($name, $label) {
        $this->fields[$this->jumField]['name'] = $name;
        $this->fields[$this->jumField]['label'] = $label;
        $this->jumField++;
    }

    public function displayForm() {
        echo "<form action='".$this->action."' method='POST'>";
        echo "<table width='100%' border='0'>";
        
        for ($j = 0; $j < count($this->fields); $j++) {
            echo "<tr><td align='right'>".$this->fields[$j]['label']."</td>";
            echo "<td><input type='text' name='".$this->fields[$j]['name']."'></td></tr>";
        }

        echo "<tr><td colspan='2'>";
        echo "<input type='submit' value='".$this->submit."'></td></tr>";
        echo "</table>";
        echo "</form>";
    }
}
?>
```
## Langkah 4
- Routing Menggunakan .htaccess
- berfungsi agar URL bisa dibaca sebagai :
  - /artikel/index
  - /artikel/tambah
  - /artikel/ubah/3
- isi file :
```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /lab11_php_oop/

    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d

    RewriteRule ^(.*)$ index.php/$1 [L]
</IfModule>
```

## Langkah 5
- File index.php (Router Utama)
- index.php bertugas:
  - Menangkap URL / PATH_INFO
  - Menentukan modul dan halaman
  - Memuat template header & footer
  - Menjalankan file modul sesuai URL

```
<?php
// Load config
include "config.php";

// Load class
include "class/Database.php";
include "class/Form.php";

session_start();

// Tangkap path (contoh: /artikel/tambah)
$path = $_SERVER['PATH_INFO'] ?? '/artikel/index';

// Ubah menjadi array
$segments = explode('/', trim($path, '/'));

$mod  = $segments[0] ?? 'artikel';
$page = $segments[1] ?? 'index';

// Lokasi file modul
$file = "module/$mod/$page.php";

// Load template header
include "template/header.php";

// Cek file ada
if (file_exists($file)) {
    include $file;
} else {
    echo "<h3>Halaman tidak ditemukan: $mod/$page</h3>";
}

// Load footer
include "template/footer.php";
?>
```

## Langkah 6 
- artikel/index.php
- Menampilkan daftar artikel dalam tabel.

```
<?php
$db = new Database();
$result = $db->query("SELECT * FROM artikel");
?>

<div class="title-page">Daftar Artikel</div>

<a href="/lab11_php_oop/artikel/tambah" class="btn">➕ Tambah Artikel</a>

<table>
    <tr>
        <th>Judul</th>
        <th>Isi</th>
        <th>Aksi</th>
    </tr>

    <?php while($row = $result->fetch_assoc()): ?>
    <tr>
        <td><?= $row['judul'] ?></td>
        <td><?= $row['isi'] ?></td>
        <td>
            <a href="/lab11_php_oop/artikel/ubah/<?= $row['id'] ?>" class="btn" style="background:#FF9800">✏ Edit</a>
        </td>
    </tr>
    <?php endwhile; ?>
</table>
```

## Langkah 7 
- artikel/tambah.php
- Menampilkan form tambah artikel dan menyimpan data.
  
```
<?php
$db = new Database();
$form = new Form("", "Simpan Artikel");

if ($_POST) {
    $data = [
        'judul' => $_POST['judul'],
        'isi'   => $_POST['isi']
    ];

    if ($db->insert('artikel', $data)) {
        echo "<p style='color:green; font-weight:bold'>✔ Artikel berhasil disimpan!</p>";
    } else {
        echo "<p style='color:red; font-weight:bold'>✖ Gagal menyimpan artikel.</p>";
    }
}
?>

<div class="title-page">Tambah Artikel</div>

<?php
$form->addField("judul", "Judul Artikel");
$form->addField("isi", "Isi Artikel");
$form->displayForm();
?>
```
<img width="1920" height="1008" alt="Image" src="https://github.com/user-attachments/assets/5d8b9ab6-1068-4baf-a2d9-ff5564e8c183" />

## Langkah 8
- artikel/ubah.php
- Mengedit artikel berdasarkan ID.

```
<?php
$db = new Database();

// Tangkap ID dari URL
$id = isset($segments[2]) ? $segments[2] : 0;

// Ambil data artikel berdasarkan ID
$data = $db->query("SELECT * FROM artikel WHERE id = $id")->fetch_assoc();

if (!$data) {
    echo "<h3>Data tidak ditemukan!</h3>";
    return;
}

// FORM
$form = new Form("", "Update Artikel");

// Jika tombol submit ditekan
if ($_POST) {

    $judul = $_POST['judul'];
    $isi   = $_POST['isi'];

    $sql = "UPDATE artikel SET 
                judul = '$judul',
                isi   = '$isi' 
            WHERE id = $id";

    if ($db->query($sql)) {
        echo "<p style='color:green'>Artikel berhasil diperbarui!</p>";
    } else {
        echo "<p style='color:red'>Gagal memperbarui artikel.</p>";
    }

    // Update ulang data (agar field tetap terisi setelah update)
    $data = $db->query("SELECT * FROM artikel WHERE id = $id")->fetch_assoc();
}

?>

<h3>Ubah Artikel</h3>

<form method="POST">
    <table width="100%" border="0">
        <tr>
            <td width="20%">Judul Artikel</td>
            <td><input type="text" name="judul" value="<?= $data['judul'] ?>"></td>
        </tr>
        <tr>
            <td>Isi Artikel</td>
            <td><input type="text" name="isi" value="<?= $data['isi'] ?>"></td>
        </tr>
        <tr>
            <td colspan="2"><input type="submit" value="Update Artikel"></td>
        </tr>
    </table>
</form>
```

