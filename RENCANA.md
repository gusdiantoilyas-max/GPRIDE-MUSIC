# Rencana Proyek: GPRIDE MUSIC (diperbarui)

Pemutar musik Android native untuk penggunaan pribadi. Kotlin, Jetpack Compose, Material 3, Media3. Semua data lokal: tanpa login, server, cloud, analitik, atau iklan. Build dan distribusi lewat GitHub Actions, APK diunduh dari HP.

Pembaruan terakhir: 9 Oktober 2026.

## 1. Ringkasan status

| Tahap | Isi | Status kode | Status verifikasi |
|---|---|---|---|
| 1 | Fondasi, Gradle, Compose, GitHub Actions | Selesai ditulis | Build belum terkonfirmasi |
| 2 | Pemutar inti: Media3, service, daftar lagu, mini-player, pemutar penuh | Selesai ditulis | Belum dibangun, belum diuji di perangkat |
| 3 | Playlist, favorit, antrean, tema, sleep timer, pemulihan sesi | Belum dimulai | - |
| 4 | Lirik .lrc, penyempurnaan UI, edge case, optimasi | Belum dimulai | - |
| 5 | Visualizer audio real-time (addendum 5A) | Belum dimulai (hanya dirancang) | - |

Catatan jujur: seluruh kode ditulis tanpa bisa menjalankan Gradle (sandbox tanpa internet). "Selesai ditulis" tidak berarti "berhasil dibangun". Bukti yang sah hanya: run GitHub Actions hijau, APK terpasang, dan uji di HP.

## 2. Keputusan arsitektur (tetap)

- Android native, Kotlin, Compose + Material 3. Tanpa WebView/PWA.
- Pemutaran: Media3 ExoPlayer di dalam MediaSessionService, satu-satunya pemilik player. UI memakai MediaController.
- Koleksi: MediaStore. SAF hanya untuk file/folder manual (nanti).
- Data lokal: Room (playlist, urutan) dan DataStore (preferensi) pada Tahap 3.
- Dilarang: login, backend, Neon, streaming, upload musik, analitik, iklan, premium.

## 3. Versi yang dipakai

| Komponen | Versi |
|---|---|
| Gradle (di CI) | 8.9 |
| Android Gradle Plugin | 8.7.3 |
| Kotlin + plugin Compose | 2.0.21 |
| Compose BOM | 2024.10.01 |
| Media3 | 1.4.1 |
| JDK | 17 |
| minSdk / target / compileSdk | 26 / 35 / 35 |

Versi dipilih dari pengetahuan saya dan belum divalidasi oleh build nyata. Jika build gagal, mulai dari sini.

## 4. Tahap yang sudah dijalankan

### Tahap 1: Fondasi
- Struktur proyek Gradle (Kotlin DSL), modul tunggal di root, package `com.srin.musicplayer`.
- Nama aplikasi dan applicationId bisa diubah di `gradle.properties` (`app.name`, `app.id`).
- Tema terang, gelap, dynamic color; edge-to-edge.
- Workflow `.github/workflows/android-build.yml`:
  - Pemicu: push ke `main` dan `workflow_dispatch`; batas 30 menit; konkurensi dibatasi.
  - Langkah: JDK 17, Gradle 8.9 + cache, unit test, lint, `assembleDebug`, cek keberadaan APK, unggah artifact `gpride-music-debug-apk` (retensi 14 hari).
  - Gagal dengan jelas jika kompilasi/test gagal atau APK tidak ada.

### Tahap 2: Pemutar inti
- `PlaybackService`: ExoPlayer + MediaSession, audio focus, penanganan headset dicabut, wake mode, foreground service `mediaPlayback`. Notifikasi media, layar kunci, headset, dan Bluetooth lewat MediaSession Media3.
- File rusak/tidak didukung: dilompati dengan batas anti-loop, pesan Toast ke pengguna.
- `MusicRepository`: baca MediaStore (judul, artis, album, durasi, tanggal), tahan terhadap metadata kosong dan izin dicabut.
- Daftar lagu dengan pencarian (judul/artis/album), urut judul, sampul album, pembaruan otomatis saat koleksi berubah.
- Izin: audio + notifikasi, layar petunjuk bila ditolak, tombol ke pengaturan aplikasi.
- Mini-player (sampul, judul, artis, play/pause, next, progres) dan pemutar penuh (sampul besar, seek, prev/next, acak, ulang). Posisi di-polling hanya saat UI terlihat.
- Unit test: filter pencarian dan format waktu.

## 4b. Struktur repository (datar)

Modul tunggal di root, tanpa folder `app/` dan tanpa folder package bertingkat (Kotlin tidak mewajibkan folder sama dengan package):

```
.github/workflows/android-build.yml
src/            semua file .kt (datar)
test/           unit test
res/values/     warna dan tema
res/drawable/   ikon aplikasi
AndroidManifest.xml
build.gradle.kts, settings.gradle.kts, gradle.properties
README.md, RENCANA.md, .gitignore
```

Aturan: file baru ditaruh di `src/` (atau `test/`) tanpa membuat subfolder baru; nama file harus unik.

## 4c. Integrasi Codebase GP (MCP)

- Project `gpride-music` ("GPRIDE MUSIC") sudah didaftarkan di Codebase GP (commit `d02125d`): `lib/projects.js`, kartu dashboard, halaman `project-gpride-music.html`, dan tautan navigasi. MCP Ai worker membaca daftar project dari `lib/projects.js`, jadi project ini otomatis muncul di tool MCP setelah deploy.
- Belum aktif sampai 3 env var diisi di Vercel project Codebase GP: `GPRIDEMUSIC_REPO_OWNER`, `GPRIDEMUSIC_REPO_NAME`, `GPRIDEMUSIC_REPO_BRANCH` (opsional, default `main`). Token yang dipakai: `GITHUB_TOKEN_GUSDIANTO` (asumsi repo di akun yang sama dengan GPRIDE; ganti `tokenEnvVar` jika beda akun).

