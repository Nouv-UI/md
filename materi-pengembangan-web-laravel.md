# Panduan Komprehensif Pengembangan Web Modern dengan Framework Laravel

Aplikasi web modern membutuhkan arsitektur yang terstruktur, aman, dan mudah dipelihara. Framework Laravel menyediakan landasan kokoh berbasis pola arsitektur **Model-View-Controller (MVC)** yang memisahkan tanggung jawab antara pengelolaan data, logika bisnis, dan antarmuka pengguna. Panduan ini menyajikan sintesis komprehensif dari seluruh materi praktikum pengembangan web berbasis Laravel, mencakup konsep dasar routing, mekanisme templating Blade, pengelolaan basis data melalui Migration, Eloquent Model, dan Seeder, hingga implementasi operasi CRUD (Create, Read, Update, Delete) yang aman dan terstandarisasi.

---

## 1. Arsitektur Model-View-Controller (MVC) dalam Laravel

Pola arsitektur Model-View-Controller (MVC) merupakan fondasi utama dalam ekosistem Laravel. Arsitektur ini dirancang untuk memisahkan komponen aplikasi berdasarkan peran dan tanggung jawabnya masing-masing (*Separation of Concerns*), sehingga memudahkan proses kolaborasi tim, pengujian, dan pemeliharaan kode program skala besar.

### Alur Kerja HTTP Request dalam Laravel
Ketika pengguna mengakses sebuah halaman web melalui *browser*, permintaan (*request*) bergerak melalui komponen-komponen utama aplikasi dengan urutan sistematis:

1. **Browser / Client**: Pengguna melakukan aksi, seperti mengetikkan URL atau menekan tombol form pada browser.
2. **Router (`routes/`)**: Memeriksa alamat URI dan metode HTTP yang masuk, lalu mencocokkannya dengan aturan rute yang terdaftar untuk diteruskan ke *Controller* atau *Closure*.
3. **Controller (`app/Http/Controllers/`)**: Bertindak sebagai pusat kendali (*brain*). Controller menerima input dari request, mengeksekusi logika bisnis, serta berinteraksi dengan *Model* jika memerlukan data dari database.
4. **Model (`app/Models/`)**: Mengelola komunikasi langsung dengan basis data menggunakan *Eloquent Object-Relational Mapper (ORM)* untuk mengambil, menyimpan, memperbarui, atau menghapus data.
5. **View (`resources/views/`)**: Menerima data hasil olahan dari Controller dan merendernya menjadi antarmuka HTML dinamis menggunakan *Blade Templating Engine*.
6. **Response**: Antarmuka HTML dikembalikan oleh View melalui Controller ke browser pengguna sebagai *HTTP Response*.

### Peran Masing-Masing Komponen MVC
Tabel berikut merangkum peran, lokasi berkas, serta contoh tanggung jawab dari ketiga komponen utama MVC dalam Laravel.

| Komponen | Lokasi Direktori Standar | Tanggung Jawab Utama | Contoh Implementasi |
| :--- | :--- | :--- | :--- |
| **Model** | `app/Models/` | Mengelola data, struktur tabel, relasi, dan aturan mass assignment. | `Post.php`, `Buku.php`, `Project.php` |
| **View** | `resources/views/` | Menyajikan tampilan HTML dinamis dan antarmuka pengguna. | `home.blade.php`, `posts/create.blade.php` |
| **Controller** | `app/Http/Controllers/` | Menerima request, memproses logika bisnis, dan menjembatani Model dengan View. | `PostController.php`, `ProjectController.php` |

---

## 2. Sistem Routing Laravel

*Routing* dalam Laravel berfungsi sebagai pintu gerbang utama yang menentukan bagaimana setiap permintaan HTTP dari alamat URL dipetakan ke penanganan logika tertentu. Router memastikan bahwa setiap permintaan pengguna diarahkan ke tujuan yang tepat dalam aplikasi.

