# Simulasi dan Optimasi Aliran Daya Jaringan Distribusi IEEE 33-Bus Menggunakan Pandapower

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python)
![Pandapower](https://img.shields.io/badge/Pandapower-Electric%20Grid%20Analysis-orange)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter)
![Course](https://img.shields.io/badge/Matkul-Pemrograman%20Komputer-success)
![Institution](https://img.shields.io/badge/UGM-Teknologi%20Rekayasa%20Elektro-yellow)

Repository ini berisi simulasi analisis aliran daya (*load flow analysis*) dan rekonfigurasi jaringan distribusi tenaga listrik standar **IEEE 33-Bus** dengan penambahan *tie-line* menggunakan library Python **Pandapower**. Proyek ini disusun sebagai laporan proyek akhir mata kuliah **Pemrograman Komputer**, Program Studi Sarjana Terapan Teknologi Rekayasa Elektro (TRE), Sekolah Vokasi, Universitas Gadjah Mada.

---

## Informasi Akademik

* **Mata Kuliah:** Pemrograman Komputer (Progkom)
* **Kelas:** RE2A2
* **Institusi:** Departemen Teknik Elektro dan Informatika, Sekolah Vokasi, Universitas Gadjah Mada
* **Dosen Pengampu:** Muhammad Rifqi Al Fauzan, S.Si., M.Si.
* **Asisten Dosen:** 
  1. Wildy Zhalifunnas
  2. Shiddiq Ahmad Fauzan
* **Penyusun:**
  1. **Farrel Amaris Ipaenen** (NIM: 24/538573/SV/24512)
  2. **Miftahul Rizqi** (NIM: 24/538915/SV/24549)

---

## Latar Belakang & Tujuan

Jaringan distribusi radial standar IEEE 33-Bus sering menghadapi masalah penurunan tegangan (*voltage drop*) yang signifikan pada bus-bus di ujung percabangan (seperti Bus 18, 22, 25, dan 33). Kondisi ini dapat menurunkan kualitas daya dan keandalan sistem distribusi kelistrikan.

Tujuan utama dari proyek ini adalah:
1. Memodelkan sistem tenaga listrik IEEE 33-Bus secara komputasional menggunakan pustaka **Pandapower**.
2. Melakukan simulasi aliran daya (*power flow study*) sebelum optimasi untuk memetakan bus-bus yang mengalami *under-voltage*.
3. Menerapkan skenario rekonfigurasi sistem dengan penambahan **Tie-Line** pada titik-titik strategis (Line 2-15 dan Line 19-29).
4. Menganalisis perbaikan profil tegangan sistem serta menghitung reduksi nilai deviasi tegangan menggunakan metrik **Root Mean Square Error (RMSE)** terhadap tegangan referensi $1.02\text{ pu}$.

---

## Diagram Satu Garis (Single Line Diagram)

### 1. Single Line Diagram Sebelum Optimasi
Pada kondisi awal (radial tanpa tie-line tambahan), bus di ujung lateral mengalami jatuh tegangan terbesar karena impedansi saluran kumulatif dan pembebanan.

![SLD Sebelum Optimasi](assets/images/sld-sebelum-optimasi.jpg)

### 2. Single Line Diagram Setelah Optimasi
Dengan menambahkan tie-line antara **Bus 2 ke Bus 15** dan **Bus 19 ke Bus 29**, daya dapat dialirkan melalui jalur alternatif, secara drastis mengurangi impedansi ekivalen ke bus-bus beban berat.

![SLD Setelah Optimasi](assets/images/sld-setelah-optimasi.jpg)

---

## Metodologi & Penjelasan Program

Pemodelan sistem dibangun melalui tahapan-tahapan kode Python berbasis modul `pandapower`:

```
              ┌────────────────────────────────┐
              │   Inisialisasi Empty Network   │
              │     pp.create_empty_network    │
              └───────────────┬────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │   Definisi Bus & Ext Grid      │
              │ 132 kV Grid, 12.66 kV Bus 1-33 │
              └───────────────┬────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │      Trafo Step-Down           │
              │ 132 kV / 12.66 kV, 40 MVA      │
              └───────────────┬────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │    Definisi Saluran (Lines)    │
              │ R, X, Max Current per km       │
              └───────────────┬────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │    Definisi Beban (Loads)      │
              │ P (MW) & Q (MVAR) per Bus      │
              └───────────────┬────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │     Aliran Daya (Power Flow)   │
              │          pp.runpp(net)         │
              └───────────────┬────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │   Analisis Profil & RMSE       │
              └────────────────────────────────┘
```

1. **Pembuatan Bus & External Grid:**
   * Grid eksternal dihubungkan pada Bus 0 dengan tegangan sumber $V_{\text{ext}} = 1.02\text{ pu}$ pada level $132\text{ kV}$.
   * Bus 1 hingga 33 didefinisikan pada taraf tegangan nominal $12.66\text{ kV}$.
2. **Transformator Daya:**
   * Menghubungkan Bus 0 ($132\text{ kV}$) ke Bus 1 ($12.66\text{ kV}$) dengan kapasitas transformator $40\text{ MVA}$.
3. **Pemberian Beban (Loads):**
   * Tiap bus diberikan beban aktif ($P$) dan reaktif ($Q$) sesuai dengan data standar IEEE 33-Bus ($Q = P \times \tan \theta$, dengan $\tan \theta \approx 0.618$).
4. **Konfigurasi Saluran & Tie-Line:**
   * Parameter impedansi resistif ($R$) dan reaktif ($X$) per kilometer didefinisikan menggunakan fungsi `pp.create_line_from_parameters`.
   * Pada jaringan sesudah optimasi (`net2`), ditambahkan 2 saluran interkoneksi (*tie-lines*):
     - Saluran Bus 2 – Bus 15
     - Saluran Bus 19 – Bus 29
5. **Eksekusi Aliran Daya:**
   * Simulasi diselesaikan menggunakan solver algoritma Newton-Raphson melalui fungsi `pp.runpp(net)`.

---

## Analisis Hasil Simulasi

### 1. Perbandingan Profil Tegangan (Voltage Profile)
Grafik berikut menampilkan perbandingan profil tegangan antara kondisi sebelum optimasi (garis merah) dan setelah optimasi (garis biru):

![Perbandingan Profil Tegangan](assets/images/perbandingan-profil-tegangan.png)

### 2. Grafik Peningkatan Tegangan per Bus
Dampak penambahan *tie-line* terhadap peningkatan magnitudo tegangan di setiap bus jaringan:

![Analisis Perbandingan Tegangan](assets/images/analisis-profil-dan-perubahan-tegangan.png)

### 3. Evaluasi Kuantitatif Deviasi Tegangan (RMSE)

Kualitas profil tegangan diukur menggunakan metrik *Root Mean Square Error* (RMSE) terhadap tegangan referensi sistem $V_{\text{ref}} = 1.02\text{ pu}$:

$$\text{RMSE} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} \left(V_{i} - V_{\text{ref}}\right)^{2}}$$

| Kondisi Jaringan | Nilai RMSE (p.u.) | Keterangan |
|---|---|---|
| **Sebelum Optimasi (`net1`)** | **0.09599** | Terjadi *drop* tegangan ekstrem pada bus-bus hilir |
| **Setelah Optimasi (`net2`)** | **0.05213** | Profil tegangan naik mendekati nilai nominal referensi |
| **Persentase Perbaikan** | **~45.7%** | Reduksi deviasi tegangan secara signifikan |

**Kesimpulan:** Penambahan *tie-line* terbukti sangat efektif menstabilkan tegangan kerja jaringan distribusi, meniadakan bus kritis dengan tegangan di bawah batas toleransi operasional, serta meningkatkan kapasitas transmisi daya lokal.

---

## Struktur Direktori Repository

```
pandapower-ieee33/
├── README.md                                          # Dokumentasi teknis lengkap dan analisis
├── .gitignore                                         # Filter cache python dan checkpoint jupyter
├── notebooks/
│   └── ieee33-pandapower-optimization.ipynb           # Jupyter Notebook lengkap dengan simulasi & plot
└── assets/
    └── images/
        ├── sld-sebelum-optimasi.jpg                  # Single Line Diagram kondisi awal
        ├── sld-setelah-optimasi.jpg                  # Single Line Diagram dengan tie-line
        ├── perbandingan-profil-tegangan.png           # Plot profil tegangan bus
        └── analisis-profil-dan-perubahan-tegangan.png # 2-subplot analisis selisih tegangan
```

---

## Panduan Menjalankan Program

### 1. Kebutuhan Sistem (Prerequisites)
Pastikan Python versi 3.8 atau lebih baru telah terpasang.

### 2. Instalasi Pustaka
Pasang pustaka yang diperlukan melalui terminal:
```bash
pip install pandapower matplotlib pandas numpy jupyter
```

### 3. Menjalankan Jupyter Notebook
Jalankan Jupyter Notebook dan buka berkas simulasi:
```bash
jupyter notebook notebooks/ieee33-pandapower-optimization.ipynb
```

Atau jalankan melalui Google Colab melalui tautan notebook terkait.

---

## Lisensi & Hak Cipta

Proyek ini dikembangkan untuk keperluan akademik dan penelitian di Program Studi Sarjana Terapan Teknologi Rekayasa Elektro, Sekolah Vokasi, Universitas Gadjah Mada.
