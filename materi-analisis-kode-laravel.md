# Panduan Analisis Kode dan Troubleshooting Laravel: Persiapan Ujian Praktikum

Dokumen ini disusun khusus sebagai panduan komprehensif bagi Anda yang akan menghadapi ujian analisis kode dan *debugging* aplikasi web berbasis framework Laravel. Berdasarkan materi praktikum yang mencakup *Routing*, *View*, *Blade*, *Controller*, *Migration*, *Model Eloquent*, hingga operasi *CRUD*, panduan ini membekali Anda dengan metode sistematis untuk membaca kode, mendiagnosis akar masalah (*root cause analysis*), dan menerapkan solusi yang sesuai dengan standar konvensi Laravel.

---

## 1. Kerangka Kerja Analisis Kode dan Debugging

Dalam soal ujian berbasis analisis kode, kunci utama bukan sekadar menebak nama *method* atau sintaks yang salah, melainkan memahami bagaimana alur eksekusi Laravel bekerja di balik layar. Gunakan 3 langkah sistematis berikut saat menganalisis setiap potongan kode atau pesan *error*:

```
[Langkah 1: Observasi Kode & Gejala Error]
  └── Identifikasi pesan error (misal: 404, 419, MassAssignmentException, Undefined variable)
  └── Amati baris kode, lokasi file, HTTP method, dan parameter yang digunakan.

[Langkah 2: Pelacakan Alur & Root Cause Analysis]
  └── Telusuri alur eksekusi data (Route -> Controller -> Model/Database -> View).
  └── Analisis MENGAPA error terjadi (misal: data tidak sampai, token tidak terverifikasi, atau kolom diproteksi).

[Langkah 3: Evaluasi Konteks & Formulasi Solusi Laravel]
  └── Terapkan aturan dan konvensi resmi Laravel (misal: penggunaan @csrf, $fillable, @yield vs @include).
  └── Koreksi kode hingga memenuhi standar kejelasan dan keamanan framework.
```

### Tabel Ringkasan Tipe Error Populer dan Akar Masalahnya

| Pesan Error / Gejala | Lokasi Kemungkinan | Akar Masalah Utama (*Root Cause*) | Solusi Standar Laravel |
| :--- | :--- | :--- | :--- |
| **404 Not Found** | `routes/web.php` / URL Browser | Parameter URL bersifat wajib (*required*) tetapi tidak dikirimkan, atau ID tidak ditemukan pada `findOrFail()`. | Tambahkan parameter opsional `{param?}` dengan nilai *default*, atau pastikan URL sesuai daftar `route:list`. |
| **419 Page Expired** | Form Blade (`<form>`) | Form dengan metode `POST`, `PUT`, atau `DELETE` tidak menyertakan token keamanan CSRF. | Tambahkan direktif `@csrf` di dalam elemen `<form>`. |
| **MassAssignmentException** | Model (`app/Models/`) / Controller | Penggunaan metode massal seperti `Model::create()` atau `update()` tanpa mendaftarkan kolom pada properti `$fillable`. | Deklarasikan `protected $fillable = ['kolom1', 'kolom2'];` pada file Model. |
| **Undefined Variable `$data`** | View Blade (`resources/views/`) | Controller memanggil `view()` tanpa mengirimkan variabel, atau nama *key* variabel di `compact()` / array tidak cocok. | Pastikan pengiriman data menggunakan `compact('variabel')`, array `['variabel' => $data]`, atau `->with()`. |
| **Class 'App\Http\Controllers\... Not Found'** | `routes/web.php` | Controller belum di-import menggunakan instruksi `use` di bagian atas file route. | Tambahkan `use App\Http\Controllers\NamaController;` di bagian atas file `web.php`. |
| **Method Not Allowed HTTP Exception (405)** | Form / Routing | Metode HTTP yang dikirimkan form tidak cocok dengan definisi di route (misal: mengirim POST ke route GET atau PUT tanpa spoofing). | Sesuaikan metode pada route atau tambahkan `@method('PUT')` / `@method('DELETE')` pada form. |
| **SQLSTATE[HY000] [1049] Unknown database** | Configuration (`.env`) | Nama database pada variabel `DB_DATABASE` di file `.env` belum dibuat di MySQL/phpMyAdmin atau salah ketik. | Buat database di MySQL dan sesuaikan atribut `DB_DATABASE` di file `.env`. |

