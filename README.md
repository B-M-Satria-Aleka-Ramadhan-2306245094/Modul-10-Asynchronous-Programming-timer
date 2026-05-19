# Modul-10-Asynchronous-Programming-timer

## 1.2 Understanding how it works.
<img width="1042" height="239" alt="image" src="https://github.com/user-attachments/assets/955f36cc-7f75-48df-8aef-49a8aca65543" />
Ketika program dijalankan, teks "hey hey" akan tercetak hampir bersamaan dengan "howdy!", diikuti jeda selama 2 detik sebelum akhirnya teks "done!" muncul di terminal.

Hal ini terjadi karena fungsi spawner.spawn bersifat asinkronus dan non-blocking. Saat fungsi tersebut dipanggil, ia hanya memasukkan blok async ke dalam antrean executor lalu langsung melanjutkan eksekusi ke baris berikutnya (mencetak "hey hey"), tanpa menunggu isi blok asinkronus selesai. Sementara tugas di dalam blok async ditangguhkan (yield) selama 2 detik oleh perintah TimerFuture::...await, alur utama main() terus berjalan hingga selesai. Setelah waktu tunggu 2 detik habis, executor barulah melanjutkan sisa tugas asinkronus untuk mencetak teks "done!".

## 1.3 Multiple Spawn and removing drop

### With Drop Spawner
<img width="1028" height="330" alt="image" src="https://github.com/user-attachments/assets/4b6c4ecc-fc9e-41e0-940d-d3b8a5cded41" />

### Without Drop Spawner
<img width="983" height="336" alt="image" src="https://github.com/user-attachments/assets/2589b6f2-9941-4ac1-9393-acc9fad36053" />

### Analisis Eksperimen: Manajemen Alur Kerja Asinkronus
Eksperimen ini menguji perilaku eksekusi beberapa tugas (task) asinkronus secara simultan serta pengaruh dari keberadaan perintah drop(spawner). Pengujian dilakukan dengan membuat tiga tugas terpisah melalui spawner.spawn(...), di mana setiap tugas mencetak pesan awal, menunda eksekusi selama dua detik menggunakan TimerFuture, lalu mencetak pesan akhir.

### Perbedaan hasil dari kedua skenario yang diuji adalah sebagai berikut:
- Dengan drop(spawner): Executor berhasil menyelesaikan seluruh tugas secara tuntas. Semua pesan "howdy!" dan "done!" tercetak lengkap di terminal sebelum program keluar dengan bersih.

- Tanpa drop(spawner): Program mengalami kendala di mana pesan "done!" tidak selalu selesai diproses secara konsisten, atau program terus berjalan (hang) tanpa henti. Hal ini terjadi karena executor terus bersiap menerima tugas baru tanpa tahu kapan harus berhenti.

### What is the effect of spawning?
Proses spawning berfungsi untuk mendaftarkan sebuah future (tugas asinkronus) ke dalam antrean executor. Pemanggilan spawn() tidak langsung menjalankan tugas tersebut saat itu juga, melainkan hanya menjadwalkannya. Mekanisme ini menjaga program tetap responsif karena alur utama tidak terhambat oleh proses tunggu seperti timer atau operasi I/O.

### What is the spawner for, what is the executor for, what is the drop for?
- Spawner: Bertugas sebagai agen yang memproduksi dan memasukkan tugas-tugas baru ke dalam antrean kerja.
- Executor: Bertindak sebagai mesin penggerak yang mengambil tugas dari antrean dan memprosesnya (polling) hingga selesai.
- Drop(spawner): Berfungsi sebagai sinyal penutup. Perintah ini memberi tahu executor bahwa tidak akan ada lagi tugas baru yang masuk. Tanpa perintah ini, executor akan terjebak dalam kondisi idle (menunggu tanpa batas waktu) meskipun semua tugas terdaftar sudah selesai.

### What is the correlation of all of that?
Ketiga elemen ini membentuk fondasi dari async runtime minimal di Rust yang saling bergantung. Spawner bertindak sebagai penyedia kerja, executor sebagai pelaksana kerja, dan drop(spawner) sebagai penanda bahwa pekerjaan telah selesai total. Jika salah satu manajemen siklus hidup ini pincang—seperti lupa melepas (drop) spawner—sistem asinkronus tidak akan tahu kapan harus menyudahi operasinya, yang akhirnya memicu bug atau deadlock.