### Struktur File dalam Folder `routes/`
Secara bawaan, Laravel menyediakan direktori `routes/` yang berisi empat berkas pendaftaran rute berdasarkan jenis layanan yang dibangun:

* **`web.php`**: Digunakan untuk mendaftarkan rute aplikasi web konvensional. Rute di berkas ini secara otomatis dilengkapi dengan middleware *session state*, verifikasi cookies, dan proteksi CSRF.
* **`api.php`**: Khusus untuk membangun *RESTful API* atau *core services*. Rute di sini bersifat *stateless* dan dilengkapi dengan pembatas laju request (*rate limiting*).
* **`channels.php`**: Mendaftarkan jalur *broadcasting event* untuk fitur aplikasi *real-time* dan notifikasi.
* **`console.php`**: Mendefinisikan perintah berbasis CLI (*Custom Artisan Commands*) yang dijalankan via terminal.

### Tipe HTTP Request Methods
Laravel mendukung seluruh metode standar HTTP untuk menangani transaksi data web:
* **`Route::get()`**: Untuk membaca atau menampilkan data dari server (bersifat *safe* dan *read-only*).
* **`Route::post()`**: Untuk mengirimkan data baru dari form ke server.
* **`Route::put()` / `Route::patch()`**: Untuk memperbarui data yang sudah ada di database.
* **`Route::delete()`**: Untuk menghapus data dari database.
* **`Route::resource()`**: Memetakan tujuh aksi standar CRUD secara otomatis ke sebuah Resource Controller.

### Penanganan Parameter URL
Parameter memungkinkan halaman web bersifat dinamis berdasarkan nilai yang dikirimkan melalui URL.

1. **Required Parameter**:
   Parameter wajib yang ditulis di dalam kurung kurawal `{param}`. Jika pengguna tidak menyertakan parameter pada URL, Laravel akan mengembalikan respons error **404 Not Found**.
   ```php
   Route::get('/halo/{nama}', function ($nama) {
       return '<h1>Halo ' . $nama . '</h1>';
   });
   ```

2. **Optional Parameter**:
   Parameter opsional yang ditandai dengan tanda tanya `{param?}` dan wajib disertai nilai *default* pada variabel fungsi penanganan.
   ```php
   Route::get('/halo/{nama?}', function ($nama = 'Tanpa Nama') {
       return '<h1>Halo ' . $nama . '</h1>';
   });
   ```

### Fitur Lanjutan Routing

#### 1. Route Redirect
Pengalihan halaman secara langsung tanpa memerlukan logika fungsi *Closure*:
```php
Route::redirect('/lama', '/baru');
```

#### 2. Named Routes
Pemberian nama unik pada rute menggunakan rantai method `->name()`. Fitur ini menghindarkan praktik *hardcoding* URL pada View maupun Controller. Jika alamat URI diubah di masa mendatang, seluruh tautan aplikasi yang menggunakan helper `route()` akan menyesuaikan secara otomatis.

* **Deklarasi Rute**:
  ```php
  Route::get('/home', fn() => view('home'))->name('home');
  Route::get('/halo/{nama}', fn($nama) => 'Halo ' . $nama)->name('halo');
  ```
* **Penggunaan pada File Blade**:
  ```html
  <a href="{{ route('home') }}">Beranda</a>
  <a href="{{ route('halo', ['nama' => 'Emcha']) }}">Sapa Emcha</a>
  ```
* **Penggunaan pada Controller (PHP)**:
  ```php
  return redirect()->route('home');
  ```

#### 3. Route Groups
Mengelompokkan sekumpulan rute yang memiliki kesamaan atribut seperti `prefix`, `middleware`, atau `namespace` untuk menjaga kerapian kode:
```php
Route::prefix('portfolio')->group(function () {
    Route::get('/', fn() => view('portfolio.home'))->name('portfolio.home');
    Route::get('/about', fn() => view('portfolio.about'))->name('portfolio.about');
    Route::get('/projects', fn() => view('portfolio.projects'))->name('portfolio.projects');
});
// Membentuk URL: /portfolio, /portfolio/about, /portfolio/projects
```

