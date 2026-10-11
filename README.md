# Eliminasi Gauss dan Dekomposisi LU

Program Python untuk menyelesaikan sistem persamaan linear **A x = b** dengan empat metode numerik. Seluruh algoritma ditulis langsung dari rumusnya, **tanpa NumPy atau library eksternal**, sehingga setiap langkah dapat dibaca, dilacak, dan dihitung jumlah operasinya.

Program ini dibuat untuk membandingkan metode langsung (*direct methods*) dari sisi **ketepatan hasil, jumlah operasi aritmetika, dan waktu eksekusi**.

## Metode yang diimplementasikan

| No | Metode | Fungsi | Pivoting |
|----|--------|--------|----------|
| 1 | Eliminasi Gauss | `gauss()` | Parsial |
| 2 | Gauss-Jordan | `gauss_jordan()` | Parsial |
| 3 | Dekomposisi LU Doolittle | `lu_faktor()`, `lu()` | Tanpa pivoting |
| 4 | LU berpivot (`PA = LU`) | `plu_faktor()`, `plu()` | Parsial |

## Fitur

- Empat metode penyelesaian SPL dalam satu file.
- Tiga studi kasus siap pakai (3×3, 4×4, 5×5) dengan solusi yang sudah diketahui.
- Mode `-v` untuk menampilkan langkah-langkah detail (operasi baris, matriks L dan U, vektor `y`).
- Penghitung operasi (perkalian/pembagian dan penjumlahan/pengurangan) yang bisa diverifikasi terhadap rumus teoretis.
- Benchmark waktu, termasuk skenario banyak ruas kanan (LU hanya difaktorkan sekali).
- Input matriks sendiri dari terminal.
- Deteksi matriks singular dan pivot nol, dengan pesan error yang jelas.

## Persyaratan

- Python 3.6 atau lebih baru
- Tidak ada dependensi eksternal (hanya library standar: `argparse`, `random`, `time`)

## Instalasi

```bash
git clone https://github.com/<username>/<nama-repo>.git
cd <nama-repo>
```

> **Catatan:** Contoh di bawah memakai nama file `spl_solver.py`. Jika file Anda bernama `Eliminasi Gauss dan Dekomposisi LU.py`, ganti namanya atau sesuaikan perintahnya (gunakan tanda kutip karena nama file mengandung spasi).

## Cara pakai

Tanpa argumen, program membuka **menu interaktif**:

```bash
python spl_solver.py
```

Atau jalankan mode tertentu langsung:

```bash
python spl_solver.py kasus        # tiga studi kasus x empat metode
python spl_solver.py kasus -v     # + langkah-langkah detail
python spl_solver.py ops          # verifikasi jumlah operasi terhadap rumus
python spl_solver.py bench        # pengukuran waktu (sekitar 1-2 menit)
python spl_solver.py input        # masukkan matriks sendiri
```

| Mode | Fungsi |
|------|--------|
| `kasus` | Menjalankan 3 studi kasus dengan keempat metode, menampilkan solusi, residu, dan jumlah operasi |
| `ops` | Membandingkan jumlah operasi hasil hitung program dengan rumus teoretis untuk n = 3, 10, 50, 100 |
| `bench` | Mengukur waktu eksekusi (median dari 5 ulangan) untuk berbagai ukuran n dan jumlah ruas kanan k |
| `input` | Meminta ukuran n, matriks A, vektor b, lalu metode yang dipilih |

## Studi kasus

| Kasus | Ukuran | Konteks | Solusi |
|-------|--------|---------|--------|
| 1 | 3×3 | Arus rangkaian listrik | (2, 3, 1) |
| 2 | 4×4 | Produksi furnitur | (10, 20, 15, 5) |
| 3 | 5×5 | Harga buah | (3, 2, 4, 5, 1) |

### Contoh keluaran

