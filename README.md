## Hasil dan Analisis

### 1. Ringkasan Hasil Evaluasi

Evaluasi dilakukan pada data uji (*test set*) yang tidak pernah digunakan selama proses pelatihan, yaitu sebanyak 2.023 gambar (1.011 kucing dan 1.012 anjing). Kedua model dilatih dan diuji pada pembagian data yang identik sehingga perbandingannya adil.

| Model | Akurasi Uji | Precision (macro) | Recall (macro) | F1-score (macro) | Jumlah Salah Prediksi |
|---|---|---|---|---|---|
| CNN dari nol | 76,27% | 76,4% | 76,3% | 76,2% | 480 dari 2.023 |
| MobileNetV2 (Transfer Learning) | 98,57% | 98,6% | 98,6% | 98,6% | 29 dari 2.023 |

| Model | Jumlah Parameter | Waktu Pelatihan (detik) |
|---|---|---|
| CNN dari nol | [isi dari output] | [isi dari output] |
| MobileNetV2 (Transfer Learning) | [isi dari output] | [isi dari output] |

Model transfer learning unggul dengan selisih akurasi sekitar 22,3 poin persentase. Jumlah gambar yang salah diprediksi turun dari 480 menjadi 29, atau berkurang sekitar 94%.

### 2. Analisis Confusion Matrix

![Confusion Matrix CNN](results/confusion_matrix_cnn.png)
![Confusion Matrix MobileNetV2](results/confusion_matrix_tl.png)

**CNN dari nol.** Dari 1.011 gambar kucing, 805 dikenali dengan benar dan 206 diprediksi sebagai anjing. Dari 1.012 gambar anjing, 738 dikenali dengan benar dan 274 diprediksi sebagai kucing. Recall kelas kucing sebesar 79,6% dan kelas anjing sebesar 72,9% menunjukkan model cenderung lebih sering mengira anjing sebagai kucing. Kesalahan yang cukup besar pada kedua arah menandakan bahwa fitur visual yang dipelajari model belum cukup diskriminatif untuk membedakan kedua kelas secara konsisten.

**MobileNetV2.** Hanya 15 gambar kucing yang diprediksi sebagai anjing dan 14 gambar anjing yang diprediksi sebagai kucing. Recall kedua kelas hampir sama (98,5% untuk kucing dan 98,6% untuk anjing), sehingga model tidak menunjukkan bias terhadap salah satu kelas. Kesalahan yang jumlahnya kecil dan seimbang ini mengindikasikan bahwa fitur hasil pelatihan awal pada ImageNet sangat sesuai untuk membedakan kucing dan anjing.

### 3. Analisis Kurva Pelatihan

![Kurva Pelatihan CNN](results/kurva_cnn.png)

Kurva pelatihan CNN dari nol memperlihatkan dua hal. Pertama, akurasi pelatihan naik perlahan dari sekitar 60% hingga sekitar 80% dan belum mencapai titik jenuh yang tinggi, sehingga kapasitas model dalam mempelajari pola data memang terbatas. Kedua, akurasi dan loss validasi sangat tidak stabil. Akurasi validasi berfluktuasi antara sekitar 49% dan 77%, sedangkan loss validasi melonjak tajam pada beberapa epoch (hingga sekitar 1,8 dan 2,4) meskipun loss pelatihan menurun secara stabil.

Pola ini lebih mencerminkan ketidakstabilan pelatihan daripada overfitting klasik, karena akurasi pelatihan sendiri masih rendah. Beberapa faktor yang mungkin berkontribusi adalah learning rate yang relatif tinggi (1e-3) untuk model yang dilatih dari bobot acak, penggunaan augmentasi data yang membuat kondisi pelatihan dan validasi berbeda, serta statistik *Batch Normalization* yang belum stabil. Mekanisme *EarlyStopping* menghentikan pelatihan pada epoch ke-11 dan mengembalikan bobot terbaik, yaitu dari epoch dengan loss validasi terendah (sekitar 0,49). Akurasi uji akhir sebesar 76,27% konsisten dengan akurasi validasi terbaik, sehingga tidak ada indikasi kebocoran data.

Kurva pelatihan MobileNetV2: [deskripsikan dari grafik, misalnya akurasi validasi sudah tinggi sejak epoch awal, kurva train dan val berdekatan, dan loss menurun stabil].

### 4. Analisis Contoh Prediksi

![Sampel Prediksi CNN](results/prediksi_cnn.png)
![Perbandingan Prediksi](results/perbandingan_model.png)

Pada sampel prediksi CNN dari nol, tingkat keyakinan model pada prediksi yang benar bervariasi cukup lebar (sekitar 62% hingga 99%), sedangkan prediksi yang salah sering memiliki keyakinan rendah hingga sedang (sekitar 52% hingga 64%), meskipun ada pula kesalahan dengan keyakinan tinggi (sekitar 80% hingga 90%). Keyakinan yang rendah pada banyak prediksi menandakan model belum memiliki batas keputusan yang tegas.

