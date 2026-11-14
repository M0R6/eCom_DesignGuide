# 📐 DESIGN GUIDE — eCom (Petani × Konsumen)

> Dokumen desain UI untuk seluruh halaman project eCom.
> Acuan: `prisma/schema.prisma`, `fitur.txt`, dan implementasi Bagian 1–4.
> Pembagian tugas: **Morgan** (Bagian 6–10), **Fadhly** (Bagian 11–12), **Ahmad** (Bagian 13 + polish).

Status halaman: ✅ = sudah dibangun · 🔜 = akan dibangun

---

## 🎨 Aturan Global (berlaku di semua halaman)

```
┌──────────────────────────────────────────────────────┐
│  UHeader (sticky, h=--ui-header-height)              │
│  [Logo eCom]   [Nav sesuai role]      [🌙][Login/👤] │
├──────────────────────────────────────────────────────┤
│                                                      │
│   KONTEN (UContainer, max-w tergantung halaman)      │
│                                                      │
└──────────────────────────────────────────────────────┘
                    ↑ Toast (fixed, pojok kanan atas)
```

**Aturan baku (wajib dipertahankan, sudah terbukti sejak Bagian 1–4):**

- Wrapper konten: `min-h-[calc(100dvh-var(--ui-header-height))]` — **jangan** pakai `min-h-screen` mentah (menyebabkan scroll kecil karena tinggi header tidak dikurangi).
- Feedback sukses/gagal selalu `useToast().add({ id: '<unik>', ... })` — `id` unik mencegah toast menumpuk/dobel.
- Error response API: `createError({ statusCode, statusMessage })` dengan pesan Bahasa Indonesia.
- Client kirim angka sebagai `Number` (bukan string) untuk harga/stok; tampil dengan `Number(x).toLocaleString('id-ID')`.
- Mapping badge status konsisten:

| Status | Warna badge |
|---|---|
| `Tersedia` / `Aktif` / `Selesai` | `success` |
| `Menunggu Pembayaran` / `Habis` | `warning` |
| `Dibatalkan` | `error` |
| `Disembunyikan` / `Jeda` / `Berhenti` | `neutral` |

---

## 1️⃣ Halaman Login — `/login` ✅

```
┌──────────────────────────────────────────────────┐
│  [Logo]  Docs Components Releases        [🌙][X] │  ← UHeader
├──────────────────────────────────────────────────┤
│                                                  │
│              ┌──────────────────────┐            │
│              │       Login          │  ← header  │
│              ├──────────────────────┤            │
│              │ Email                │            │
│              │ [__________________] │            │
│              │ Password             │            │
│              │ [••••••••••••••••••] │            │
│              │                      │            │
│              │ [      MASUK       ] │  ← block   │
│              ├──────────────────────┤            │
│              │ Belum punya akun?    │  ← footer  │
│              │    Daftar            │            │
│              └──────────────────────┘            │
│           max-w-sm, center vertikal              │
└──────────────────────────────────────────────────┘
```

**Penjelasan:** form 2 field saja (email + password), tanpa distraksi. CTA tunggal. Toast warning otomatis muncul di pojok atas jika user diarahkan ke sini dari middleware (`?unauthorized=true`).

