# Laporan Perubahan Proyek - UCP 1

Laporan ini merangkum semua perubahan yang dilakukan pada proyek Laravel berdasarkan instruksi dalam dokumen `UCP 1.pdf`. Fokus utama perubahan adalah implementasi CRUD Kategori, relasi antar tabel, dan sistem otorisasi (Gates).

## 1. Perubahan Database (Migration)

Dilakukan pembuatan dua migrasi baru untuk menyesuaikan struktur database:
- **`create_category_table`**: Membuat tabel `category` dengan kolom `id`, `name` (unique), dan `timestamps`.
- **`add_category_id_to_products_table`**: Menambahkan kolom `category_id` ke tabel `products` sebagai foreign key yang merujuk ke tabel `category`. Menggunakan `onDelete('cascade')` untuk memastikan integritas data.
- **Pembersihan**: Menghapus tabel `kategoris` lama yang memiliki struktur yang tidak sesuai (memiliki `product_id`).

## 2. Model & Relasi Eloquent

### Model `Category`
Menambahkan model baru `Category.php` yang terhubung ke tabel `category`.
```php
public function products(): HasMany {
    return $this->hasMany(Product::class, 'category_id');
}
```

### Model `Product`
Memperbarui model `Product.php` untuk mendukung relasi ke kategori.
```php
protected $fillable = [..., 'category_id', ...];

public function category(): BelongsTo {
    return $this->belongsTo(Category::class, 'category_id');
}
```

## 3. Implementasi Controller

### `CategoryController`
Dibuat untuk menangani operasi CRUD kategori. Seluruh metode dilindungi oleh Gate `manage-category`.
- **`index`**: Menampilkan daftar kategori beserta jumlah produk menggunakan `withCount('products')`.
- **`store` / `update`**: Melakukan validasi nama kategori agar unik.

### `ProductController`
- Memperbarui metode `create` dan `edit` untuk mengirimkan data kategori ke view agar dapat dipilih dalam dropdown.
- Menggunakan eager loading `with(['category', 'user'])` pada metode `index` untuk meningkatkan performa.

## 4. Sistem Otorisasi (Gates)

Mendefinisikan Gate baru di `AppServiceProvider.php` untuk membatasi akses fitur kategori hanya bagi pengguna dengan role **Admin**.
```php
Gate::define('manage-category', function (User $user) {
    return $user->role === 'admin';
});
```

## 5. Pembaruan Antarmuka (Views)

- **Navigasi**: Menambahkan link "Category" di sidebar/header yang hanya muncul jika user adalah Admin (`@can('manage-category')`).
- **Form Produk**: Menambahkan field dropdown "Category" pada form tambah dan edit produk.
- **Daftar Produk**: Menampilkan kolom "Category" pada tabel produk.
- **Halaman Kategori**: Membuat halaman index, tambah, dan edit kategori dengan desain yang konsisten (Tailwind CSS).
