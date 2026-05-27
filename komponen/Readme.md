# Sistem Monitoring Parkir (Counter) Dengan Sensor Ultrasonik

Proyek Tugas Besar — Kelas B, Kelompok 4

**Anggota:**
- Finda Wulan Febrianti (H1H024055)
- Muhammad Raihan Izzuddin Ismadi (H1H024056)
- Farhan Nur Sahid (H1H024057)
- Reva Aura Ramadhani (H1H024059)
- Ahmadin Irfan Fauzi (H1H024060)
- Ramandhanu Isnaera Ahnaf Wibawa (H1H024061)

---

## Gambaran Umum

Sistem ini memantau ketersediaan slot parkir secara real-time menggunakan metode **counter** — menghitung kendaraan yang masuk dan keluar melalui sensor ultrasonik HC-SR04 di pintu masuk dan pintu keluar. Jumlah kendaraan di dalam area parkir dihitung secara otomatis dan ditampilkan pada LCD, serta palang pintu dikendalikan berdasarkan kapasitas yang tersisa.

---

## Daftar Komponen

| No | Komponen | Jumlah | Fungsi |
|----|----------|--------|--------|
| 1 | **Arduino Uno / ESP32** | 1 | Mikrokontroler utama; memproses data counter dan mengendalikan seluruh sistem |
| 2 | **Sensor Ultrasonik HC-SR04** | 2 | Satu di pintu **masuk**, satu di pintu **keluar** — mendeteksi kendaraan yang melintas untuk menambah/mengurangi counter |
| 3 | **Servo Motor SG90** | 1–2 | Menggerakkan palang pintu masuk (dan/atau keluar) secara otomatis |
| 4 | **LCD 16x2 + Modul I2C** | 1 | Menampilkan jumlah slot tersedia dan status parkir secara real-time |
| 5 | **Buzzer** | 1 | Memberikan peringatan audio saat kapasitas parkir penuh |
| 6 | **Kabel Jumper** | Secukupnya | Menghubungkan komponen pada breadboard dan mikrokontroler |
| 7 | **Breadboard** | 1 | Papan prototyping untuk merangkai rangkaian tanpa solder |
| 8 | **Power Supply** | 1 | Sumber daya untuk seluruh sistem |

---

## Spesifikasi Komponen

### Sensor Ultrasonik HC-SR04
- Digunakan **2 unit**: 1 di pintu masuk, 1 di pintu keluar
- Jangkauan deteksi: 2 cm – 400 cm
- Kendaraan terdeteksi jika jarak terukur ≤ ambang batas (misal < 20 cm)
- Sensor masuk → counter **+1** (kendaraan masuk)
- Sensor keluar → counter **-1** (kendaraan keluar)

### Servo Motor SG90
- Sudut buka palang: **90°**
- Sudut tutup palang: **0°**
- Menutup otomatis setelah kendaraan melewati sensor

### LCD 16x2 + I2C
Pesan yang ditampilkan:
- `Sistem Siap` — saat inisialisasi
- `Slot: XX / YY` — jumlah slot tersedia dari total kapasitas (real-time)
- `Selamat Datang` — saat kendaraan masuk dan palang membuka
- `PARKIR PENUH` — saat counter mencapai kapasitas maksimum

---

## Logika Counter

```
Kapasitas maksimum = N slot

Kendaraan terdeteksi di sensor MASUK:
  → Jika counter < N : buka palang, counter = counter + 1, tampilkan sisa slot
  → Jika counter = N : palang tetap tutup, LCD "PARKIR PENUH", buzzer berbunyi

Kendaraan terdeteksi di sensor KELUAR:
  → Jika counter > 0 : counter = counter - 1, tampilkan sisa slot
```

---

## Cara Kerja Singkat

1. Sistem menyala dan menginisialisasi semua komponen, counter diset ke 0.
2. Sensor ultrasonik di pintu masuk dan keluar terus membaca jarak secara real-time.
3. Jika kendaraan melintas sensor **masuk** dan kapasitas belum penuh → servo buka palang, counter bertambah, LCD update sisa slot.
4. Jika kendaraan melintas sensor **masuk** dan kapasitas penuh → palang tetap tutup, LCD tampilkan "PARKIR PENUH", buzzer berbunyi.
5. Jika kendaraan melintas sensor **keluar** → counter berkurang, LCD update sisa slot.
6. Sistem terus memantau kedua sensor secara real-time dan mengulang proses.