```
====================================================================
Kasus 1 (3x3): arus rangkaian listrik    solusi diharapkan (2, 3, 1)
====================================================================

--- Gauss ---
   x = [2.0, 3.0, 1.0]   residu = 7.1e-15   operasi = 28

--- Gauss-Jordan ---
   x = [2.0, 3.0, 1.0]   residu = 3.6e-15   operasi = 33

--- LU Doolittle ---
   x = [2.0, 3.0, 1.0]   residu = 7.1e-15   operasi = 37

--- PA = LU ---
   x = [2.0, 3.0, 1.0]   residu = 7.1e-15   operasi = 28
```

## Memasukkan matriks sendiri

```bash
python spl_solver.py input
```

```
Ukuran n: 3
Masukkan matriks A baris demi baris (pisahkan dengan spasi):
  baris 1: 10 -2 -1
  baris 2: -2 8 -3
  baris 3: -1 -3 9
Vektor b (n angka): 13 17 -2
Metode: 1 Gauss, 2 Gauss-Jordan, 3 LU, 4 PA = LU
Pilih: 1
```

## Memakai sebagai modul

Semua fungsi dapat diimpor dan dipanggil dari kode lain:

```python
from spl_solver import gauss, lu, plu, Cnt

A = [[10, -2, -1], [-2, 8, -3], [-1, -3, 9]]
b = [13, 17, -2]

c = Cnt()                  # penghitung operasi (opsional)
x = gauss(A, b, c)
print(x)                   # [2.0, 3.0, 1.0]
print(c.total)             # 28
```

Setiap fungsi solver memiliki tanda tangan `f(A, b, c=None, v=False)`, dengan `c` penghitung operasi dan `v` penanda mode detail.

## Rumus jumlah operasi

Mode `ops` memverifikasi bahwa jumlah operasi yang terhitung cocok dengan rumus berikut (n = ukuran matriks):

| Komponen | Rumus |
|----------|-------|
| Eliminasi Gauss (total) | `2n³/3 + 3n²/2 − 7n/6` |
| Gauss-Jordan (total) | `n³ + n² − n` |
| Faktorisasi LU Doolittle | `(4n³ + 3n² − n) / 6` |
| Faktorisasi `PA = LU` | `(4n³ − 3n² − n) / 6` |
| Satu kali substitusi (maju + mundur) | `2n² − n` |

Untuk **k** ruas kanan, rasio keunggulan LU terhadap Gauss yang diulang adalah:

```
R(n, k) = k · T_gauss(n) / (T_LU(n) + k · (2n² − n))
```

Artinya, semakin besar `k`, semakin besar pula keuntungan memfaktorkan matriks sekali saja.

## Kapan memakai metode yang mana?

- **Gauss**: pilihan paling hemat untuk satu ruas kanan.
- **Gauss-Jordan**: lebih banyak operasi (orde n³ dengan koefisien lebih besar), cocok untuk tujuan pembelajaran atau mencari invers.
- **LU Doolittle**: efisien bila A yang sama dipakai untuk banyak vektor b, tetapi **gagal bila ada pivot nol** dan kurang stabil tanpa pivoting.
- **`PA = LU`**: sama efisiennya dengan LU dan lebih stabil karena memakai pivoting parsial. Pilihan umum untuk banyak ruas kanan.

## Penanganan error

Program melempar `ZeroDivisionError` dengan pesan yang jelas, misalnya:

- `matriks singular pada kolom k` pada metode berpivot.
- `pivot nol pada U(i,i): LU tanpa pivoting gagal (gunakan PA = LU)` pada LU Doolittle.

## Struktur kode

```
spl_solver.py
├── Cnt                      # penghitung operasi
├── gauss, gauss_jordan      # metode eliminasi
├── lu_faktor, lu            # LU Doolittle
├── plu_faktor, plu          # LU berpivot
├── substitusi               # substitusi maju dan mundur
├── T_*, S_sub, R_teori      # rumus jumlah operasi
└── mode_kasus / mode_ops / mode_bench / mode_input / menu
```

## Lisensi

Tentukan lisensi sesuai kebutuhan Anda (misalnya MIT) dan tambahkan file `LICENSE` di repositori.
