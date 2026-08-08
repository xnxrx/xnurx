# batch20 — origin / address-bar / fullscreen spoofing + UXSS oracles

20 PoC HTML mandiri untuk riset kelas **"apa yang ditampilkan browser ≠ apa yang sebenarnya berlaku"**.
Dibangun mengikuti bentuk-minimum `spoof-forge.md §8.1` (satu mekanisme per file) dan disiplin 3-lengan
`uxss-forge.md §4` (`alert(1)` tidak membuktikan apa pun).

## ⚖️ Etika — baca sebelum menjalankan

Ini **artefak riset**, bukan kit phishing. Gerbang yang ditegakkan di setiap file:

| Wajib ada | Dilarang keras — dan tidak ada di sini |
|---|---|
| header mekanisme + predikat + expected-secure | input password / kartu / OTP, form penerima kredensial |
| panel yang mencetak `location.href` **asli** | `fetch` / `XHR` / `sendBeacon` / WebSocket ke host mana pun |
| label tepercaya = string netral (`https://www.google.com`) | logo / merek / nama organisasi nyata |
| instruksi keluar dari state (fullscreen, keyboard-lock) | service worker, persistensi, auto-nav ke pihak ketiga |

Jalankan **hanya di browser & profil milik sendiri**. Satu bukti predikat → **screenshot → STOP → lapor VRP**
(Chrome VRP / Mozilla / Apple / Arc). Jangan pernah diarahkan ke pengguna lain.

## ▶️ Cara menjalankan

Kelas ini butuh **origin HTTPS sungguhan**. `file://` bukan pengganti — banyak API (fullscreen, keyboard-lock,
notification, speech) mati atau berperilaku beda di sana, dan hasilnya jadi false-negative.

**Cara termudah — GitHub Pages:** aktifkan Pages di repo ini, lalu buka `https://<user>.github.io/<repo>/`.

**Lokal:**

```bash
python -m http.server 8000
```

lalu buka `http://localhost:8000/` (sebagian API tetap butuh https — utamakan Pages).

## 🗺️ Isi

`1` no-click · `1-click` · **brutal** = menumpuk amplifier

| # | Mode | Mekanisme | Permukaan yang diserang |
|---|---|---|---|
| 1 | NO-CLICK | nav-timing HOLD, port di-**DROP** (bukan REFUSE) | omnibox |
| 2 | 1-CLICK | popup never-commit, `location.replace` loop | omnibox popup |
| 3 | 1-CLICK | **LENGAN KONTROL** untuk 1 & 2 — port hidup 443 | baseline |
| 4 | 1-CLICK | sumber aktivasi = `<select>` natif, bukan tombol | omnibox |
| 5 | NO-CLICK | pemicu `visibilitychange` (saat balik ke tab) | omnibox |
| 6 | 1-CLICK | HOLD **tanpa jaringan** — `window.stop()` | omnibox |
| 7 | 1-CLICK | HOLD via skema tak dikenal (nol aplikasi terbuka) | omnibox |
| 8 | NO-CLICK | path menyamar jadi host + elision layar sempit | strip URL mobile |
| 9 | NO-CLICK | RTL override `U+202E` + confusable Cyrillic | host-label + judul tab |
| 10 | 1-CLICK | fullscreen + chrome digambar sendiri | toast fullscreen |
| 11 | **BRUTAL** | fullscreen + `keyboard.lock(['Escape'])` — butakan pembantahnya | toast + jalan keluar |
| 12 | NO-CLICK | topologi chrome: standalone / PWA / tab-vertikal | strip origin non-omnibox |
| 13 | 1-CLICK | popup chromeless (`toolbar=no,location=no`) | chrome popup |
| 14 | 1-CLICK | baris origin dialog JS dari `srcdoc` / `blob:` | js-dialog |
| 15 | 1-CLICK | notifikasi OS — atribusi **di luar** browser | panel notifikasi OS |
| 16 | **BRUTAL** | FUSI: TTS hidup lebih lama dari dokumennya + HOLD | omnibox **+ audio** |
| 17 | NO-CLICK | UXSS oracle — pewarisan origin, 3 lengan | INHERIT/OPAQUE/CONTROL |
| 18 | 1-CLICK | UXSS oracle — carrier `javascript:`/`srcdoc`/`write` | 4 carrier |
| 19 | NO-CLICK | UXSS oracle — **inkonsistensi dua pandangan** | selisih waktu antar-cek |
| 20 | NO-CLICK | UXSS oracle — origin diturunkan ulang di back/forward | session-history / bfcache |

## 🔬 Cara membaca hasil (ini yang menentukan laporan diterima atau tidak)

**Spoofing (1–16).** Predikatnya satu kalimat: *permukaan X menampilkan origin A, padahal dokumennya milik origin B.*
Bukti = **satu screenshot** yang memuat keduanya sekaligus — permukaan yang bohong **dan** panel origin asli.

> **Jalankan 3 setiap kali menjalankan 1 atau 2.** Kalau kontrol ikut "berhasil spoof", yang kamu lihat bukan
> mekanisme HOLD — pembacaanmu salah, jangan dilapor. Tanpa lengan kontrol, angka 1 dan 2 tidak punya arti.

**UXSS (17–20).** Verdictnya dicetak sendiri oleh file: `OK` = build sehat, `CROSSED` = penyeberangan origin.
`CROSSED` = **langsung STOP**, screenshot, catat engine+versi+OS, lapor. Jangan lanjut ke pencurian sesi nyata —
dampaknya sudah terbukti oleh pembacaan lintas-origin itu sendiri.

**Kalau semuanya `OK` / `INERT`: itu bukan kegagalan.** Itu **ilmu-negatif** — catat browser+versi+OS+tanggal +
`Retest-after`. Sel yang mati hari ini menandai batas hardening, dan batas itu justru peta ke sumbu yang belum dijaga.

## 📋 Template catat hasil

```
file | genome                          | browser+versi+OS | verdict                        | retest-after
-----+---------------------------------+------------------+--------------------------------+-------------
1    | A=dead-port C=onload D=omnibox  | Chrome 1xx / Win | SPOOFED|INERT|BLOCKED|INCONCL. | 2026-11-xx
```

`BLOCKED` **tidak dihapus** — catat **mekanisme pemblokirnya**. Itu peta hardening, dan pintu berikutnya.

## 🧪 Kalau ada yang jalan — langkah berikutnya

Jangan berhenti di satu file. Ganti **satu knob** untuk mengubah satu bug jadi satu **famili**:

- cara-hang (`A`): dead-port → `window.stop()` → skema asing → dialog → kerja sinkron panjang
- sumber aktivasi (`C`): klik → `<select>` → drag → tombol media → `visibilitychange`
- permukaan (`D`): omnibox → dialog → notifikasi → panel OS → audio
- topologi (`G`): tab biasa → PWA standalone → tab vertikal (Arc/Vivaldi/Zen) → popup chromeless
- engine: Chromium → **fork** (Arc / Brave / Edge / Vivaldi / Samsung / Opera) — patch-lag fork = jalur termurah

Fork yang tertinggal patch adalah tempat paling produktif: fix dipasang di satu jalur upstream dan sering
tidak di kembarnya.