---

## 2. Domain 1: Routing, Parameter URL, dan HTTP Method

Routing merupakan pintu masuk utama aplikasi Laravel yang mengarahkan *request* HTTP ke logika yang sesuai.

### A. Konsep dan Aturan Penting
1. **Pemisahan File Route**: File `routes/web.php` khusus untuk *route* berbasis web (memiliki sesi, CSRF, dan merender View), sedangkan `routes/api.php` untuk API *stateless*.
2. **Sintaks Deklarasi Route**:
   * *Laravel 8+ (Modern)*: Menggunakan sintaks array `Route::get('/uri', [NamaController::class, 'method']);`.
   * *Laravel 7 (Legacy)*: Menggunakan sintaks string `Route::get('/uri', 'NamaController@method');`.
3. **Melihat Daftar Route**: Perintah terminal `php artisan route:list` digunakan untuk menginspeksi seluruh URI, nama route, metode HTTP, dan *action controller* yang terdaftar.

### B. Analisis Kasus & Code Snippet

#### Kasus 1.1: Error 404 Pada Parameter URL
* **Potongan Kode Bermasalah**:
  ```php
  // routes/web.php
  Route::get('/halo/{nama}', function($nama) {
      return 'Halo ' . $nama;
  });
  ```
* **Gejala**: Ketika pengguna mengakses URL `http://localhost:8000/halo` tanpa mengisi nama, browser menampilkan halaman **404 Not Found**.
* **Analisis Akar Masalah**: Parameter `{nama}` dikategorikan sebagai *Required Parameter*. Jika segmen URL tersebut kosong, *router* Laravel menganggap URL tidak cocok dengan rute mana pun.
* **Solusi Perbaikan Kode**:
  Ubah parameter menjadi *Optional Parameter* dengan menambahkan tanda tanya `?` dan memberikan nilai *default* pada variabel *closure*:
  ```php
  Route::get('/halo/{nama?}', function($nama = 'Tanpa Nama') {
      return 'Halo ' . $nama;
  });
  ```

#### Kasus 1.2: Hardcoding URL vs Named Routes
* **Potongan Kode Bermasalah**:
  ```html
  <!-- resources/views/navbar.blade.php -->
  <a href="/user/profile/settings/edit">Edit Profil</a>
  ```
* **Gejala**: Jika struktur URL pada `routes/web.php` diubah di kemudian hari, seluruh tautan di dalam file Blade akan patah (*broken link*).
* **Analisis Akar Masalah**: Penulisan URL secara *hardcoded* menciptakan ketergantungan kaku antara View dan struktur URI.
* **Solusi Perbaikan Kode**:
  Gunakan *Named Route* pada penulisan rute dan panggil menggunakan *helper* `route()` di Blade:
  ```php
  // routes/web.php
  Route::get('/user/profile/settings/edit', [UserController::class, 'edit'])->name('profile.edit');
  ```
  ```html
  <!-- resources/views/navbar.blade.php -->
  <a href="{{ route('profile.edit') }}">Edit Profil</a>
  
  <!-- Tautan dengan parameter -->
  <a href="{{ route('posts.show', ['post' => $post->id]) }}">Detail Post</a>
  ```

#### Kasus 1.3: Pengelompokan Route dengan Route Group
* **Kode Rujukan**:
  ```php
  Route::prefix('portfolio')->group(function() {
      Route::get('/', fn() => view('portfolio.home'))->name('portfolio.home');
      Route::get('/about', fn() => view('portfolio.about'))->name('portfolio.about');
  });
  ```
* **Poin Analisis**: `Route::prefix('portfolio')` secara otomatis menambahkan prefiks `/portfolio` pada seluruh URI di dalamnya, sehingga URL yang dihasilkan adalah `/portfolio` dan `/portfolio/about`.

---

