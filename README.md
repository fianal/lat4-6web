# lab7_php_ci

| Data diri| 
|-----------------|
| Nama : Alfian Nur Rizki  | 
| Kelas : TI 23.A6 | 
| NIM : 312310665 | 

<h1> Praktikum 4: Framework Lanjutan (Modul Login) </h1>

## Membuat Tabel: User Login

|  Field                 | Tipe Data  | Ukuran | Keterangan |
|---------------------------|----------|-------|--------|
| id |    INT   |     11    | PRIMARY KEY, auto_increment |
| username |   VARCHAR    |     200    | |
| useremail |   VARCHAR    |    200     | |
| userpassword |    VARCHAR   |     200    | |

## Membuat Tabel User

```
CREATE TABLE user (
id INT(11) auto_increment,
username VARCHAR(200) NOT NULL,
useremail VARCHAR(200),
userpassword VARCHAR(200),
PRIMARY KEY(id)
);
```

## Membuat Model User 
<p>Selanjutnya adalah membuat Model untuk memproses data Login. Buat file baru pada direktori app/Models dengan nama UserModel.php</p>

```
<?php

namespace App\Models;

use CodeIgniter\Model;

class UserModel extends Model
{
protected $table = 'user';
protected $primaryKey = 'id';
protected $useAutoIncrement = true;
protected $allowedFields = ['username', 'useremail', 'userpassword'];
}
```

## Membuat Controller User
<p>Buat Controller baru dengan nama User.php pada direktori app/Controllers. Kemudian tambahkan method index() untuk menampilkan daftar user, dan method login() untuk proses login.</p>

```
<?php

namespace App\Controllers;
use App\Models\UserModel;

class User extends BaseController
{
    public function index()
    {
        $title = 'Daftar User';
        $model = new UserModel();
        $users = $model->findAll();
        
        return view('user/index', compact('users', 'title'));
    }

    public function login()
    {
        helper(['form']);
        $email = $this->request->getPost('email');
        $password = $this->request->getPost('password');

        if (!$email) {
            return view('user/login');
        }

        $session = session();
        $model = new UserModel();
        $login = $model->where('useremail', $email)->first();

        if ($login) {
            $pass = $login['userpassword'];

            if (password_verify($password, $pass)) {
                $login_data = [
                    'user_id'    => $login['id'],
                    'user_name'  => $login['username'],
                    'user_email' => $login['useremail'],
                    'logged_in'  => true,
                ];
                $session->set($login_data);
                return redirect()->to('admin/artikel');
            } else {
                $session->setFlashdata("flash_msg", "Password salah.");
                return redirect()->to('/user/login');
            }
        } else {
            $session->setFlashdata("flash_msg", "Email tidak terdaftar.");
            return redirect()->to('/user/login');
        }
    }
    public function logout()
    {
        session()->destroy();
        return redirect()->to('/user/login');
    }
}

```

## Membuat View Login
<p>Buat direktori baru dengan nama user pada direktori app/views, kemudian buat file baru dengan nama login.php.</p>

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <link rel="stylesheet" href="<?= base_url('/login.css'); ?>">
</head>
<body>
    <div id="login-wrapper">
        <h1>Sign In</h1>

        <?php if (session()->getFlashdata('flash_msg')): ?>
            <div class="alert alert-danger">
                <?= session()->getFlashdata('flash_msg') ?>
            </div>
        <?php endif; ?>

        <form action="" method="post">
            <div class="mb-3">
                <label for="InputForEmail" class="form-label">Email address</label>
                <input type="email" name="email" class="form-control" id="InputForEmail" value="<?= set_value('email') ?>">
            </div>

            <div class="mb-3">
                <label for="InputForPassword" class="form-label">Password</label>
                <input type="password" name="password" class="form-control" id="InputForPassword">
            </div>

            <button type="submit" class="btn btn-primary">Login</button>
        </form>
    </div>
</body>
</html>
```

## Membuat database seeder

<p>Database seeder digunakan untuk membuat data dummy. Untuk keperluan ujicoba modul login, kita perlu memasukkan data user dan password kedaalam database. Untuk itu buatdatabase seeder untuk tabel user. Buka CLI, kemudian tulis perintah berikut:</p>

```
php spark make:seeder UserSeeder
```

<p>Selanjutnya, buka file UserSeeder.php yang berada di lokasi direktori /app/Database/Seeds/UserSeeder.php kemudian isi dengan kode berikut:</p>

```
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run()
    {
        $model = model('UserModel');
        $model->insert([
            'username' => 'admin',
            'useremail' => 'admin@email.com',
            'userpassword' => password_hash('admin123', PASSWORD_DEFAULT),
        ]);
    }
}
```

<p>Selanjutnya buka kembali CLI dan ketik perintah berikut:</p>

```
php spark db:seed UserSeeder
```

## Uji Coba Login
<p>Selanjutnya buka url http://localhost:8080/user/login seperti berikut:</p>

![img](https://github.com/fianal/lat4-6web/blob/main/imgLat4-6/Login%20-%20Google%20Chrome%2008-05-2025%2011_38_30.png)

## Menambahkan Auth Filter
<p>Selanjutnya membuat filer untuk halaman admin. Buat file baru dengan nama Auth.php pada direktori app/Filters.</p>

```
<?php namespace App\Filters;

use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use CodeIgniter\Filters\FilterInterface;

