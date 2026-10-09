# GPRIDE MUSIC

Pemutar musik Android native (Kotlin + Jetpack Compose + Material 3). Pemutaran lokal, tanpa login, tanpa server.

## Status
- Tahap 1: fondasi + build otomatis (selesai).
- Tahap 2: pemutar inti (Media3 + MediaSessionService, daftar lagu MediaStore, pencarian, mini-player, pemutar penuh, notifikasi media). Perlu diuji di perangkat.
- Berikutnya: Tahap 3 (playlist, favorit, antrean, tema, sleep timer, pemulihan sesi).
- Rencana lengkap dan status: [RENCANA.md](RENCANA.md)

## Struktur
Datar: kode di `src/`, tes di `test/`, resource di `res/`, manifest dan Gradle di root. Tambah file baru langsung ke `src/` tanpa subfolder.

## Mengubah nama / package
Edit `gradle.properties`:
- `app.name` : nama aplikasi di launcher
- `app.id`   : applicationId

## Build dari HP
1. Edit kode di GitHub (browser), commit ke `main`.
2. Buka tab **Actions** → **Android Build** (otomatis jalan, atau **Run workflow**).
3. Buka run yang berhasil → bagian **Artifacts** → unduh `gpride-music-debug-apk`.
4. Ekstrak zip, pasang `app-debug.apk` (izinkan instalasi dari sumber ini jika diminta).

Artifact dihapus otomatis setelah 14 hari (sesuai `retention-days` di workflow dan pengaturan repository). Unduh sebelum kedaluwarsa.