#### 4. Route Model Binding
Mekanisme di mana Laravel secara otomatis mencari dan menginjeksi instance Model berdasarkan nilai ID yang ditangkap dari parameter URL:
```php
public function show(Post $post)
{
    // Variabel $post otomatis berisi data model dari database berdasarkan ID di URL
    return view('posts.show', compact('post'));
}
```

#### 5. Inspeksi Rute via Terminal
Untuk memeriksa seluruh daftar rute yang terdaftar di aplikasi beserta HTTP Method, URI, nama rute, dan controller penanggung jawabnya, jalankan perintah:
```bash
php artisan route:list
```

---

## 3. Presentation Layer: Views dan Blade Templating Engine

*View* bertugas menyajikan antarmuka visual kepada pengguna. Dalam Laravel, berkas View disimpan di direktori `resources/views/` dengan ekstensi `.blade.php`.

### Pemanggilan View dan Struktur Sub-Folder
View dipanggil dari Router atau Controller menggunakan helper function `view()`. Tanda titik `.` digunakan sebagai pemisah direktori sub-folder.
```php
// Memanggil berkas resources/views/halo_dunia.blade.php
Route::get('/halo-dunia', fn() => view('halo_dunia'));

// Memanggil berkas resources/views/portfolio/home.blade.php
Route::get('/portfolio', fn() => view('portfolio.home'));
```

### Mengirim Data dari Controller/Route ke View
Terdapat tiga metode standar untuk mengirimkan variabel data dari Controller ke berkas Blade View:

1. **Metode Array Associative**:
   ```php
   return view('profil', [
       'nama' => 'Emcha',
       'umur' => 20,
       'kota' => 'Depok'
   ]);
   ```
2. **Metode Rantai `with()`**:
   ```php
   return view('profil')
       ->with('nama', 'Emcha')
       ->with('umur', 20);
   ```
3. **Metode Helper `compact()`** (Sangat Direkomendasikan):
   ```php
   $nama = 'Emcha';
   $umur = 20;
   $kota = 'Depok';
   return view('profil', compact('nama', 'umur', 'kota'));
   ```

### Syntax dan Directives Blade Templating Engine
Blade adalah *template engine* bawaan Laravel yang memungkinkan penulisan sintaks dinamis dengan bersih tanpa perlu menggunakan tag PHP mentah `<?php ?>`.

* **Pencetakan Data Escaped & Unescaped**:
  ```html
  <!-- Otomatis di-escape (aman dari XSS) -->
  <p>{{ $nama }}</p>

  <!-- Tanpa escape (untuk merender HTML mentah) -->
  <div>{!! $deskripsi_html !!}</div>

  <!-- Komentar Blade (tidak akan terdeteksi pada HTML Source Code browser) -->
  {{-- Ini adalah komentar di Blade --}}
  ```

* **Struktur Kondisional**:
  ```blade
  @if ($skor >= 80)
      <p>Nilai Sangat Memuaskan</p>
  @elseif ($skor >= 60)
      <p>Nilai Cukup</p>
  @else
      <p>Nilai Kurang</p>
  @endif

  @isset($nama)
      <p>Nama terdefinisi: {{ $nama }}</p>
  @endisset

  @empty($data)
      <p>Data masih kosong.</p>
  @endempty
  ```

* **Struktur Perulangan**:
  ```blade
  <!-- Foreach biasa -->
  @foreach ($projects as $project)
      <li>{{ $project->title }}</li>
  @endforeach

  <!-- Forelse: Otomatis menangani kondisi jika array/collection kosong -->
  @forelse ($projects as $project)
      <li>{{ $project->title }}</li>
  @empty
      <p>Belum ada data project yang tersedia.</p>
  @endforelse

  <!-- For dan While -->
  @for ($i = 0; $i < 5; $i++)
      <p>Item ke-{{ $i + 1 }}</p>
  @endfor
  ```

