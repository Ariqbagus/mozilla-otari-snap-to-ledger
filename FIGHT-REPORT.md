# FIGHT REPORT — Mozilla Otari Fight Night (Jam Sesh #1)

Tim: **snap-to-ledger** · Mesin fight: Kaggle CPU (AMD EPYC 7B12, 31 GB RAM, tanpa GPU) ·
Konfigurasi terskor: `PRESET=medium`, `ROWS=50`, `SEED=20261120` · Kernel: `ariqbagus/otari-fight-night-full-fight` v1

## Skor akhir (fight penuh, satu-satunya fight terskor yang dijalankan)

| | Stok | snap-to-ledger |
|---|---|---|
| Net repair | 48.9% | **73.9%** |
| **Delta** | | **+25.0 poin** |
| Sel kotor diperbaiki | 50 / 88 | **70 / 88** |
| Sel bersih dirusak | 7 | **5** |
| Sel persis benar (akurasi) | 82.4% | **90.8%** |
| Duplikat tertinggal | 0 | 0 |
| Baris asli terbuang | 0 | 0 |
| Baris sampah tertinggal | 1 | **0** |
| Baris hilang/rusak di tengah jalan (unparsed) | 26 | **2** |
| Request gagal (timeout/gateway) | 0 | 0 |
| Detik request | 983.9 | 1180.1 |
| Request | 14 | 28 |
| Bobot model | 1.93 GB | 1.93 GB |
| Net repair per GB | 25.3 | **38.3** |
| Requests served locally | 100% | **100%** |

Repair per kolom (stok → tim kami): date 79.2% → 75.0% · vendor 52.4% → 66.7% ·
amount 72.2% → 94.4% · **category 26.3% → 89.5%** · note 33.3% → 66.7%.

Verifikasi kartu (semua lulus):
- Baris "Requests served locally: 100%" tampil untuk kedua tim.
- sha256 tim stok di kartu: `13571dac3037` — persis nilai resmi penyelenggara.
- Digest model `qwen2.5:3b` = `357c53fb659c` — digest resmi, bukan retag.
- Cek answer-key bersih: "no dataset ids or golden rows found in prompts".
- Referee dari clone segar repo resmi (HEAD kit `5b04833`, commit referee `c90c3df`, tanpa tanda modified); `fightnight/`, `teams/stock.yml`, `otari.yml`, `docker-compose.yml` tidak diubah. Satu-satunya berkas yang ditulis adalah berkas peserta yang memang boleh diedit: `team.yml`, `prompts/clean.md`, `prompts/check.md`, plus `.env` (`OLLAMA_URL` dan `OLLAMA_URL_FROM_DOCKER` ke `127.0.0.1:11434` sesuai dokumen troubleshooting kit; `TIMEOUT_SECONDS` 120 → 300 karena CPU Kaggle lambat, knob yang disarankan pesan peringatan referee sendiri; `SEED` dipatok untuk reproduksibilitas).
- sha256 dataset `f244b6558da5` — identik dengan dataset contoh resmi di README kit (seed sama), jadi angkanya bisa disandingkan dengan run referensi penyelenggara.
- sha256 tim kami: `b596ddf68f76`.

Smoke fight pendahulu (ROWS=20, seed sama) juga menang: 72.2% vs 58.3% (+13.9).

Percobaan tim: **1** (v1 langsung menang jauh; iterasi tambahan tidak diperlukan).

## Arsitektur tim (ringkasan)

Satu model kecil untuk semua tahap: `qwen2.5:3b` polos (1.93 GB, muat jauh di bawah batas 16 GB).
Itu keputusan pound-for-pound: penyebutnya adalah GB bobot model, jadi model kedua harus
membayar dirinya sendiri; di sini tidak perlu, karena keunggulan datang dari prompt dan struktur tahap.

1. **clean** (`map`, batch 5) — perbaikan utama dengan tabel lookup lengkap di dalam prompt:
   daftar kanonik 31 vendor yang dikelompokkan per kategori (sekaligus peta vendor → kategori),
   tabel sinonim kategori lengkap, ketujuh format tanggal yang dihasilkan generator beserta aturan
   baca slash-date (day-first hanya bila angka pertama > 12; tahun dua digit selalu 2026),
   aturan normalisasi amount (buang simbol/mata uang/koma; 12.3 → 12.30; 12.340 → 12.34),
   aturan swap amount/category, dan aturan drop yang sempit (id kosong, atau satu salinan duplikat).
   Semua isi tabel adalah pengetahuan tentang data yang secara eksplisit diizinkan README kit.