## 3. Domain 2: Controller dan Resource Mapping

Controller berfungsi sebagai pengatur logika bisnis dalam arsitektur MVC, menjembatani *request* pengguna dari View menuju Model dan mengembalikan *response*.

### A. Konsep dan Aturan Penting
1. **Perintah Artisan**:
   * Controller Kosong: `php artisan make:controller PostController`
   * Resource Controller (CRUD Lengkap): `php artisan make:controller PostController --resource`
2. **Resource Controller Mapping**: Perintah `Route::resource('posts', PostController::class);` secara otomatis memetakan 7 metode standar CRUD tanpa perlu menuliskan rute satu per satu.

### B. Tabel Pemetaan 7 Method Resource Controller

| Action Method | URL URI | Method HTTP | Fungsi Utama | Nama Route Otomatis |
| :--- | :--- | :--- | :--- | :--- |
| **`index()`** | `/posts` | `GET` | Menampilkan daftar seluruh data | `posts.index` |
| **`create()`** | `/posts/create` | `GET` | Menampilkan form untuk membuat data baru | `posts.create` |
| **`store()`** | `/posts` | `POST` | Menyimpan data baru ke database | `posts.store` |
| **`show()`** | `/posts/{post}` | `GET` | Menampilkan detail satu data berdasarkan ID | `posts.show` |
| **`edit()`** | `/posts/{post}/edit` | `GET` | Menampilkan form untuk mengedit data | `posts.edit` |
| **`update()`** | `/posts/{post}` | `PUT/PATCH` | Menyimpan perubahan data ke database | `posts.update` |
| **`destroy()`** | `/posts/{post}` | `DELETE` | Menghapus data dari database | `posts.destroy` |

> **Jebakan Ujian**: Perhatikan nama method untuk menghapus data pada Resource Controller adalah **`destroy()`**, BUKAN `delete()` atau `drop()`.

### C. Analisis Kasus & Code Snippet

#### Kasus 2.1: Penanganan Data Tidak Ditemukan (`find` vs `findOrFail`)
* **Potongan Kode Bermasalah**:
  ```php
  public function show($id) {
      $post = Post::find($id);
      return view('posts.show', compact('post'));
  }
  ```
* **Gejala**: Jika `$id` yang dicari tidak ada di database, `$post` bernilai `null`, menyebabkan error `Attempt to read property "title" on null` saat View mencoba mengakses `$post->title`.
* **Analisis Akar Masalah**: `Post::find($id)` mengembalikan `null` jika record tidak ditemukan dan tidak menghentikan eksekusi program.
* **Solusi Perbaikan Kode**:
  Gunakan `findOrFail($id)` yang secara otomatis melempar kegagalan dan menampilkan halaman **404 Not Found** jika data tidak ditemukan:
  ```php
  public function show($id) {
      $post = Post::findOrFail($id);
      return view('posts.show', compact('post'));
  }
  ```

---

## 4. Domain 3: Blade Templating dan Layout Inheritance

Blade adalah *template engine* bawaan Laravel yang dikompilasi menjadi kode PHP murni tanpa menambahkan beban *overhead* performa.

### A. Konsep dan Aturan Penting
1. **Ekstensi File**: File View harus disimpan di direktori `resources/views/` dengan ekstensi `.blade.php`.
2. **Layout Inheritance**:
   * **`@extends('layouts.app')`**: Digunakan di file *child view* untuk mewarisi struktur dari *master layout*.
   * **`@yield('content')`**: Ditempatkan di file *master layout* sebagai penanda (*placeholder*) area dinamis yang akan diisi oleh *child view*.
   * **`@section('content') ... @endsection`**: Didefinisikan di *child view* untuk menyuplai isi konten ke dalam `@yield` yang cocok.
3. **Partial View (`@include`)**: Digunakan untuk menyisipkan komponen terpisah yang dipakai berulang kali (seperti navbar, footer, atau sidebar), contoh: `@include('partials.navbar')`.

### B. Analisis Kasus & Code Snippet