### Template Inheritance (Layouting Master & Child View)
Blade mendukung konsep pewarisan layout untuk menghindari pengulangan kode struktur HTML umum (seperti `<head>`, navbar, dan footer).

#### 1. File Master Layout (`resources/views/layouts/app.blade.php`):
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', config('app.name'))</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    @include('partials.navbar')

    <main class="container mx-auto my-6">
        @yield('content')
    </main>

    @include('partials.footer')

    @stack('scripts')
</body>
</html>
```

#### 2. File Child View (`resources/views/home.blade.php`):
```blade
@extends('layouts.app')

@section('title', 'Halaman Utama Portfolio')

@section('content')
    <h1>Selamat Datang di Portfolio Saya</h1>
    <p>Ini adalah konten utama halaman rumah.</p>
@endsection

@push('scripts')
    <script>
        console.log('Halaman Home berhasil dimuat');
    </script>
@endpush
```

#### 3. Partial Views (`@include`)
Digunakan untuk memisahkan komponen antarmuka berulang seperti komponen navbar, card, atau footer:
```blade
{{-- Menyisipkan partial navbar --}}
@include('partials.navbar')

{{-- Menyisipkan partial card dengan pengiriman parameter ekstra --}}
@include('partials.card', ['judul' => 'Project Utama'])
```

---

## 4. Business Logic Layer: Controller

*Controller* merupakan komponen penyatu dalam arsitektur MVC. Controller menerima HTTP Request dari Router, menjalankan verifikasi dan kalkulasi logika bisnis, berinteraksi dengan Eloquent Model, serta memilih View yang sesuai untuk dikembalikan.

### Membuat Controller
Pembuatan berkas Controller terletak di folder `app/Http/Controllers/`.

1. **Membuat Controller Kosong via Artisan**:
   ```bash
   php artisan make:controller PostController
   ```
2. **Membuat Resource Controller (Lengkap dengan Method CRUD)**:
   ```bash
   php artisan make:controller PostController --resource
   ```

### 7 Method Standar Resource Controller
Opsi `--resource` secara otomatis menyediakan 7 metode standar untuk pengolahan data lengkap:

| Method Controller | HTTP Method | URI Path | Fungsi Utama |
| :--- | :--- | :--- | :--- |
| `index()` | `GET` | `/posts` | Menampilkan daftar seluruh data |
| `create()` | `GET` | `/posts/create` | Menampilkan form tambah data baru |
| `store()` | `POST` | `/posts` | Menyimpan data baru ke database |
| `show()` | `GET` | `/posts/{id}` | Menampilkan detail satu data spesifik |
| `edit()` | `GET` | `/posts/{id}/edit` | Menampilkan form ubah data |
| `update()` | `PUT`/`PATCH` | `/posts/{id}` | Menyimpan perubahan data yang di-edit |
| `destroy()` | `DELETE` | `/posts/{id}` | Menghapus data dari database |

### Menghubungkan Route dengan Controller
Terdapat perubahan evolusi sintaks pendaftaran rute dari versi Laravel terdahulu ke versi modern:

* **Sintaks Laravel 8+ (Array Tuple Class)**:
  ```php
  use App\Http\Controllers\PostController;

  // Pendaftaran Rute Manual per Method
  Route::get('/posts', [PostController::class, 'index']);
  Route::post('/posts', [PostController::class, 'store']);

  // Pendaftaran Ringkas 7 Method Resource Sekaligus
  Route::resource('posts', PostController::class);
  ```
* **Sintaks Legacy Laravel 7 (String Syntax)**:
  ```php
  Route::get('/posts', 'PostController@index');
  ```

---

## 5. Database Layer: Migration, Model Eloquent, dan Seeder

Pengelolaan basis data dalam Laravel dilakukan secara terprogram tanpa memerlukan ekspor-impor berkas SQL manual melalui phpMyAdmin.

### Database Migration sebagai Version Control
Migration memungkinkan pengembang mendefinisikan, mengubah, dan membagikan skema tabel basis data dalam bentuk kode PHP terstruktur.

#### 1. Persiapan Koneksi Database
Sebelum menjalankan migration, konfigurasi basis data harus disesuaikan pada file `.env`:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=belajar_laravel
DB_USERNAME=root
DB_PASSWORD=
```