Pada perbandingan kedua model terhadap gambar yang sama, MobileNetV2 memprediksi seluruh sampel dengan benar dan dengan keyakinan tinggi (sekitar 92% hingga 100%). Sebaliknya, CNN dari nol salah pada dua dari delapan sampel, termasuk satu kesalahan dengan keyakinan sekitar 89% pada gambar anjing putih bertelinga tegak yang secara visual menyerupai kucing. Hal ini menunjukkan bahwa kesalahan CNN dari nol tidak selalu disertai keraguan, sehingga tingkat keyakinan model tersebut kurang dapat diandalkan.

### 5. Analisis Kesalahan Prediksi

![Salah Prediksi CNN](results/salah_cnn.png)
![Salah Prediksi MobileNetV2](results/salah_tl.png)

Gambar yang salah diprediksi oleh CNN dari nol umumnya memiliki kondisi visual yang menyulitkan, yaitu kucing yang sedang digendong atau tertutup tangan manusia, kucing di dalam kandang dengan jeruji, latar belakang yang ramai atau berwarna mencolok, pose tidak biasa, serta anak kucing yang berukuran kecil dalam frame. Pada kondisi tersebut model mudah terpengaruh oleh latar belakang dan objek lain, bukan oleh ciri khas hewan itu sendiri.

Kesalahan MobileNetV2 jumlahnya sangat kecil, dan sebagian di antaranya tampaknya tidak sepenuhnya merupakan kegagalan model. Beberapa gambar berlabel kucing yang salah diprediksi ternyata berisi logo organisasi, papan teks bertuliskan nomor telepon, atau bahkan gambar yang tampak seperti anjing. Temuan ini menunjukkan adanya gambar tidak relevan dan kemungkinan label yang keliru (*label noise*) pada dataset. Dengan demikian, akurasi MobileNetV2 sebesar 98,57% sudah mendekati batas atas yang dapat dicapai pada dataset ini tanpa pembersihan data. Sisa kesalahan lainnya terjadi pada kucing yang berada di dalam kandang atau digendong orang, serupa dengan pola kesalahan CNN dari nol.

### 6. Pembahasan Perbandingan Kedua Model

Perbedaan performa yang besar antara kedua model dapat dijelaskan oleh beberapa faktor:

1. **Sumber pengetahuan awal.** CNN dari nol memulai pelatihan dari bobot acak, sehingga seluruh fitur harus dipelajari hanya dari sekitar 8.000 gambar latih. MobileNetV2 sudah mempelajari fitur visual umum (tepi, tekstur, bentuk, bagian tubuh hewan) dari jutaan gambar ImageNet, sehingga hanya perlu melatih lapisan klasifikasi di bagian akhir.
2. **Ukuran data.** Dataset sebesar ini tergolong kecil untuk melatih CNN yang cukup ekspresif dari awal, sedangkan transfer learning memang dirancang untuk situasi data terbatas.
3. **Stabilitas dan kecepatan konvergensi.** MobileNetV2 mencapai performa tinggi dengan pelatihan yang lebih singkat dan stabil, sementara CNN dari nol memerlukan lebih banyak epoch dan tetap menunjukkan fluktuasi validasi.
4. **Keandalan keyakinan prediksi.** Prediksi MobileNetV2 lebih tegas dan lebih konsisten dengan kebenarannya dibandingkan CNN dari nol.

### 7. Kesimpulan

Transfer learning dengan MobileNetV2 terbukti jauh lebih efektif daripada CNN dari nol untuk klasifikasi kucing dan anjing pada dataset ini, dengan akurasi uji 98,57% berbanding 76,27%. Hasil ini sejalan dengan teori bahwa transfer learning memberikan keuntungan terbesar ketika data latih terbatas. CNN dari nol tetap berguna sebagai model pembanding (*baseline*) yang menunjukkan seberapa besar kontribusi pengetahuan awal dari ImageNet.

### 8. Keterbatasan dan Saran Pengembangan

**Keterbatasan:**
- Eksperimen hanya dijalankan satu kali dengan satu nilai *random seed*, sehingga variasi antar-percobaan belum diukur.
- Tidak dilakukan penyetelan hyperparameter (*hyperparameter tuning*).
- Pada MobileNetV2, hanya lapisan klasifikasi yang dilatih (basis dibekukan), tanpa *fine-tuning*.
- Dataset memuat sejumlah gambar tidak relevan dan kemungkinan label keliru.

**Saran pengembangan:**
- Menurunkan learning rate dan menambahkan penjadwalan (*learning rate scheduling*) untuk menstabilkan pelatihan CNN dari nol.
- Memperdalam arsitektur CNN atau menambah variasi augmentasi data.
- Melakukan *fine-tuning* pada beberapa lapisan terakhir MobileNetV2.
- Membersihkan dataset dari gambar tidak relevan, lalu mengulang eksperimen dengan beberapa seed untuk melaporkan rata-rata dan simpangan baku.