#### Kasus 3.1: Kesalahan Penggunaan `@yield` vs `@include`
* **Potongan Kode Bermasalah**:
  ```html
  <!-- resources/views/layouts/app.blade.php (Master Layout) -->
  <main>
      @include('content')
  </main>
  ```
* **Gejala**: Error `View [content] not found` saat aplikasi dijalankan.
* **Analisis Akar Masalah**: `@include` digunakan untuk memanggil file fisik view/partial tertentu. Untuk menyediakan blok dinamis yang diisi oleh halaman anak (*child view*), directives yang benar adalah `@yield`.
* **Solusi Perbaikan Kode**:
  Gunakan `@yield('content')` pada file induk dan `@section('content')` pada file anak:
  ```html
  <!-- Master Layout: resources/views/layouts/app.blade.php -->
  <html>
  <body>
      @include('partials.navbar') <!-- Mengimpor partials/navbar.blade.php -->
      <main>
          @yield('content') <!-- Slot dinamis -->
      </main>
  </body>
  </html>

  <!-- Child View: resources/views/home.blade.php -->
  @extends('layouts.app')

  @section('content')
      <h1>Selamat Datang di Halaman Home</h1>
  @endsection
  ```

#### Kasus 3.2: Penanganan Array Kosong pada Looping (`@forelse`)
* **Potongan Kode Biasa (`@foreach`)**:
  ```html
  @if(count($posts) > 0)
      @foreach($posts as $post)
          <p>{{ $post->title }}</p>
      @foreach
  @else
      <p>Belum ada data post.</p>
  @endif
  ```
* **Penyederhanaan Idiomatik Laravel (`@forelse`)**:
  ```html
  @forelse($posts as $post)
      <p>{{ $post->title }}</p>
  @empty
      <p>Belum ada data post.</p>
  @endforelse
  ```

#### Kasus 3.3: Penomoran Otomatis pada Tabel View
* **Potongan Kode**:
  ```html
  @foreach($books as $index => $book)
      <tr>
          <td>{{ $index + 1 }}</td> <!-- $index dimulai dari 0 -->
          <td>{{ $book->judul }}</td>
      </tr>
  @endforeach
  ```
  *Catatan*: Alternatif Blade modern dapat menggunakan variabel bawaan loop: `{{ $loop->iteration }}`.

---

## 5. Domain 4: Database Migration dan Konfigurasi Environment

Migration berfungsi sebagai *Version Control System* untuk basis data, memungkinkan pengembang mengelola skema tabel secara terprogram.