#### 2. Perintah Membuat Migration
* **Migration Terpisah**:
  ```bash
  php artisan make:migration create_posts_table
  ```
* **Model + Migration Sekaligus**:
  ```bash
  php artisan make:model Post --migration
  ```

#### 3. Anatomi File Migration (`up()` dan `down()`)
Berkas migration yang dihasilkan di `database/migrations/` memiliki dua fungsi utama:
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    // Dijalankan saat 'php artisan migrate' (membuat tabel/kolom)
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id(); // Primary Key Auto-Increment (UNSIGNED BIGINT)
            $table->string('title', 200); // VARCHAR(200)
            $table->mediumText('description'); // TEXT
            $table->timestamps(); // Menghasilkan kolom created_at dan updated_at
        });
    }

    // Dijalankan saat rollback (menghapus tabel)
    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

#### 4. Eksekusi Migration
Jalankan perintah berikut pada terminal untuk mengeksekusi migration:
```bash
php artisan migrate
```
Laravel akan membaca seluruh file migration di direktori `database/migrations/` dan mencatat daftar eksekusi pada tabel sistem `migrations` di dalam basis data untuk menghindari eksekusi ganda.

### Model Eloquent & Mass Assignment Protection
Model Eloquent (`app/Models/`) mewakili tabel dalam basis data. Secara *default*, Laravel melindungi model dari ancaman keamanan **Mass Assignment** (penyusupan kolom berbahaya saat pengisian data masal).

Untuk mengaktifkan pengisian masal via `Post::create()`, definisikan properti `$fillable` pada berkas Model:
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasFactory;

    // Kolom yang diizinkan untuk diisi secara Mass Assignment
    protected $fillable = [
        'title',
        'description',
    ];
}
```

### Database Seeder & Fake Data
Seeder digunakan untuk mengisi data awal (*seed*) atau data simulasi (*dummy*) ke dalam basis data.

1. **Membuat Seeder**:
   ```bash
   php artisan make:seeder BukuSeeder
   ```
2. **Menuliskan Kode Seeder (`database/seeders/BukuSeeder.php`)**:
   ```php
   namespace Database\Seeders;

   use App\Models\Buku;
   use Illuminate\Database\Seeder;

   class BukuSeeder extends Seeder
   {
       public function run(): void
       {
           for ($i = 0; $i < 10; $i++) {
               Buku::create([
                   'judul' => fake()->sentence(3),
                   'penulis' => fake()->name(),
                   'harga' => fake()->numberBetween(10000, 50000),
                   'tgl_terbit' => fake()->date(),
               ]);
           }
       }
   }
   ```
3. **Mengeksekusi Seeder**:
   ```bash
   php artisan db:seed --class=BukuSeeder
   ```

---

## 6. Implementasi Operasi CRUD Lengkap

Berikut adalah implementasi menyeluruh operasi CRUD pada `PostController` beserta penanganan validasi input, proteksi keamanan, dan flash message.

### 1. Read Operations (`index()` dan `show()`)
```php
// Menampilkan seluruh data post
public function index()
{
    $posts = Post::latest()->get(); // Mengambil data terbaru
    return view('posts.index', compact('posts'));
}

// Menampilkan detail satu data spesifik
public function show(string $id)
{
    // findOrFail akan otomatis melempar HTTP Error 404 jika ID tidak ditemukan
    $post = Post::findOrFail($id);
    return view('posts.show', compact('post'));
}
```

### 2. Create Operations (`create()` dan `store()`)
```php
// Menampilkan form tambah data
public function create()
{
    return view('posts.create');
}

