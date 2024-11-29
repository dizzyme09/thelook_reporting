# thelook_reporting
> Code:
```sql
CREATE TEMP TABLE report_monthly_orders_product_agg
AS
SELECT 
    DATE_TRUNC(o.created_at, YEAR) AS year,
    DATE_TRUNC(o.created_at, MONTH) AS month,
    p.name AS product_name,
    p.category AS product_category,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    SUM(oi.sale_price) AS total_sales,
    SUM(o.num_of_item) AS total_items_sold,
    ROUND(SUM(oi.sale_price) / NULLIF(SUM(o.num_of_item), 0), 2) AS average_price
FROM `bigquery-public-data.thelook_ecommerce.orders` o
JOIN 
    `bigquery-public-data.thelook_ecommerce.order_items` oi ON o.order_id = oi.order_id
JOIN 
    `bigquery-public-data.thelook_ecommerce.products` p ON oi.product_id = p.id
GROUP BY 
    year, month, product_name, product_category
ORDER BY 
    year, month, total_sales DESC;

SELECT * FROM report_monthly_orders_product_agg LIMIT 1000;
```
> Penjelasan:
1. CREATE TEMP TABLE report_monthly_orders_product_agg: Membuat temporary table bernama `report_monthly_orders_product_agg` yang akan menyimpan hasil aggregasi data penjualan bulanan produk. Tabel ini hanya akan tersedia selama sesi SQL berjalan dan otomatis dihapus setelah sesi berakhir.
3. SELECT:
    - DATE_TRUNC(o.created_at, YEAR) AS year: Mengambil tanggal pembuatan order (`created_at`) dari tabel `orders` dan mengubahnya menjadi tahun untuk agregasi tahunan. 
    - DATE_TRUNC(o.created_at, MONTH) AS month: Mengambil tanggal pembuatan order dan mengubahnya menjadi bulan untuk agregasi bulanan.
    - p.name AS product_name: Mengambil nama produk dari tabel `products` untuk mengetahui produk apa yang terjual.
    - p.category AS product_category: Mengambil kategori produk dari tabel `products` untuk mengetahui kategori produk yang terjual.
    - COUNT(DISTINCT oi.order_id) AS total_orders: Menghitung jumlah transaksi unik untuk setiap produk di setiap bulan, dengan `DISTINCT` digunakan untuk memastikan bahwa hanya satu order yang dihitung meskipun produk yang sama dipesan lebih dari sekali dalam order tersebut.
    - SUM(oi.sale_price) AS total_sales: Menghitung total penjualan dari produk yang dijual dengan menjumlahkan harga jual setiap item dalam order.
    - SUM(o.num_of_item) AS total_items_sold: Menghitung total jumlah item yang terjual berdasarkan jumlah unit yang tercatat dalam tabel `orders`.
    - ROUND(SUM(oi.sale_price) / NULLIF(SUM(o.num_of_item), 0), 2) AS average_price: Menghitung rata-rata harga per unit produk yang terjual. `NULLIF(SUM(o.num_of_item), 0)` digunakan untuk menghindari pembagian dengan nol jika tidak ada item yang terjual, sehingga menghindari kesalahan perhitungan.
4. FROM \`bigquery-public-data.thelook_ecommerce.orders\` o: Data utama yang digunakan berasal dari tabel `orders` dalam dataset `thelook_ecommerce`.
5. JOIN \`bigquery-public-data.thelook_ecommerce.order_items\` oi ON o.order_id = oi.order_id: Menggabungkan data dari tabel `orders` dengan tabel `order_items` berdasarkan `order_id` yang ada pada kedua tabel tersebut.
6. JOIN \`bigquery-public-data.thelook_ecommerce.products\` p ON oi.product_id = p.id: Menggabungkan data dari tabel `order_items` dengan tabel `products` berdasarkan `product_id` yang ada pada kedua tabel tersebut.
7. GROUP BY year, month, product_name, product_category: Mengelompokkan data berdasarkan tahun, bulan, nama produk, dan kategori produk. Ini memungkinkan perhitungan yang dilakukan di `SELECT` (seperti total order, total penjualan, dan rata-rata harga / aggregation) dihitung untuk setiap kombinasi kelompok tersebut.
8. ORDER BY year, month, total_sales DESC: Hasil query akan diurutkan berdasarkan tahun dan bulan, dan kemudian berdasarkan total penjualan (`total_sales`) secara menurun. Dengan cara ini, produk dengan penjualan tertinggi akan ditampilkan terlebih dahulu di setiap bulan.
9. SELECT * FROM report_monthly_orders_product_agg LIMIT 1000: Menampilkan hasil aggregasi data penjualan produk bulanan yang telah dihitung sebelumnya. `LIMIT 1000` digunakan untuk membatasi jumlah baris yang ditampilkan agar tidak terlalu banyak.