### A. Konsep dan Aturan Penting
1. **Konfigurasi File `.env`**: Sebelum menjalankan migrasi, koneksi ke MySQL harus diatur pada file `.env`:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=belajar_laravel
   DB_USERNAME=root
   DB_PASSWORD=
   ```
2. **Struktur File Migration**: Tersimpan di `database/migrations/` dan memiliki 2 *method* utama:
   * **`up()`**: Mengeksekusi pembuatan tabel atau penambahan kolom baru saat `php artisan migrate` dijalankan.
   * **`down()`**: Batalkan perubahan skema (*rollback*) atau menghapus tabel saat `php artisan migrate:rollback` dijalankan.
3. **Mekanisme Eksekusi**: Saat `php artisan migrate` dijalankan, Laravel membandingkan berkas di folder `database/migrations/` dengan catatan pada tabel sistem `migrations` di database.

### B. Analisis Kasus & Code Snippet

#### Kasus 4.1: Anatomi File Migration
* **Kode Rujukan**:
  ```php
  use Illuminate\Database\Migrations\Migration;
  use Illuminate\Database\Schema\Blueprint;
  use Illuminate\Support\Facades\Schema;

  return new class extends Migration
  {
      public function up(): void
      {
          Schema::create('posts', function (Blueprint $table) {
              $table->id(); // Primary key Auto-Increment
              $table->string('title', 200); // VARCHAR(200)
              $table->mediumText('description'); // TEXT
              $table->timestamps(); // Menghasilkan created_at dan updated_at
          });
      }

      public function down(): void
      {
          Schema::dropIfExists('posts');
      }
  };
  ```
* **Poin Analisis**: Jika perintah `php artisan migrate` gagal karena database tidak ditemukan, akar masalahnya berada pada konfigurasi file `.env` (variabel `DB_DATABASE`) atau basis data belum dibuat di MySQL/phpMyAdmin.

---

## 6. Domain 5: Eloquent Model dan Proteksi Mass Assignment

Eloquent ORM memetakan tabel database ke dalam kelas PHP (*Model*) untuk mempermudah manipulasi data berbasis objek.

### A. Konsep dan Aturan Penting
1. **Perintah Pembuatan Modul Otomatis**:
   `php artisan make:model Post --migration` (membuat Model sekaligus file Migration).
2. **Mass Assignment Protection**: Secara *default*, Eloquent melindungi aplikasi dari kerentanan *Mass Assignment* (penyusupan data kolom secara tak terizin).
3. **Metode Penyimpanan Data**:
   * **Manual**: Menggunakan `new Model` lalu set properti satu per satu dan panggil `save()`. (Tidak memerlukan `$fillable`).
   * **Massal (`create`)**: Menggunakan `Model::create($arrayData)`. **Wajib** mendeklarasikan properti `$fillable` di dalam Model.

### B. Analisis Kasus & Code Snippet

#### Kasus 5.1: Error `MassAssignmentException`
* **Potongan Kode Bermasalah**:
  ```php
  // Controller
  public function store(Request $request) {
      Post::create([
          'title' => $request->title,
          'description' => $request->description,
      ]);
      return redirect()->route('posts.index');
  }

  // app/Models/Post.php
  class Post extends Model {
      // Properti $fillable kosong/belum didefinisikan
  }
  ```
* **Gejala**: Aplikasi mengalami crash dan menampilkan error **`Illuminate\Database\Eloquent\MassAssignmentException`**.
* **Analisis Akar Masalah**: Metode `Post::create()` mencoba memasukkan data secara massal, namun Eloquent memblokirnya karena kolom `title` dan `description` belum terdaftar pada array proteksi `$fillable`.
* **Solusi Perbaikan Kode**:
  Buka file `app/Models/Post.php` dan tambahkan properti `$fillable`:
  ```php
  namespace App\Models;

  use Illuminate\Database\Eloquent\Model;

  class Post extends Model
  {
      protected $fillable = [
          'title',
          'description',
      ];
  }
  ```

#### Kasus 5.2: Perbandingan Penyimpanan Manual vs Massal
```php
// CARA 1: Manual (Aman tanpa $fillable)
$post = new Post;
$post->title = $request->input('title');
$post->description = $request->input('description');
$post->save();