**Komponen Nuxt UI:** `UHeader` · `UContainer` · `UCard` (#header, #footer slots) · `UForm` + `UFormField` + `UInput type="password"` · `UButton block loading` · `UTooltip` di header.

---

## 2️⃣ Halaman Register — `/register` ✅

```
┌──────────────────────────────────────────────────┐
│  [Logo]                                  [🌙]    │
├──────────────────────────────────────────────────┤
│              ┌──────────────────────┐            │
│              │     Buat Akun        │            │
│              ├──────────────────────┤            │
│              │ Nama                 │            │
│              │ [__________________] │            │
│              │ Email                │            │
│              │ [__________________] │            │
│              │ Daftar sebagai       │            │
│              │ (•) Konsumen         │  ← radio   │
│              │     "Saya ingin      │            │
│              │      membeli produk" │            │
│              │ ( ) Petani           │            │
│              │     "Saya ingin      │            │
│              │      menjual hasil"  │            │
│              │ Password             │            │
│              │ [••••••••••••••••••] │            │
│              │ Konfirmasi Password  │            │
│              │ [••••••••••••••••••] │            │
│              │ [     DAFTAR       ] │            │
│              ├──────────────────────┤            │
│              │ Sudah punya akun?    │            │
│              │    Login             │            │
│              └──────────────────────┘            │
└──────────────────────────────────────────────────┘
```

**Penjelasan:** satu-satunya halaman yang menentukan `peran` (enum `Petani`/`Konsumen`). Deskripsi tiap radio penting supaya user paham bedanya. Pilihan role **tidak bisa diganti setelah daftar**, jadi posisinya mencolok (di atas password).

**Komponen Nuxt UI:** `URadioGroup` + `:items` dengan `label/value/description` · sisanya sama dengan login.

---

## 3️⃣ Halaman Profil — `/profil` ✅

```
┌──────────────────────────────────────────────────┐
│  [Logo]                          [🌙][Dropdown👤]│
├──────────────────────────────────────────────────┤
│  Profil Akun                                     │
│                                                  │
│  ┌─────────────┬─────────────┬─────────────┐     │  ← UTabs
│  │ Data Diri   │  Alamat     │ Rekening*   │     │
│  └─────────────┴─────────────┴─────────────┘     │
│                                                  │
│  ┌─ TAB DATA DIRI (khusus Petani) ────────────┐  │
│  │  Nama          [________________]          │  │
│  │  Nama Pertanian[________________]          │  │
│  │  No. Telepon   [________________]          │  │
│  │  Alamat Pertanian                          │  │
│  │  [____________________________________]    │  │
│  │  Deskripsi                                 │  │
│  │  [____________________________________]    │  │
│  │  [  Simpan Perubahan  ]                    │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌─ TAB ALAMAT ───────────────────────────────┐  │
│  │        [ + Tambah Alamat ]   ← header kanan│  │
│  │  ┌──────────────────────────────────────┐  │  │
│  │  │ [RUMAH] 🏷            [✏️] [🗑]        │  │  │
│  │  │ Siti Aminah · 089876543210           │  │  │
│  │  │ Jl. Melati No.12, Desa..., 35362     │  │  │
│  │  └──────────────────────────────────────┘  │  │
│  │  ┌──────────────────────────────────────┐  │  │
│  │  │ [KANTOR] 🏷           [✏️] [🗑]        │  │  │
│  │  └──────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌─ TAB REKENING (Petani saja) ───────────────┐  │
│  │        [ + Tambah Rekening ]               │  │
│  │  ┌──────────────────────────────────────┐  │  │
│  │  │ BCA                  [✏️] [🗑]        │  │  │
│  │  │ 1234567890 — a.n. Budi Santoso       │  │  │
│  │  └──────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌─ MODAL (popup saat Tambah/Edit) ───────────┐  │
│  │  Alamat Pengiriman                    [X]  │  │
│  │  Label   [Rumah____]                       │  │
│  │  Penerima[________]  Telepon [_______]     │  │
│  │  Detail Alamat [______________________]    │  │
│  │  ┌─────────┬──────────┬───────────────┐    │  │
│  │  │Desa     │Kecamatan │               │    │  │
│  │  │Kota     │Provinsi  │  (grid 2 col) │    │  │
│  │  │Kode Pos │          │               │    │  │
│  │  └─────────┴──────────┴───────────────┘    │  │
│  │  [        Simpan Alamat        ]           │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

**Penjelasan:** struktur tab adaptif — Konsumen hanya 2 tab (Data Diri + Alamat), Petani dapat tab Rekening Bank. Modal dipakai alih-alih halaman terpisah karena formnya sederhana dan CRUD-nya cepat beruntun (tambah → edit → hapus). Kartu alamat pakai badge label ("Rumah" vs "Kantor") supaya mudah dibedakan.

**Komponen Nuxt UI:** `UTabs` + named slots (`#data-diri`, `#alamat`, `#rekening`) · `UCard variant="subtle"` untuk kartu list · `UBadge` label alamat · `UModal v-model:open` + `#body` slot · `UButton variant="ghost"` icon pencil/trash · `UEmpty` untuk kondisi kosong.

---

## 4️⃣ Dashboard Petani — CRUD Hasil Panen — `/dashboard-petani/produk` ✅

```
┌──────────────────────────────────────────────────────┐
│  [Logo]                              [🌙][Dropdown👤]│
├──────────────────────────────────────────────────────┤
│  Hasil Panen Saya              [ + Posting Panen ]   │
│                                                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │  │
│  │ │  FOTO    │ │ │ │  FOTO    │ │ │ │  FOTO    │ │  │
│  │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │  │
│  │ Bayam Hijau  │ │ Cabai Rawit  │ │ Tomat Sayur  │  │
│  │ Sayur Daun   │ │ Bumbu Dapur  │ │ Bumbu Dapur  │  │
│  │  [Tersedia]  │ │  [Tersedia]  │ │   [Habis]    │  │
│  │              │ │              │ │              │  │
│  │ Rp8.000/ikat │ │ Rp45.000/kg  │ │ Rp12.000/kg  │  │
│  │ Stok: 50     │ │ Stok: 15     │ │ Stok: 0      │  │
│  ├──────────────┤ ├──────────────┤ ├──────────────┤  │
│  │ [Edit][Hapus]│ │ [Edit][Hapus]│ │ [Edit][Hapus]│  │
│  └──────────────┘ └──────────────┘ └──────────────┘  │
│   grid: 1 col (HP) / 2 col (tablet) / 3 (desktop)    │
│                                                      │
│  ⚠️ Produk yang pernah dibeli + [Hapus] →            │
│     toast: "Produk disembunyikan" (soft-delete)      │
└──────────────────────────────────────────────────────┘
```

**Penjelasan:** format kartu grid (bukan tabel) karena tiap produk punya foto — lebih cocok untuk konten visual panen. Badge status di pojok kanan atas kartu = info yang paling sering dilihat petani. Hapus yang produknya pernah dibeli otomatis jadi "Disembunyikan" (konstrain FK `barangpesanan`), bukan terhapus — riwayat pesanan konsumen tetap utuh.

**Komponen Nuxt UI:** `UCard` (#header = foto, #footer = aksi) · `UBadge :color` · `UEmpty` dengan CTA · grid Tailwind `sm:grid-cols-2 lg:grid-cols-3` · `UButton :loading` per-kartu (state `loadingHapus`).

---

## 5️⃣ Form Posting/Edit Panen — `/dashboard-petani/produk/baru` & `.../[id]/edit` ✅

```
┌──────────────────────────────────────────────────┐
│  [Logo]                                          │
├──────────────────────────────────────────────────┤
│  Posting Hasil Panen          (Edit: judul beda) │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │  Nama Produk   [Bayam Hijau_____________]  │  │
│  │                                            │  │
│  │  Kategori      [Sayur Daun        ▼]       │  │
│  │                                            │  │
│  │  ┌───────────┬───────────┬──────────┐      │  │
│  │  │Harga (Rp) │ Stok      │ Satuan   │      │  │
│  │  │[8000    ▲▼]│[50     ▲▼]│[kg    ▼]│      │  │
│  │  └───────────┴───────────┴──────────┘      │  │
│  │                                            │  │
│  │  Status Tampil  [Tersedia (tampil)  ▼]     │  │ ← hanya di EDIT
│  │                                            │  │
│  │  Deskripsi                                 │  │
│  │  [____________________________________]    │  │
│  │  [____________________________________]    │  │
│  │                                            │  │
│  │  [        POSTING / SIMPAN       ]         │  │
│  └────────────────────────────────────────────┘  │
│              max-w-2xl, single column            │
└──────────────────────────────────────────────────┘
```

**Penjelasan:** satu layout untuk Create dan Edit (beda judul + field `Status Tampil` ekstra saat edit). Harga/Stok/Satuan digroup 1 baris karena satu paket informasi. Validasi client-side (zod) + server-side (zod juga). Foto menyusul di Bagian 4 sebagai dropzone di bawah deskripsi.

**Komponen Nuxt UI:** `UForm :schema :state` (zod) · `UInputNumber` (fallback: `UInput type="number" v-model.number`) · `USelect` · `UTextarea :rows` · `UButton block :loading`.

---

## 6️⃣ Katalog Konsumen — `/` 🔜 (Morgan)

```
┌────────────────────────────────────────────────────────────┐
│  [Logo]  Katalog   [🔍 Cari sayur…]     [🛒 3]  [👤]       │
├────────────┬───────────────────────────────────────────────┤
│  FILTER    │  Hasil panen segar langsung dari petani desa  │
│            │                                               │
│ Kategori   │  ┌────────────┐ ┌────────────┐ ┌────────────┐ │
│ ☑ Sayur    │  │    FOTO    │ │    FOTO    │ │    FOTO    │ │
│   Daun     │  │ Bayam      │ │ Kangkung   │ │ Semangka   │ │
│ ☑ Buah     │  │ Rp8rb/ikat │ │ Rp6rb/ikat │ │ Rp25rb/bh  │ │
│ ☑ Bumbu    │  │ Tani Makmur│ │ Tani Makmur│ │ Tani Makmur│ │
│            │  │ [Tersedia] │ │ [Tersedia] │ │ [Tersedia] │ │
│ Harga      │  │ [+ Keranjang]│[+ Keranjang]│[Lihat →] │ │
│ Min [___]  │  └────────────┘ └────────────┘ └────────────┘ │
│ Maks [___] │  ┌────────────┐ ┌────────────┐                │
│            │  │    FOTO    │ │    FOTO    │                │
│ Desa/Petani│  │ Cabai      │ │ Ubi Ungu   │                │
│ [Semua  ▼] │  │ ...        │ │ ...        │                │
│            │  └────────────┘ └────────────┘                │
│ [Reset]    │        (HP: sidebar jadi tombol Filter ▼)     │
└────────────┴───────────────────────────────────────────────┘
```

**Penjelasan:** dua kolom di desktop (sidebar filter + grid), di HP sidebar berubah jadi drawer. Search bar di **header** selalu terlihat saat scroll. Kartu produk polanya sama dengan kartu dashboard petani, tapi aksinya beda: petani → Edit/Hapus, konsumen → Tambah ke Keranjang. Produk `Habis` tetap tampil tapi tombolnya disabled + badge warning. Icon keranjang di header pakai `UChip` jumlah item — update saat tambah.

**Komponen Nuxt UI:** `UInput` + icon search · `UCheckboxGroup`/`UCheckbox` kategori · `USelect` desa · `UInputNumber` rentang harga · `UDrawer` filter mobile · `UChip` di `UButton` keranjang · `UPagination` di bawah grid.

---

## 7️⃣ Detail Produk (Traceability View) — `/produk/[id]` 🔜 (Morgan)

```
┌──────────────────────────────────────────────────────────┐
│  [Logo]                                    [🛒 3]  [👤]  │
├──────────────────────────────────────────────────────────┤
│  ← Kembali        [Breadcrumb: Katalog / Sayur Daun]     │
│                                                          │
│  ┌────────────────┐   Bayam Hijau              [Tersedia]│
│  │                │   Rp8.000 / ikat                     │
│  │  FOTO UTAMA    │   Stok: 50 ikat                      │
│  │                │   ┌──────────────────────────────┐   │
│  └────────────────┘   │ Jumlah: [ 1 ] [ + Keranjang ] │   │
│  [t1][t2][t3]         └──────────────────────────────┘   │
│  ← thumbnail         Stok muncul REAL-TIME               │
│                                                          │
│  ┌─ Deskripsi ──────────────────────────────────────┐   │
│  │ Bayam hijau segar dipetik pagi hari…             │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─ Jejak Panen (Traceability) ─────────────────────┐   │
│  │ 👨‍🌾  Tani Makmur Desa Sukamaju                    │   │
│  │ 📍  Desa Sukamaju, Kec. Natar, Lampung Selatan   │   │
│  │ 🌱  Petani sayur organik keluarga, 3 generasi…   │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─ Produk lain dari petani ini ────────────────────┐   │
│  │ [mini kartu] [mini kartu] [mini kartu]           │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

**Penjelasan:** ini fitur khas tema "Traceability" — konsumen tahu siapa petaninya dan dari desa mana. Struktur: galeri kiri (foto utama + thumbnail), info + aksi kanan. Seksi "Jejak Panen" wajib ada (nama pertanian, lokasi, deskripsi dari `profilpetani`). Rekomendasi produk lain dari petani yang sama = nilai jual tambahan.

**Komponen Nuxt UI:** grid 2 kolom · thumbnail `UButton variant="ghost"` berisi img + ring aktif · `UCard` per seksi · `UInputNumber` jumlah · `UBadge` status.

---

## 8️⃣ Keranjang — `/keranjang` 🔜 (Morgan)

```
┌──────────────────────────────────────────────────────────┐
│  [Logo]                                    [🛒 3]  [👤]  │
├──────────────────────────────────────────────────────────┤
│  Keranjang Belanja                                       │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │ ┌────┐ Bayam Hijau              [🗑]             │   │
│  │ │FOTO│ Rp8.000/ikat                             │   │
│  │ │    │ Jumlah: [−] 2 [+]        Subtotal 16.000  │   │
│  │ └────┘                                          │   │
│  ├──────────────────────────────────────────────────┤   │
│  │ ┌────┐ Cabai Rawit Merah        [🗑]             │   │
│  │ │FOTO│ Rp45.000/kg                              │   │
│  │ │    │ Jumlah: [−] 1 [+]        Subtotal 45.000  │   │
│  │ └────┘                                          │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌────────────────────────────┐                          │
│  │ Total:            Rp61.000 │   ← ringkasan sticky     │
│  │ [     LANJUT CHECKOUT    ] │                          │
│  └────────────────────────────┘                          │
│                                                          │
│  ⚠️ Jika stok berubah: baris item merah                  │
│     "Stok tersisa 2 ikat" + jumlah otomatis disesuaikan  │
└──────────────────────────────────────────────────────────┘
```

**Penjelasan:** list baris item (bukan grid) karena fokus manipulasi jumlah. Tombol `−/+` langsung update `jumlah` via API (debounce). Ringkasan total di bawah (di HP: sticky bottom bar). **Validasi stok di endpoint wajib** — jika stok berubah setelah item masuk keranjang, server menolak dan UI menandai barisnya.

**Komponen Nuxt UI:** `UCard variant="subtle"` per item atau `UButtonGroup` untuk `−/+` · `UInputNumber` · icon trash · sticky footer `UCard` + `UButton block`.

---

## 9️⃣ Checkout — `/checkout` 🔜 (Morgan, middleware `konsumen`)

```
┌──────────────────────────────────────────────────────────┐
│  [Logo]                                                  │
├──────────────────────────────────────────────────────────┤
│  Checkout                                                │
│                                                          │
│  STEP 1 — Alamat Pengiriman                              │
│  ┌──────────────────────────────────────────────────┐   │
│  │ (•) [Rumah]  Siti Aminah, Jl. Melati No.12…      │   │
│  │ ( ) [Kantor] Siti Aminah, Dinas Pendidikan…      │   │
│  │ [ + Alamat Baru ]  ← buka modal sama dgn profil  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  STEP 2 — Ringkasan Pesanan                              │
│  ┌──────────────────────────────────────────────────┐   │
│  │ 2× Bayam Hijau ........... 16.000                │   │
│  │ 1× Cabai Rawit ........... 45.000                │   │
│  │ ─────────────────────────────────                 │   │
│  │ Total                   61.000                   │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  STEP 3 — Konfirmasi                                     │
│  [   BAYAR & PESAN SEKARANG   ]                          │
│                                                          │
│  (sukses → toast + redirect /pesanan)                    │
│  (belum login → middleware: toast + /login)              │
└──────────────────────────────────────────────────────────┘
```

**Penjelasan:** satu halaman dibagi seksi bernomor — bukan multi-step wizard (transaksinya sederhana, wizard hanya menambah friksi). Pilihan alamat memakai radio dari `alamat` milik user (Bagian 2) — sinkron dengan `langganan.idAlamat`. Tombol terakhir memanggil `checkout.post.ts` yang semuanya dalam **1 Prisma transaction** (kurangi stok → buat pesanan+barang+riwayat → kosongkan keranjang).

**Komponen Nuxt UI:** `URadioGroup` alamat · ringkasan `UCard` · `UButton :loading` besar · opsional `UStepper` jika mau visual langkah.

---

## 🔟 Riwayat & Detail Pesanan — `/pesanan` & `/pesanan/[id]` 🔜 (Morgan)

```
Riwayat:                              Detail pesanan:
┌──────────────────────────────┐   ┌──────────────────────────────────┐
│  Pesanan Saya                │   │  Pesanan #2      [Dikirim] 🟢    │
│                              │   │  Dibuat: 12 Nov 2026             │
│ ┌──────────────────────────┐ │   │  Kirim ke: Jl. Melati No.12…     │
│ │ #2 [Dikirim]   [🟢]      │ │   │                                  │
│ │ 1× Kangkung    8.000     │ │   │  Barang:                         │
│ │ 12 Nov 2026              │ │   │ ┌────┐ 1× Kangkung  8.000       │
│ │                    [Lihat]│ │   │ └────┘                          │
│ └──────────────────────────┘ │   │                                  │
│ ┌──────────────────────────┐ │   │  Lacak Pesanan:                  │
│ │ #1 [Selesai]   [✅]      │ │   │  ●──●──◉──○   ← timeline        │
│ │ 5× Bayam +1   61.000     │ │   │  Menunggu→Diproses→Dikirim→     │
│ │ 6 Nov 2026               │ │   │  Selesai                        │
│ │                    [Lihat]│ │   │  ─ Dikirim: "Barang diantar ke  │
│ └──────────────────────────┘ │   │    drop point desa" (6 Nov)     │
│  [Dibatalkan] → badge merah  │   └──────────────────────────────────┘
└──────────────────────────────┘
```

**Penjelasan:** riwayat = list ringkas dengan badge status + warna. Detail memakai **timeline vertikal** dari tabel `riwayatstatuspesanan` (statusLama → statusBaru + catatan + waktu). Ini "lacak status" yang diminta di fitur 2.

**Komponen Nuxt UI:** `UBadge` · list `UCard` · timeline dari `UStepper orientation="vertical"` (statis) atau elemen li kustom dengan dot Tailwind · `UBreadcrumb`.

---

## 1️⃣1️⃣ Dashboard Penjualan Petani — `/dashboard-petani/pesanan` 🔜 (Fadhly)

```
┌────────────────────────────────────────────────────────────┐
│  [Logo]                              [🌙][Dropdown👤]      │
├────────────────────────────────────────────────────────────┤
│  Pesanan Masuk   [Semua ▼] [Menunggu ▼] [Diproses] [Dikirim]│
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ #2 · Siti Aminah                    [Diproses] 🟡     │ │
│  │ 1× Kangkung Hidroponik (ikat)  —  8.000              │ │
│  │ Kirim ke: Jl. Melati No.12, Sukamaju…                │ │
│  │                                     [ Ubah Status ▼ ]│ │
│  │   → Diproses │ → Dikirim │ → Selesai │ → Batalkan    │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ┌─ Ringkasan ────────────────────────────────────────┐   │
│  │ Pesanan baru: 2   Perlu dikirim: 1   Omzet bulan:… │   │
│  └────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

**Penjelasan:** tampilan berbeda dari riwayat konsumen — petani butuh **aksi** (ubah status), konsumen butuh **informasi** (timeline). Dropdown ubah status memvalidasi transisi yang legal (tidak boleh lompat Dikirim → Menunggu Pembayaran) dan setiap perubahan menulis ke `riwayatstatuspesanan` (`idDiubahOlehPengguna` = petani yang login). Ringkasan di bawah opsional tapi murah dibuat.

**Komponen Nuxt UI:** `UTabs`/`UButtonGroup` filter status · `UDropdownMenu` aksi status · `UModal` konfirmasi batal (field catatan → kolom `catatan` riwayat) · `USelect` filter.

---

## 1️⃣2️⃣ Langganan — `/langganan` 🔜 (Fadhly)

```
┌──────────────────────────────────────────────────────────┐
│  [Logo]                                    [🛒 3]  [👤]  │
├──────────────────────────────────────────────────────────┤
│  Langganan Rutin                                         │
│                                                          │
│  (Di detail produk juga ada blok:)                       │
│  ┌─ Berlangganan produk ini ────────────────────────┐   │
│  │ Frekuensi: ( )Harian (•)Mingguan ( )Bulanan      │   │
│  │ Mulai: [2026-11-20 📅]   Alamat: [Rumah ▼]       │   │
│  │ [ Langgankan ]  → toast sukses + link /langganan │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─ Langganan #1 ──────────────── [Aktif] 🟢 ───────┐   │
│  │ Frekuensi: Mingguan                              │   │
│  │ Pengiriman berikutnya: 20 Nov 2026               │   │
│  │ 3× Bayam Hijau + 3× Kangkung                     │   │
│  │                                   [⏸ Jeda][🗑]   │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Status: Aktif🟢 | Jeda⚪ | Berhenti⚫                   │
└──────────────────────────────────────────────────────────┘
```

**Penjelasan:** entry point dua jalur — dari detail produk (create) dan halaman ringkasan (manage). Toggle `Jeda` hanya menghentikan scheduler; `Berhenti` permanen (konfirmasi via modal). Alamat yang dipilih harus dari daftar alamat milik user — endpoint hapus alamat sudah menolak jika dipakai langganan aktif (Bagian 2). Scheduler membaca `tanggalPengirimanBerikutnya` tiap hari.

**Komponen Nuxt UI:** `URadioGroup` frekuensi · `UInput type="date"` · `USelect` alamat · `UBadge` status · `USwitch` untuk Jeda · `UModal` konfirmasi berhenti.

---

## 1️⃣3️⃣ Chatbot — floating widget, semua halaman 🔜 (Ahmad)

```
                                          ┌──────────────────┐
  (di semua halaman, pojok kanan bawah)   │ Tanya eCom  [—][X]│
                                          ├──────────────────┤
        ┌──────────┐                      │ [Percakapan ▼]   │
        │ 💬 (chip)│  ← UButton floating  ├──────────────────┤
        └──────────┘                      │        ┌────────┐│
                klik ▼                    │        │ Cara   ││ User
                                          │        │ pesan? ││
                                          │ ┌──────┴──────┐ ││
                                          │ │ Pilih produk│ ││ Bot
                                          │ │ di katalog… │ ││
                                          │ └─────────────┘ ││
                                          ├──────────────────┤
                                          │ [Tulis pesan…][➤]│
                                          └──────────────────┘
```

**Penjelasan:** widget mengambang global (di `app.vue`), bukan halaman. Percakapan tersimpan di `percakapanchat`/`pesanchat` — user bisa melanjutkan riwayat lama. Jawaban bot dicari dari `pengetahuanchatbot` (kataKunci, `aktif = true`); jika tidak ketemu, jawab generik + tawarkan hubungi admin (enum `pesanchat_peran` sudah menyediakan `Admin`).

**Komponen Nuxt UI:** `UDrawer`/panel kustom absolute bottom-right · bubble chat = div Tailwind (kiri = Bot/neutral, kanan = User/primary) · `UTextarea` autosize · `UButton` icon send · `USelect` kecil untuk pilih percakapan.

---

## 📋 Cheat Sheet Pemetaan Page → Komponen Nuxt UI 4

| Pattern di sketsa | Komponen Nuxt UI |
|---|---|
| Halaman dengan header sticky | `UHeader` + `UContainer` |
| Form + validasi | `UForm` + `UFormField` + zod schema |
| Input angka (harga/stok/jumlah) | `UInputNumber` (fallback: `UInput type="number" v-model.number`) |
| Pilihan role/kategori/status/frekuensi | `USelect`, `URadioGroup :items` |
| CRUD cepat (alamat/rekening) | `UModal` + `UCard` list + tombol icon `variant="ghost"` |
| Status/kondisi | `UBadge :color` (mapping di Aturan Global) |
| Kondisi kosong | `UEmpty` (+ CTA) |
| CRUD multi-bagian | `UTabs` + named slots |
| Langkah/riwayat berurutan | `UStepper` / timeline kustom |
| Widget chat / filter mobile | `UDrawer` |
| Notifikasi | `UApp` toaster + `useToast` dengan `id` unik |
| Keranjang di header / badge kecil | `UChip` |

---

## 🔄 Titik Integrasi Antar-Anggota

1. **`server/utils/createOrder.ts`** — satu fungsi pembuat pesanan (transaction: validasi stok → pesanan + barang + riwayat → kurangi stok). Morgan pakai untuk checkout manual, Fadhly untuk langganan otomatis.
2. **`schema.prisma`, `Header.vue`, `auth.d.ts`** — jangan diedit bersamaan; sepakati dulu di grup jika perlu perubahan.
3. **Kontrak endpoint** — respons `{ ok: true }` + toast, error via `createError` dengan `statusMessage` Bahasa Indonesia.
4. **Format harga** — kirim `Number`, tampil `toLocaleString('id-ID')`.

## 📅 Urutan Pengerjaan

```
Minggu 1: M1-M2 (katalog)     | F1-F2 (dashboard penjualan)  | A1-A2 (chatbot)
Minggu 2: M3-M4 (detail+cart) | F3-F4 (status+langganan)     | A3 (polish UI)
Minggu 3: M5 (checkout)       | F5 (scheduler) + integrasi   | A4 (QA end-to-end)
```