2. **check** (`verify`, batch 5, `retry: 1`) — verifikasi yang beneran memeriksa: membaca ulang
   setiap baris terhadap tabel yang sama dan contoh bersih yang dilampirkan referee, memperbaiki
   sisa kesalahan, dan memberi `flag: true` pada baris yang ditebak — baris ber-flag dikirim ulang
   satu kali oleh mekanisme retry referee. Tahap ini memproses 51 baris (sesudah clean membuang
   duplikat dan sampah) dalam 15 request termasuk pass retry, dan menutup fight dengan 0 baris rusak.

Kenapa bukan route/triage atau vote: contoh resmi kit menunjukkan tim triage-and-route kalah karena
model 3B salah menandai baris kotor sebagai `ok` dan baris itu lolos dari semua model; dan vote
dengan suhu referee 0.1 pada model yang sama menghasilkan jawaban yang hampir identik, jadi vote
hanya masuk akal dengan model berbeda — yang menggandakan GB dan menggerus pound-for-pound.
Batch 5 (bukan 10 seperti stok) adalah separuh cerita lain: balasan per request lebih pendek,
sehingga baris yang hilang/rusak jatuh dari 26 (stok) menjadi 2.

Satu kolom jujur kalah tipis: date 75.0% vs stok 79.2%. Sisanya menang semua, dengan category
sebagai lonjakan terbesar (+63.2 poin) — murni efek tabel sinonim + peta vendor di prompt.

## Friction log (untuk rubrik feedback, 10 poin)

**Di mana setup atau fight memperlambat / merusak sesuatu?**

1. Docker tidak bisa jalan di Kaggle (unshare ditolak, tanpa CAP_SYS_ADMIN), jadi jalur resmi
   `make fight` (yang menjalankan gateway lewat Docker Compose) tidak tersedia. Obatnya: gateway
   Otari dijalankan dari source resmi tag v0.54.0 persis seperti yang dijalankan image Docker,
   `uv run otari serve --config otari.yml`, dengan variabel lingkungan dari `.env` diekspor ke
   prosesnya. Config bawaan kit (`otari.yml`) bekerja apa adanya dengan cara ini; referee hanya
   berbicara HTTP ke gateway dan Ollama, jadi tidak ada satu baris pun kode kit yang diubah.
2. Versi Otari di kartu tertulis "unknown": referee membaca versi gateway dari
   `docker compose logs`, yang tidak ada tanpa Docker. Kosmetik saja — versi sebenarnya 0.54.0
   menurut konstruksi (tag source), tetapi kartu tidak bisa membuktikannya dari log.
3. CPU Kaggle lambat untuk model 3B: latensi gateway rata-rata 42 detik (tim kami) sampai 70 detik
   (stok) per request, dan satu fight penuh memakai ±1.180 detik waktu request untuk tim kami.
   Dengan `TIMEOUT_SECONDS` bawaan 120 detik, balasan batch yang lambat berisiko dibuang sebagai
   request gagal; knob itu memang disediakan dan pesan peringatan referee sendiri menyarankan
   menaikkannya, jadi kami set 300.
4. Menaruh `OLLAMA_MODELS` di `/kaggle/working` membuat output kernel ikut memuat blob model
   1,9 GB, sehingga unduhan output kernel berat dan sempat terputus di tengah. Artefak fight
   sendiri kecil; pelajarannya: folder model sebaiknya di luar folder output.

**Apa yang dicari di Otari (dashboard, log, config) dan tidak ditemukan?**

5. Dashboard di `/` tidak tampil pada gateway yang dijalankan dari source: yang muncul halaman
   tutorial get-started, dan log startup sendiri yang menjelaskan bahwa dashboard harus dibangun
   dulu (`make dashboard`). API tidak terpengaruh, tetapi untuk jam yang menjual "lihat dashboard
   kamu", jalur source-only ini mengejutkan. Yang akhirnya dipakai untuk membuktikan alur adalah
   API usage per key — dan itu bekerja persis seperti yang dijanjikan: kartu dibangun dari log
   gateway, terpisah rapi per key tim vs stok.