// CARA 2: Ringkas dengan create() (Wajib menggunakan $fillable di Model)
Post::create([
    'title' => $request->input('title'),
    'description' => $request->input('description'),
]);
```

---

## 7. Domain 6: Pengolahan Form, Keamanan CSRF, dan Method Spoofing

Pengolahan form dalam Laravel memiliki mekanisme keamanan ketat dan aturan penanganan metode HTTP.

### A. Konsep dan Aturan Penting
1. **Proteksi CSRF (Cross-Site Request Forgery)**: Setiap form dengan metode `POST`, `PUT`, `PATCH`, atau `DELETE` wajib menyertakan token CSRF melalui `@csrf`.
2. **Keterbatasan Form HTML & Method Spoofing**: Form standar HTML browser hanya mendukung metode `GET` dan `POST`. Untuk melakukan permintaan `PUT` (update) atau `DELETE` (destroy), Laravel menggunakan trik *Method Spoofing* lewat direktif `@method('PUT')` atau `@method('DELETE')`.
3. **Bahaya Menggunakan HTTP GET untuk Operasi Hapus Data**:
   * *Risiko Bot/Crawler*: Bot pencari indeks dapat mengikuti tautan `<a href="/posts/1/delete">` dan secara tidak sengaja menghapus seluruh data.
   * *Risiko Serangan CSRF*: Tautan `GET` tidak diperiksa token CSRF-nya oleh Laravel middleware, sehingga rentan disusupi lewat elemen tersembunyi seperti `<img src="http://situs.com/posts/1/delete">`.

### B. Analisis Kasus & Code Snippet

#### Kasus 6.1: Error 419 Page Expired
* **Potongan Kode Bermasalah**:
  ```html
  <!-- resources/views/posts/create.blade.php -->
  <form action="{{ route('posts.store') }}" method="POST">
      <input type="text" name="title">
      <button type="submit">Simpan</button>
  </form>
  ```
* **Gejala**: Pengguna mengirimkan form namun browser menampilkan layar error **`419 | Page Expired`**.
* **Analisis Akar Masalah**: Form dikirimkan tanpa menyertakan token CSRF, sehingga middleware `VerifyCsrfToken` Laravel menolak permintaan karena menganggapnya sebagai serangan dari pihak ketiga.
* **Solusi Perbaikan Kode**:
  Tambahkan direktif `@csrf` di dalam elemen form:
  ```html
  <form action="{{ route('posts.store') }}" method="POST">
      @csrf
      <input type="text" name="title">
      <button type="submit">Simpan</button>
  </form>
  ```

#### Kasus 6.2: Implementasi Form Hapus Data (Delete) yang Aman
* **Potongan Kode SALAH (Menggunakan Tautan `<a>`)**:
  ```html
  <!-- SALAH: Tautan <a> selalu mengirimkan request GET -->
  <a href="/posts/{{ $post->id }}">Hapus</a>
  ```
* **Analisis**: Tautan `<a>` akan membuka halaman detail `posts.show`, BUKAN menghapus data, karena rute `destroy` pada Resource Controller mewajibkan metode HTTP `DELETE`.
* **Potongan Kode BENAR (Menggunakan Form, `@csrf`, dan `@method('DELETE')`)**:
  ```html
  <form action="{{ route('posts.destroy', $post->id) }}" method="POST" style="display:inline">
      @csrf
      @method('DELETE')
      <button type="submit" onclick="return confirm('Yakin hapus data?')">Delete</button>
  </form>
  ```
* **Penjelasan HTML Terkompilasi di Browser**:
  Di balik layar, Laravel menerjemahkan Blade di atas menjadi HTML standar berikut:
  ```html
  <form action="http://localhost:8000/posts/1" method="POST" style="display:inline">
      <input type="hidden" name="_token" value="xYz123RandomCsrfToken">
      <input type="hidden" name="_method" value="DELETE">
      <button type="submit">Delete</button>
  </form>
  ```

---

## 8. Domain 7: Validasi Request, Error Messages, dan Flash Data

Validasi masukan dan penyampaian umpan balik (*feedback*) merupakan bagian vital dari pengalaman pengguna (UX) yang aman.

### A. Konsep dan Aturan Penting
1. **`$request->validate()`**: Jika validasi gagal, Laravel secara otomatis menghentikan eksekusi, mengembalikan (*redirect*) pengguna ke halaman form sebelumnya, serta membawa variabel `$errors` dan data masukan lama (*old input*).
2. **Penyimpanan Nilai Input Lama**: Menggunakan helper `old('nama_field', $default_value)` di Blade agar data yang sudah diketik pengguna tidak hilang saat terjadi error validasi.
3. **Flash Message Session**: Pesan pemberitahuan sementara yang disimpan di *session* hanya untuk satu kali *request cycle* berikutnya (misal setelah berhasil menyimpan/mengubah data).

### B. Analisis Kasus & Code Snippet

#### Kasus 7.1: Validasi dan Penyimpanan di Controller
```php
public function store(Request $request) {
    // 1. Validasi Input
    $validatedData = $request->validate([
        'title'       => 'required|string|min:5|max:200',
        'description' => 'required|string|min:10',
    ]);

    // 2. Simpan Data ke Database
    Post::create($validatedData);

    // 3. Redirect dengan Flash Message
    return redirect()->route('posts.index')
        ->with('success', 'Data post berhasil ditambahkan!');
}
```

#### Kasus 7.2: Menampilkan Error Validasi dan Flash Message di View Blade
```html
<!-- Form Edit / Create -->
<form action="{{ route('posts.update', $post->id) }}" method="POST">
    @csrf
    @method('PUT')

    <div>
        <label for="title">Judul Post</label>
        <!-- Menggunakan old() dengan fallback data database ($post->title) -->
        <input type="text" name="title" id="title" 
               value="{{ old('title', $post->title) }}" 
               class="form-control @error('title') is-invalid @enderror">
        
        <!-- Menampilkan pesan error spesifik kolom -->
        @error('title')
            <div class="alert alert-danger">{{ $message }}</div>
        @enderror
    </div>

    <button type="submit">Update</button>