class Auth implements FilterInterface
{
    public function before(RequestInterface $request, $arguments = null)
    {
        // jika user belum login
        if(! session()->get('logged_in')){
            // maka redirct ke halaman login
            return redirect()->to('/user/login');
        }
    }
    public function after(RequestInterface $request, ResponseInterface
$response, $arguments = null)
    {
        // Do something here
    }
}
```

<p>Selanjutnya buka file app/Config/Filters.php tambahkan kode berikut:</p>

```
'auth' => \App\Filters\Auth::class
```

```
public array $aliases = [
        'csrf'          => CSRF::class,
        'toolbar'       => DebugToolbar::class,
        'honeypot'      => Honeypot::class,
        'invalidchars'  => InvalidChars::class,
        'secureheaders' => SecureHeaders::class,
        'cors'          => Cors::class,
        'forcehttps'    => ForceHTTPS::class,
        'pagecache'     => PageCache::class,
        'performance'   => PerformanceMetrics::class,
        'auth'          => \App\Filters\Auth::class,
    ];
```

<p>Selanjutnya buka file app/Config/Routes.php dan sesuaikan kodenya.</p>

```
$routes->group('admin', ['filter' => 'auth'], function($routes) {
    $routes->get('/', 'Admin::dashboard'); // opsional, admin dashboard
    $routes->get('artikel', 'Artikel::admin_index');
    $routes->add('artikel/add', 'Artikel::add');
    $routes->add('artikel/edit/(:any)', 'Artikel::edit/$1');
    $routes->get('artikel/delete/(:any)', 'Artikel::delete/$1');
});
```

## Percobaan Akses Menu Admin
<p>Buka url dengan alamat http://localhost:8080/admin/artikel ketika alamat tersebut diakses maka, akan dimuculkan halaman login.</p>

![img](https://github.com/fianal/lat4-6web/blob/main/imgLat4-6/Login%20-%20Google%20Chrome%2008-05-2025%2011_39_09.png)

## Fungsi Logout
<p>Tambahkan method logout pada Controller User seperti berikut:</p>

```
public function logout()
{
session()->destroy();
return redirect()->to('/user/login');
}
```

<h1>Praktikum 5: Pagination dan Pencarian</h1>

## Membuat Pagination

<p>Pagination merupakan proses yang digunakan untuk membatasi tampilan yang panjang dari data yang banyak pada sebuah website. Fungsi pagination adalah memecah tampilan menjadi beberapa halaman tergantung banyaknya data yang akan ditampilkan pada setiap halaman.</p>

<p>Untuk membuat pagination, buka Kembali Controller Artikel, kemudian modifikasi kode pada method admin_index seperti berikut.</p>

```
public function admin_index()
    {
        $title = 'Daftar Artikel';
        $model = new ArtikelModel();
        $data = [
            'title' => $title,
            'artikel' => $model->paginate(10), // data dibatasi 10 record per halaman
            'pager' => $model->pager,
        ];
        return view('artikel/admin_index', $data);
    }
```

<p>Kemudian buka file views/artikel/admin_index.php dan tambahkan kode berikut dibawah deklarasi tabel data.</p>

```
<?= $pager->links(); ?>
```

<p>Selanjutnya buka kembali menu daftar artikel, tambahkan data lagi untuk melihat hasilnya.</p>

![img](https://github.com/fianal/lat4-6web/blob/main/imgLat4-6/Daftar%20Artikel%20-%20Google%20Chrome%2008-05-2025%2012_37_00.png)

## Membuat Pencarian 
<p>Pencarian data digunakan untuk memfilter data</p>

<p>Untuk membuat pencarian data, buka kembali Controller Artikel, pada method admin_index ubah kodenya seperti berikut</p>

```
public function admin_index()
    {
        $title = 'Daftar Artikel';
        $model = new ArtikelModel();
        $data = [
            'title' => $title,
            'artikel' => $model->paginate(10), // data dibatasi 10 record per halaman
            'pager' => $model->pager,
        ];
        return view('artikel/admin_index', $data);
    }
```

<p>Dan pada link pager ubah seperti berikut.</p>

```
<?= $pager->only(['q'])->links(); ?>
```

<p>Selanjutnya uji coba dengan membuka kembali halaman admin artikel, masukkan kata kunci tertentu pada form pencarian.</p>

![img](https://github.com/fianal/lat4-6web/blob/main/imgLat4-6/Daftar%20Artikel%20-%20Google%20Chrome%2008-05-2025%2012_34_49.png)

<h1> Praktikum 6: Upload File Gambar </h1>

## Upload Gambar pada Artikel
<p>Menambahkan fungsi unggah gambar pada tambah artikel.
Buka kembali Controller Artikel pada project sebelumnya, sesuaikan kode pada method add seperti berikut:</p>

```
public function add()
    {
        // validasi data.
        $validation = \Config\Services::validation();
        $validation->setRules(['judul' => 'required']);
        $isDataValid = $validation->withRequest($this->request)->run();

        if ($isDataValid) 
        {
            $file = $this->request->getFile('gambar');
            $file->move(ROOTPATH . 'public/gambar');

            $artikel = new ArtikelModel();
            $artikel->insert([
                'judul' => $this->request->getPost('judul'),
                'isi' => $this->request->getPost('isi'),
                'slug' => url_title($this->request->getPost('judul')),
                'gambar' => $file->getName(),
            ]);
            return redirect('admin/artikel');
        }

        $title = "Tambah Artikel";
        return view('artikel/form_add', compact('title'));
    }
```

<p>Kemudian pada file views/artikel/form_add.php tambahkan field input file seperti berikut.</p>

```
<p>
<input type="file" name="gambar">
</p>
```

<p>Dan sesuaikan tag form dengan menambahkan ecrypt type seperti berikut.</p>

```
<form action="" method="post" enctype="multipart/form-data">
```

<p>Uji coba file upload dengan mengakses menu tambah artikel.</p>

![img](https://github.com/fianal/lat4-6web/blob/main/imgLat4-6/Daftar%20Artikel%20-%20Google%20Chrome%2008-05-2025%2013_11_24.png)