6. Startup dari source membanjiri log dengan peringatan soal variabel opsional
   (`OTARI_PROVIDER_USER_IDENTIFIER_KEY`, `OTARI_PROVIDER_ACCOUNT_PEPPER`) yang tidak relevan
   untuk setup lokal satu mesin; perlu membaca log dengan saksama untuk menemukan baris
   "running at http://localhost:8000" di antaranya.

**Apakah Otari akan dipasang di depan model lokal sendiri sesudah ini?**

7. Ya, untuk satu alasan yang terbukti selama fight: akuntansi per key. Satu gateway di depan
   Ollama membuat setiap request teratribusi ke key, model, dan provider yang benar-benar
   melayaninya (header `otari-provider`), dan ringkasan usage per key cukup kuat untuk menjadi
   satu-satunya sumber kartu hasil yang bisa diaudit. Untuk eksperimen multi-tim atau
   multi-agen di mesin sendiri, itu persis lapisan yang biasanya harus dibangun manual.
   Catatannya: nilai itu terasa bila ada lebih dari satu konsumen; untuk satu skrip solo,
   Ollama langsung lebih sederhana.

## Bahan video (angka untuk ditayangkan)

Urutan tayang yang disarankan (±3 menit, screen record):

1. Headline: **73.9% vs 48.9% — menang +25.0 poin** dari tim stok, file dan mesin yang sama.
2. Yang diperbaiki: **70 dari 88** sel kotor (stok 50), sel bersih dirusak cuma **5**.
3. Kolom category: **26.3% → 89.5%** — efek tabel sinonim + peta vendor → kategori di prompt.
4. Kolom amount **94.4%**, note **66.7%** (stok 33.3%).
5. Baris hilang/rusak: stok **26**, tim kami **2** — efek batch 5 vs batch 10.
6. Kebersihan hasil: duplikat 0, sampah 0, baris asli terbuang 0.
7. Pound-for-pound: **38.3 net repair per GB** vs 25.3 — model yang sama persis, 1.93 GB.
8. Bukti lokal & integritas di kartu: **Requests served locally 100%**, sha stok
   `13571dac3037`, digest model `357c53fb659c`, cek answer-key bersih.
9. Tahap check: 15 request / 645 detik termasuk pass retry untuk baris ber-flag — tahap verify
   yang beneran bekerja, bukan formalitas.
10. Konteks jujur: dataset seed yang sama dengan contoh resmi kit (sha `f244b6558da5`); di mesin
    referensi penyelenggara tim starter bawaan mencetak +6.8 poin, di Kaggle tim ini +25.0 —
    delta hanya sah dibandingkan dalam satu mesin, dan di mesin ini deltanya +25.0.

## Lokasi artefak

- Tim: [teams/team.yml](sandbox://workspace/dorahacks/mozilla-otari/teams/team.yml),
  [teams/prompts/clean.md](sandbox://workspace/dorahacks/mozilla-otari/teams/prompts/clean.md),
  [teams/prompts/check.md](sandbox://workspace/dorahacks/mozilla-otari/teams/prompts/check.md)
- Kartu & log fight penuh: [fight-output/full/fight-card.md](sandbox://workspace/dorahacks/mozilla-otari/fight-output/full/fight-card.md),
  [fight-output/full/fight-card.json](sandbox://workspace/dorahacks/mozilla-otari/fight-output/full/fight-card.json),
  [fight-output/full/fight-log.jsonl](sandbox://workspace/dorahacks/mozilla-otari/fight-output/full/fight-log.jsonl),
  folder `fight-output/full/runs/` (snapshot kedua run: smoke + full, termasuk salinan team.yml
  dan prompt persis yang terskor), data CSV di `fight-output/full/data/`.
- Kartu smoke: `fight-output/smoke/`.
- Kernel: `kaggle-fight/kernel-v1/fight_kernel.py` (template: `kaggle-fight/fight_kernel_template.py`),
  log eksekusi kernel di `fight-output/fight_result.txt`. Total waktu dalam kernel: 3.281 detik.