## 5. Penyimpangan dari rancangan awal

| Rancangan awal | Kenyataan sekarang | Alasan |
|---|---|---|
| Gradle wrapper | Tidak ada; CI memakai `setup-gradle` dengan Gradle 8.9 | Jar wrapper perlu diunduh; tidak bisa dari sandbox |
| Lint menggagalkan build | Lint tidak menggagalkan build | Agar APK pertama tidak terhambat peringatan kecil; ketatkan nanti |
| Navigasi bawah 4 tab | Belum ada | Halaman lain belum ada; hindari tombol palsu |
| Sortir banyak kriteria | Hanya judul | Ditunda ke Tahap 3-4 |
| Antrean dapat diubah | Belum | Tahap 3 |
| Pemulihan sesi penuh | Hanya selama service hidup | Penyimpanan sesi masuk Tahap 3 |
| Icon library | Ikon digambar sendiri | Hindari dependency icons-extended yang berat |

## 6. Rencana tahap berikutnya

### Tahap 3: Data dan pengalaman pengguna
1. Room: playlist, urutan lagu, favorit. DataStore: tema, pengaturan pemutaran, lagu/posisi terakhir.
2. Navigasi bawah: Beranda, Koleksi, Playlist, Pengaturan (hanya setelah halamannya nyata).
3. Antrean: lihat, ubah urutan, hapus item; tidak pernah menghapus file asli.
4. Tema: terang, gelap, ikuti sistem, warna aksen.
5. Sleep timer di service (menit tertentu, akhir lagu, batal, sisa waktu), tidak bergantung UI.
6. Pemulihan sesi: simpan antrean dan posisi, pulihkan saat aplikasi dibuka.
7. Tangani URI hilang/izin tidak berlaku.

### Tahap 4: Lanjutan
- Lirik .lrc (pilih file lewat SAF, sorot baris sesuai waktu, lirik biasa tanpa timestamp), referensi disimpan lokal.
- Halaman Album, Artis, Favorit; sortir lengkap.
- Edge case, optimasi performa, uji perangkat nyata.

### Tahap 5: Visualizer (addendum 5A)
Dikerjakan hanya setelah pemutar inti terbukti stabil di HP.

Temuan desain (evaluasi API, belum diimplementasikan):
- Opsi A, `android.media.audiofx.Visualizer`: membutuhkan izin RECORD_AUDIO, bergantung pada audio session id ExoPlayer (milik service, jadi perlu perintah sesi khusus), dan perilakunya bervariasi antar perangkat.
- Opsi B (disarankan), `AudioProcessor` kustom di ExoPlayer (mis. TeeAudioProcessor) untuk menerima buffer PCM yang sedang diputar, lalu FFT sendiri untuk bass/mid/treble. Tanpa izin tambahan, tanpa menyalin audio, data benar-benar dari audio yang diputar.
- Data visual dikirim dari service ke UI lewat objek in-process (satu proses), bukan IPC, dengan throttling ~30 fps dan smoothing.
- Fallback jika spektrum gagal: amplitudo dari PCM yang sama, dengan label jujur bahwa itu bukan analisis frekuensi.
- Mode: Spectrum Bars, Waveform, Circular Pulse, Particle Flow, Neon Equalizer, Minimal Pulse; pengaturan glow, intensitas, hemat baterai, warna dari sampul; berhenti saat UI tidak terlihat.
- Prioritas rilis: mulai dari Spectrum Bars dan Minimal Pulse, sisanya bertahap.

## 7. Kriteria penerimaan dan status uji

Uji otomatis (CI):
- [ ] Kompilasi berhasil
- [ ] Unit test lulus
- [ ] APK dihasilkan dan bisa diunduh

Uji perangkat nyata (manual):
- [ ] Izin dan daftar lagu terisi
- [ ] Putar, jeda, lompat, seek
- [ ] Musik jalan saat pindah aplikasi dan layar mati
- [ ] Notifikasi dan kontrol layar kunci
- [ ] Pause saat headset dicabut; interupsi telepon
- [ ] File rusak tidak membuat crash
- [ ] (Tahap 3) Playlist/favorit bertahan setelah restart; sleep timer saat layar mati; tema
- [ ] (Tahap 5) Visualizer bereaksi ke audio nyata, tidak membebani CPU/baterai

Build hijau tidak membuktikan pemutaran layar mati berfungsi di semua perangkat.

## 8. Risiko dan hal terbuka

- Build pertama belum terkonfirmasi; versi library bisa perlu disesuaikan.
- Perilaku notifikasi, foreground service, dan optimasi baterai berbeda antar vendor HP (mis. penghemat daya agresif); perlu uji di perangkat Anda.
- Memuat seluruh daftar lagu sebagai antrean cukup untuk koleksi biasa; koleksi sangat besar perlu dioptimalkan.
- Visualizer: pilihan Opsi A vs B perlu diputuskan sebelum Tahap 5 (rekomendasi: B).

## 9. Alur kerja dari HP

1. Edit kode di GitHub (browser), commit ke `main`.
2. Tab Actions, Android Build (otomatis atau Run workflow).
3. Buka run yang hijau, unduh artifact `gpride-music-debug-apk`, ekstrak, pasang APK.
4. Artifact kedaluwarsa setelah 14 hari; unduh sebelum itu. Jangan commit keystore atau kredensial.