// Memproses penyimpanan data baru
public function store(Request $request)
{
    // 1. Validasi Input
    $validatedData = $request->validate([
        'title' => 'required|min:5|max:200',
        'description' => 'required|min:10',
    ]);

    // 2. Simpan ke Database
    Post::create($validatedData);

    // 3. Redirect disertai Flash Message
    return redirect()->route('posts.index')
        ->with('success', 'Post berhasil ditambahkan.');
}
```

### 3. Update Operations (`edit()` dan `update()`)
```php
// Menampilkan form edit data
public function edit(string $id)
{
    $post = Post::findOrFail($id);
    return view('posts.edit', compact('post'));
}

// Memproses perubahan data
public function update(Request $request, string $id)
{
    // 1. Validasi Input
    $validatedData = $request->validate([
        'title' => 'required|min:5|max:200',
        'description' => 'required|min:10',
    ]);

    // 2. Cari Data & Perbarui
    $post = Post::findOrFail($id);
    $post->update($validatedData);

    // 3. Redirect disertai Flash Message
    return redirect()->route('posts.index')
        ->with('success', 'Post berhasil diperbarui.');
}
```

### 4. Delete Operations (`destroy()`)
```php
// Memproses penghapusan data
public function destroy(string $id)
{
    $post = Post::findOrFail($id);
    $post->delete();

    return redirect()->route('posts.index')
        ->with('success', 'Post berhasil dihapus.');
}
```

---

## 7. Keamanan Aplikasi Web dan Best Practices Laravel

Laravel menyertakan proteksi keamanan tingkat tinggi secara langsung dari kotaknya (*out-of-the-box*).

### Proteksi Cross-Site Request Forgery (CSRF)
CSRF adalah teknik serangan berbahaya di mana pihak ketiga mengeksekusi aksi yang tidak disadari pengguna saat mereka sedang terautentikasi.
* **Mekanisme**: Setiap form HTML berbuka `POST`, `PUT`, `PATCH`, atau `DELETE` **wajib** menyertakan direktif `@csrf`. Direktif ini menghasilkan input tersembunyi yang membawa token rahasia session.
  ```html
  <form action="{{ route('posts.store') }}" method="POST">
      @csrf
      <!-- Input fields -->
  </form>
  ```

### Method Spoofing
Browser web secara standar **hanya mendukung metode HTTP GET dan POST** pada form HTML. Untuk mengirimkan permintaan `PUT` atau `DELETE`, Laravel menggunakan teknik *Method Spoofing* melalui direktif `@method()`.

Contoh Form Edit Data (`PUT`):
```html
<form action="{{ route('posts.update', $post->id) }}" method="POST">
    @csrf
    @method('PUT')

    <input type="text" name="title" value="{{ old('title', $post->title) }}">
    <button type="submit">Update</button>
</form>
```

### Mengapa Aksi Delete Wajib Menggunakan `<form>` dan Bukan Tag `<a>`?
Penghapusan data tidak boleh dilakukan melalui tautan hyperlink biasa (`<a href="/posts/delete/1">`).
1. **Semantik HTTP**: Tag `<a>` selalu mengirimkan metode HTTP `GET`. Berdasarkan standar HTTP, request `GET` bersifat *safe* dan *idempotent* (hanya untuk membaca, tidak boleh mengubah/menghapus state server).
2. **Bahaya Bot & Crawler**: Mesin pencari atau *bot crawler* yang menjelajah tautan halaman dapat tidak sengaja mengeksekusi URL penghapusan dan mengosongkan seluruh isi database.
3. **Celah Keamanan CSRF**: Tautan GET tidak dapat diverifikasi dengan token CSRF, sehingga penyerang dapat menanamkan tag gambar palsu `<img src="https://situs-anda.com/posts/1/delete">` di situs luar untuk menghapus data pengguna secara otomatis.

Form Aksi Hapus Data yang Aman (`DELETE`):
```html
<form action="{{ route('posts.destroy', $post->id) }}" method="POST" style="display:inline">
    @csrf
    @method('DELETE')
    <button type="submit" onclick="return confirm('Yakin ingin menghapus data ini?')">Hapus</button>