</form>

<!-- Menampilkan Flash Message Sukses pada Index View -->
@if (session('success'))
    <div class="alert alert-success">
        {{ session('success') }}
    </div>
@endif
```

---

## 9. Matriks Penanganan Kasus Ujian Analisis Kode

Gunakan tabel rujukan cepat ini saat membaca soal ujian yang menyajikan potongan kode dan meminta diagnosis perbaikan:

| Potongan Kode / Soal Ujian | Pertanyaan Ujian | Root Cause Analysis | Jawaban & Solusi Kode |
| :--- | :--- | :--- | :--- |
| `Route::get('/post/{id}', [PostController::class, 'show']);`<br>`public function show($id) { $post = Post::find($id); return view('show', compact('post')); }` | Mengapa halaman error saat ID tidak ada di database? | `find()` mengembalikan `null`, sehingga saat view mengakses `$post->title` terjadi error *property on null*. | Ganti `Post::find($id)` menjadi `Post::findOrFail($id)` agar otomatis menampilkan halaman 404 jika ID tidak ditemukan. |
| `<form action="/posts/1" method="POST">`<br>`<button type="submit">Delete</button></form>` | Mengapa form ini malah mengeksekusi method `store` alih-alih `destroy`? | Form dikirim dengan metode POST tanpa *method spoofing*, sehingga dipetakan ke rute `store`. | Tambahkan `@method('DELETE')` dan `@csrf` di dalam form agar dibaca sebagai HTTP `DELETE` oleh Laravel. |
| `Post::create($request->all());` | Mengapa muncul `MassAssignmentException`? | Kolom-kolom tabel belum didaftarkan pada properti `$fillable` di file Model `Post.php`. | Deklarasikan `protected $fillable = ['title', 'description'];` pada kelas `Post` di `app/Models/Post.php`. |
| `<form action="/posts" method="POST">`<br>`<input type="text" name="title">...</form>` | Mengapa setelah tombol klik muncul error 419 Page Expired? | Form tidak membawa token CSRF untuk verifikasi keamanan *request*. | Tambahkan `@csrf` langsung di bawah elemen pembuka `<form>`. |
| `<a href="/posts/{{ $post->id }}/delete">Hapus</a>` | Apa 2 potensi bahaya dari kode hapus data tersebut? | 1. Menggunakan method GET (bisa di-crawl bot).<br>2. Tidak terlindungi token CSRF. | Ganti tautan `<a>` dengan `<form>` bertipe POST, tambahkan `@csrf` dan `@method('DELETE')`. |

---

## 10. Panduan Ringkas Menjawab Soal Analisis Kode

Saat menjawab soal ujian tertulis atau praktikum, susun jawaban Anda dengan struktur berikut agar mendapatkan nilai maksimal:

1. **Identifikasi Masalah**: Sebutkan jenis error atau perilaku tak diharapkan yang muncul dari potongan kode (contoh: *"Kode mengalami error 419 Page Expired"* atau *"Terjadi MassAssignmentException"*).
2. **Penjelasan Akar Masalah (*Root Cause*)**: Jelaskan mekanisme Laravel yang melandasi terjadinya error tersebut (contoh: *"Error 419 terjadi karena form menggunakan method POST tetapi tidak menyertakan token CSRF yang diwajibkan oleh middleware VerifyCsrfToken Laravel"*).
3. **Kode Solusi yang Dibenarkan**: Tuliskan potongan kode perbaikan yang lengkap dan bersih, serta jelaskan perubahan yang dilakukan (contoh: *"Menambahkan direktif `@csrf` pada elemen form"*).