</form>
```

### Repopulasi Input Form dan Pesan Error Validasi
Saat validasi gagal, Laravel otomatis mengembalikan pengguna ke halaman form. Gunakan helper `old()` dan direktif `@error` untuk memberikan umpan balik pengguna yang responsif:
```html
<div>
    <label for="title">Judul Post</label>
    <input type="text" name="title" id="title" value="{{ old('title', $post->title ?? '') }}">
    @error('title')
        <span class="text-red-500">{{ $message }}</span>
    @enderror
</div>
```

### Integrasi Version Control (Git & GitHub)
Untuk mempublikasikan proyek ke repositori GitHub, gunakan urutan perintah Git berikut di terminal:
```bash
git init
git add .
git commit -m "Initial commit - Laravel MVC Feature Complete"
git branch -M main
git remote add origin https://github.com/username/repository-name.git
git push -u origin main
```

---

## 8. Ringkasan Sintaks dan Alur Kerja Pengembangan Fitur

Sebagai panduan praktis harian, berikut adalah rangkuman perintah CLI Artisan dan urutan langkah terstruktur dalam membangun fitur baru pada Laravel.

### Rangkuman Perintah CLI Artisan Utama
| Perintah CLI Artisan | Deskripsi dan Kegunaan |
| :--- | :--- |
| `php artisan serve` | Menjalankan server pengembangan lokal (`http://127.0.0.1:8000`). |
| `php artisan route:list` | Memeriksa seluruh daftar rute, URI, method HTTP, dan controller terdaftar. |
| `php artisan make:controller NamaController --resource` | Membuat Resource Controller lengkap dengan 7 method CRUD. |
| `php artisan make:model NamaModel --migration` | Membuat Eloquent Model beserta file Migration tabel terkait. |
| `php artisan make:migration create_nama_tabel` | Membuat berkas migration skema tabel baru. |
| `php artisan migrate` | Mengeksekusi seluruh file migration yang belum berjalan ke basis data. |
| `php artisan make:seeder NamaSeeder` | Membuat berkas seeder untuk pengisian data dummy. |
| `php artisan db:seed --class=NamaSeeder` | Mengeksekusi kelas seeder tertentu. |

### Checklist Langkah Sistematis Pengembangan Fitur Baru
Saat membangun fitur baru pada aplikasi berbasis Laravel, ikuti alur kerja terstandar berikut:

1. **Inisialisasi Database**:
   * Buat database pada phpMyAdmin/MySQL dan hubungkan via `.env`.
   * Jalankan `php artisan make:model NamaModel --migration` untuk membangkitkan Model dan Migration sekaligus.
   * Lengkapi skema tabel pada method `up()` di berkas migration, lalu jalankan `php artisan migrate`.
   * Tambahkan properti `protected $fillable` pada berkas Model untuk mengaktifkan Mass Assignment.
2. **Inisialisasi Logika dan Rute**:
   * Jalankan `php artisan make:controller NamaController --resource` untuk membuat Resource Controller.
   * Daftarkan rute pada `routes/web.php` menggunakan `Route::resource('nama-fitur', NamaController::class)`.
   * Verifikasi kepastian rute menggunakan `php artisan route:list`.
3. **Pengembangan Tampilan (Views) dan Logika**:
   * Buat struktur berkas tampilan di `resources/views/nama-fitur/` (`index`, `create`, `show`, `edit`).
   * Terapkan pewarisan master layout (`@extends`, `@section`, `@yield`) dan parsial navbar/footer (`@include`).
   * Lengkapi logika method controller (`index`, `store`, `update`, `destroy`) dengan validasi `$request->validate()` dan proteksi form (`@csrf`, `@method`).
