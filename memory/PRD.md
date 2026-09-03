# PRD — Kain Nusantara (ERP Tekstil)

## Problem Statement Asli
Lanjutkan development dari repo https://github.com/wakasajanamasa/KN (ERP tekstil multi-entitas:
React + FastAPI + MongoDB). Development sebelumnya berhenti setelah iteration_273 (semua tes PASS,
tersisa action items minor).

## Arsitektur
- Backend: FastAPI modular (`backend/routers/*`, `backend/services/*`), MongoDB via Motor.
- Frontend: React CRA, **dilayani dari bundle statis `frontend/build` oleh `static_server.js`
  (TIDAK ada hot reload — setelah edit `frontend/src` WAJIB `bash scripts/rebuild_frontend.sh`)**.
- Multi-entitas: PT Kain Suka Cita (ent_ksc / KSC) + CV Kanda Suka (KANDA); isolasi via
  `entity_scope.py` (entity_ctx / resolve_scope_ids / assert_entity_access), header `X-Entity-Id`.
- Restore lingkungan setelah clone: `bash /app/.restore_env.sh` (pip + yarn + mongo + seed_realistic.py + build).
- Kredensial demo: lihat `/app/memory/test_credentials.md` (admin@kainnusantara.id / demo12345).

## Persona
- Admin (Budi) — akses penuh; Manager (Dewi) — persetujuan; Admin Sales, Finance, Sales, Gudang, Desainer.

## Yang Sudah Diimplementasikan
- s/d iteration_273: Aturan Persetujuan dikolapskan ke skema mesin tunggal
  {doc_type, entity_id, min_amount, max_amount, required_role, sort, active, is_percent};
  UI + mesin (config_service.evaluate_approval) membaca koleksi yang sama. 3 fix kosmetik
  (blanket modal unit 130px, label mini PO create, thead COA).
- 2026-06 (sesi ini — action items iteration_273):
  1. CSS `.form-row-3col` ditambahkan (`styles/components.css`) — modal aturan kini 3 kolom.
  2. Cakupan entitas pada Aturan Persetujuan: dropdown `rule-entity-id` di form
     (Semua entitas / KSC / KANDA), payload POST & PATCH kirim `entity_id`, kolom Cakupan
     menampilkan nama entitas. Backend memvalidasi `entity_id` ∈ allowed_entity_ids
     (403 bila tidak berwenang) pada POST & PATCH (`_assert_rule_entity_allowed`).
  3. GET /approval-rules/{id}: cek 404 dipindah SEBELUM assert_entity_access.
  4. Lebar select Satuan & Grade pada baris Tambah Item PO create: 104px → 130px (kedua grid).
  5. Escape menutup FormModal — sudah tertangani `useEscapeClose` (INV-UI-10), diverifikasi ulang.

- 2026-06 (verifikasi FASE I): FASE I (Inspeksi & QC sebagai dokumen) TERNYATA SUDAH SELESAI
  di repo (plan.md §STATUS I, 2026-08-24, POC 93/93). Diverifikasi ulang di lingkungan ini:
  layar SPK Inspeksi & QC (Gudang → Operasi Gudang → tab SPK Inspeksi & QC) render 2 SPK KSC
  + banner kebijakan; kebijakan default qc.color_mismatch_action=tahan /
  qc.handfeel_mismatch_action=peringatkan; release-hold oleh warehouse → 403
  (HOLD_RELEASE_ROLES = admin, manager); seed 3 SPK + 7 complaint_reasons ada.

- 2026-06 (Dasbor Tahanan QC): beranda manajer kini memuat kartu KPI "Barang ditahan QC"
  (klik → layar SPK Inspeksi, testid manager-home-qc-hold-kpi) + papan antrean
  `inspection_hold` (baris per SPK ber-tahanan). Backend: `MANAGER_BOARD_KEYS =
  HOME_BOARD_KEYS + ("inspection_hold",)` di home_service.manager_home — definisi antrean
  tetap satu di approval_backlog_service (INV-HOME-01), Control Tower admin tidak diubah.

- 2026-06 (FASE N + M ditutup): DRIFT terakhir FASE N dibereskan — notifikasi
  `contra_bon_cycle` kini beralamat `create_addressed(roles=("finance","manager"))`
  (contra_bon_reminder.py), 0 dokumen `recipient_role="all"`. FASE M dibukukan dengan POC
  baru `backend/test_core_makloon_lini_poc.py` (M1–M6, 35/35 PASS, nol residu).
  `audit_md_erp_readiness`: SELESAI=96 · BELUM=0 · DRIFT=0. Rencana MD-ERP habis.

- 2026-06 (sesi lanjutan ke-7 — penutupan & backlog): restore diverifikasi (audit 96/0/0,
  POC N 35/35, POC M 35/35, demo persis, residu 0); commit WIP di-amend jadi pesan penutup
  N+M. Backlog kecil selesai: P2 merged_max (POC approval 41/41), Bug #7 badge tab
  (center_delta 0.00px), 3 pintu RFID verify → DOOR_EXEMPT (penjaga 233 cek HIJAU).
  MASTER_ROADMAP EPIC 0 & 1 diverifikasi TERNYATA SUDAH DIBANGUN sesi lampau — flag
  ui.show_coming_soon terbukti mengendalikan sidebar (dibalik via API lalu dikembalikan),
  role-home per peran hidup, akses sales bersih dari HPP/vendor bill. Regresi
  iteration_274: backend 100% · frontend 100% · nol residu. Header MASTER_ROADMAP dibetulkan.

- 2026-06 (audit vs panduan training MD/Admin Sales): 2 gelombang testing agent
  (iteration_275 alur A/B/C/E/F, iteration_276 alur D/G/H/I/J/K/L + data demo bab 33).
  Mayoritas SESUAI-DOKUMEN; deviasi DITAMPUNG (belum diperbaiki, sesuai permintaan) di
  `TEMUAN_AUDIT_TRAINING.md`: 3 KRITIS (T1 gerbang confirm SO mati — bisa lompati
  verifikasi & ACC manajer; T2 antrean PIN jalan buntu utk sales_admin; T3 isolasi
  entitas bocor di detail/PDF inspeksi), 2 TINGGI (T4 putuskan-ulang pemenuhan 400,
  T5 revisi baris PO yg sudah diterima), 4 SEDANG, 2 MINOR, 7 temuan data-demo/dokumen.
  Seed demo di-reset via scripts/seed_reset.sh setelah audit.

- 2026-06 (sesi lanjutan ke-8 — repo dipindah ke github.com/pandeyoga/KN020926, restore
  via `.restore_env.sh`, demo persis 5·3·12·20 rolls). **3 temuan KRITIS audit training
  DIPERBAIKI:** T1 gerbang confirm SO (default verifikasi ON + 409 "belum disetujui manajer"
  bila status ≠ approved & approval_required), T2 PIN untuk sales_admin (DECIDER/CROSS roles
  di `routers/internal_requests.py`), T3 isolasi detail/PDF inspeksi
  (`entity_scope.assert_active_entity_access`). Bukti: POC E-8 desk 97/97, gate --full HIJAU,
  iter275 A4 + Alur F PASS, iter276 DD2 404.

## Backlog / Prioritas
- **P1 berikutnya (TEMUAN_AUDIT_TRAINING.md):** ~~T4~~ (iter278) · ~~T5~~ (iter279) · ~~T7–T8~~ (iter280,
  2026-09-02: SPK makloon hasil kurang → status Sebagian + `completion_hold` sampai klaim diputus;
  stage `makloon_claim` di /approvals/my-queue + kartu "Klaim Selisih Makloon"; IssueModal makloon
  menampilkan stok tersedia per gudang + 409 menuntun bila bahan kurang); lalu T9 (saran reorder
  R&D/warehouse_id), T10–T11 minor; paket seed demo D1–D5. T6 sudah array-aware di `PoBoardView.jsx`.
- ~~P2: refactor readability `merged_max` di PATCH approval_rules~~ — SELESAI 2026-06
  (ternary bersarang → dua baris jelas; POC approval coverage tetap 41/41, PATCH menolak
  min>max dengan benar). Bug #7 badge tab "Menunggu" juga SELESAI (screenshot before/after).
  Bonus: 3 pintu RFID verify (scan fisik label, bukan keputusan manusia) didaftarkan ke
  `DOOR_EXEMPT` penjaga INV-APPR-01 — penjaga kembali HIJAU 233 cek.
- Rencana fase MD-ERP selesai semua; backlog berikutnya dari BUG_BACKLOG.md / permintaan pemilik.

## Catatan Verifikasi Sesi Ini
- curl: POST rule entity ent_ksc → evaluate-approval mengembalikan rule_id tsb (mesin membaca);
  POST/PATCH entity bogus → 403; PATCH entity sah → OK.
- UI (screenshot): 9 aturan seed tampil, modal 3 kolom, dropdown entitas berisi 3 opsi,
  buat aturan ber-entitas KSC via UI berhasil (Cakupan = "PT Kain Suka Cita (KSC)"),
  Esc menutup modal, select Satuan/Grade 130px terbaca.
- Data uji dibersihkan; DB kembali 9 aturan seed, semua active.


## 2026-09-02 — Klon ulang repo pandeyoga/KNKN ke environment baru
- Sumber: https://github.com/pandeyoga/KNKN (commit 55bb32a). `.env` backend/frontend & `.emergent` environment dipertahankan.
- `.restore_env.sh` dijalankan: pip/yarn install, MongoDB hidup, bootstrap fondasi (expense_categories 8 · gl_accounts 75 · uoms 8), `seed_realistic.py` OK, frontend build OK.
- Verifikasi: login admin@/manager@/salesadmin@ (demo12345) 200, GET /api/sales-admin/desk 200, UI Meja Admin Sales tampil.
- Pengujian terakhir sebelum jeda: iteration_278 (T4 audit training / B guard UI / C pagar gudang E4.1) — belum dijalankan ulang di sesi ini.

## 2026-09-02 — Sesi #072: Restore repo skkajshs/sipro + penutup gelombang 1+2 catatan demo
- Restore via `.restore_env.sh` (`.env` dipertahankan), semua service RUNNING, FE bundle statis di-build.
- Titik henti iter281 DITUTUP: KNSelect Radix menghormati `opt.render` (terverifikasi UI), seed `artwork_parang.png` 480×360 (menggantikan 1×1 px, juga pada master lama), kontras `pabrik:` + 5 warna seed ber-`factory_name`.
- Regresi MD-05 diperbaiki: `scripts/seed_rnd_kpi_demo.py` & `backend/test_core_sampling_poc.py` — proofing tanpa hasil ukur (POC S 66/66, seed KPI desainer 4 permintaan).
- Verifikasi: testing agent iteration_282 BE 100% / FE 100%, pytest iter279–281 hijau.
- Status catatan demo: AS-01, MD-03, MD-04, MD-05, MD-07, PB-02 = SELESAI. **Berikutnya (gelombang 3, keputusan owner sudah ada):** AS-02, AS-03, MD-06, PB-01; lalu T9–T11 & seed D1–D5.

## 2026-09-02 — Sesi #073: Gelombang 3 catatan demo (AS-02 · AS-03 · MD-06) + lencana artwork
- AS-02: `PATCH /api/purchase-requisitions/{id}/lines/{line_no}` — qty beli PR (termasuk PR dari SO) boleh dinaikkan; `order_qty`/`extra_qty`/`qty_history`; UI "ubah qty" di detail PR.
- AS-03: `POST /api/sales-orders/{id}/items/{pid}/release-rolls` — Admin Sales melepas roll sebagian per baris; status SO tetap; jejak `reservation_releases`; UI "Lepas Roll" + log.
- MD-06: `GET /api/rnd/labdip-history` + modal Riwayat Labdip (Pustaka Warna & detail sample), tanggal butuh per putaran, deep-link + highlight putaran.
- Galeri Desain: lencana "Belum ada artwork" pada kartu tanpa berkas.
- Verifikasi: `iter283_as02_as03_scenario.py` 21/21, testing agent iteration_283 BE 100% / FE 100%.
- **Berikutnya:** PB-01 (blanket PO/kontrak → PO), MD-01/02/08, FB-01/02; audit training T9–T11; seed D1–D5.

## 2026-09-02 — Sesi #074: PB-01 · MD-01/02/08 · lencana labdip telat
- PB-01: kontrak blanket menyimpan termin (`payment_term`), PPN (`tax_mode`), harga incl/excl PPN → turun otomatis ke call-off/PO (`payment_due_date` = ETA + net_days); UI blok termin di modal buat & detail kontrak, baris termin di detail PO.
- MD-02/01: master benang (bahan, ply, puntiran, status celup + nomor/sistem) di produk stage Benang & spesifikasi R&D stage Benang; gramasi/lebar disembunyikan.
- MD-08: kode/nama versi supplier ikut ke katalog (`supplier_codes`); pencarian Master Produk & pemilih produk PR/PO/amandemen cocok di kedua sisi.
- Lencana merah "Labdip telat N" di kartu Pustaka Warna (putaran terbuka lewat tanggal butuh) → buka riwayat labdip.
- Verifikasi: testing agent iteration_284 BE 100% / FE 100% (pytest `test_iter284_pb01_md02_md08.py` 13/13).
- **Berikutnya:** FB-01/FB-02 (modul baru), audit training T9–T11, seed D1–D5.

## 2026-09-02 — Sesi #075: Feedback Pelanggan per SO · Ekspor Katalog Benang · Hutang Jatuh Tempo
- Feedback/komplain pelanggan per SO (`customer_feedbacks`, `/api/customer-feedback`): kategori, tingkat, penanggung jawab, tenggat, status open→in_progress→resolved→closed (penyelesaian wajib), timeline; panel di detail SO.
- Ekspor Katalog Benang CSV (`/api/master-data/export-yarn`) dengan kode/nama versi supplier; tombol di Master Produk.
- Meja Finance: antrean "Hutang supplier jatuh tempo" (lencana merah lewat tempo, ≤7 hari segera) → layar PO (finance kini hanya-lihat PO). Backfill `scripts/backfill_po_payment_due.py`.
- Verifikasi: testing agent iteration_285 BE 100% / FE lulus (bug navigasi Bayar diperbaiki + verifikasi Playwright).
- **Berikutnya:** FB-01 (AI Galeri Desain) / FB-02 catatan demo; audit training T9–T11; seed D1–D5.

## 2026-09-02 — Sesi #076: Klon ulang repo kakaudbdb/KN2 + FB-01 AI Galeri Desain ✅
- Repo diklon ulang ke env baru (`.restore_env.sh`), .env dijaga; `GEMINI_API_KEY=` ditambah (kosong → MODE DEMO).
- **FB-01 (BE)** `services/gemini_image_service.py` — Gemini "Nano Banana Pro" via `google-genai` LANGSUNG
  (model default `gemini-3-pro-image-preview`, alternatif `gemini-3.1-flash-image-preview`), 2 mode `mockup` / `modify`;
  bila key kosong (settings & env) → render demo lokal Pillow bercap "MODE DEMO". `design_gallery_service.add_ai_illustration`
  menyimpan hasil sebagai berkas `kind: "ai_illustration"` + `ai{mode,prompt,model,demo,source_file_id,by,at}` pada desain yang SAMA
  (keputusan pemilik: BUKAN versi baru, BUKAN artwork — ilustrasi ARAHAN atasan untuk di-rework desainer). Berkas lama tanpa `kind` = artwork.
  `submit/approve/autotag` kini menghitung ARTWORK saja. Endpoint `POST /design-gallery/{id}/ai-illustrate` (izin `_perm_manage`:
  admin/manager/designer), `GET /design-gallery-ai/status`. Integrasi: `system_settings.integrations.gemini {api_key, model, enabled}`
  via `PUT /admin/integrations` (`gemini_api_key|gemini_clear_key|gemini_model|gemini_enabled`), GET memuat `gemini.demo_mode`.
- **FB-01 (FE)** `features/hr/AiIllustrationPanel.jsx` (blok ungu di modal Kelola: mode, artwork acuan, prompt, daftar ilustrasi, pratinjau;
  testid `gallery-ai-*`), `features/hr/GalleryImage.jsx` (dipisah), kartu galeri: sampul = artwork pertama + lencana `gallery-card-ai-illus-*`;
  `features/admin/GeminiIntegrationPanel.jsx` di Pengaturan → Master Data & Audit → Integrasi AI (testid `gemini-*`).
- Testing agent iteration_286: BE 13/13 (`backend/tests/test_fb01_ai_illustration.py`, jalankan `-n 0` karena memutasi settings global), FE 100%.
- **Berikutnya:** isi GEMINI_API_KEY (Pengaturan → Integrasi AI) untuk mode LIVE; FB-02 (delivery tracking); audit training T9–T11; seed D1–D5.

## 2026-09-02 — Sesi #077: FB-02 MODUL LOGISTIK + komentar ilustrasi AI ✅
- **Keputusan pemilik:** sumber data MANUAL (ekspedisi+resi ATAU armada sendiri); foto WAJIB saat MUAT + POD; operator = gudang membuat,
  sopir (peran baru `driver`) mengupdate foto/posisi/tahapan; tahapan Disiapkan→Dimuat→Dalam perjalanan→Terkirim→Selesai / Gagal kirim;
  komentar ilustrasi AI oleh desainer + admin/manager.
- **BE Logistik** `services/logistics_service.py` + `routers/logistics.py` (prefix `/api/logistics`) + `schemas_logistics.py`: koleksi
  `logistics_deliveries` (SCOPED, nomor `LG-`), 1 pengiriman = ≥1 Surat Jalan (`shipments`) SATU pesanan; shipments diberi
  `logistics_id/number/status`. Endpoint: `meta`, `summary`, `shipments/unassigned`, `deliveries` CRUD (PATCH terkunci setelah terkirim),
  `photos` (multipart `kind=load|pod|other`, storage `logistics/`), `positions`, `transition` (gerbang: loaded↔foto muat; in_transit↔resi
  atau plat+sopir; delivered↔foto POD + `receiver_name`; failed↔alasan; failed→prepared). `order_journey_service.journey()` += `logistics`.
- **RBAC** modul `logistics`: admin/manager/warehouse `view,manage,update`; sales/sales_admin `view`; **peran ke-8 `driver`** `view,update`
  (`role_registry`, `permissions_config`, `roles.js`, `navMeta ROLE_HOME`, seed `driver@kainnusantara.id` Joko Susilo / demo12345).
- **FE** `features/logistics/{LogisticsView,DeliveryCreateModal,DeliveryDetailModal,DeliveryPhoto,logisticsApi}.jsx` — nav standalone
  `nav-logistics` (id/view `logistics`), kartu ringkasan per status, tabel, stepper, blok foto muat/POD (input `capture="environment"`),
  posisi, data ekspedisi/armada, riwayat (testid `logistics-*`). `OrderJourneyPanel` blok `journey-logistics`.
- **Komentar ilustrasi AI** `POST /design-gallery/{id}/files/{file_id}/comments` (izin `_perm_manage`: admin/manager/designer) →
  `files[].comments[]`; FE `IllustrationComments` di pratinjau `AiIllustrationPanel` (testid `gallery-ai-comment-*`), designer boleh berkomentar.
- Perbaikan kecil: ETA divalidasi `YYYY-MM-DD`; banner demo Gemini memakai font TrueType (FreeSans) agar terbaca.
- Testing agent iteration_287: BE 29/29 (`backend/tests/test_fb02_logistics.py`), FE 100%. Catatan minor terbuka: sales punya izin
  `logistics.view` tetapi tanpa menu sidebar (sengaja: pelacakan lewat Perjalanan Pesanan); 403 bootstrap `/document-templates`,`/uoms`
  untuk peran sempit (pola lama, tidak fungsional).
- **Berikutnya:** isi GEMINI_API_KEY → LIVE; integrasi API ekspedisi (opsional); audit training T9–T11; seed D1–D5.

## 2026-09-02 — Sesi #078: Peta posisi GPS + menu Logistik hanya-lihat untuk sales ✅
- `features/logistics/PositionMap.jsx` — Leaflet 1.9.4 (`yarn add leaflet`) + tile OpenStreetMap: titik posisi ber-GPS (merah = terakhir),
  garis putus-putus urutan perjalanan, popup lokasi/waktu; `logistics-map` / `logistics-map-empty`.
- `DeliveryDetailModal`: tombol **Ambil GPS** (`logistics-pos-gps`, `navigator.geolocation`) mengisi lat/lng ke `POST /positions`
  (skema `PositionIn.lat/lng` sudah ada); daftar posisi menampilkan koordinat (`logistics-pos-gps-{id}`).
- `navStructure` `logistics` roles += `sales`, `sales_admin` (izin BE `logistics.view` sudah ada) → `LogisticsView` mode hanya-lihat
  (`logistics-readonly`, tanpa tombol buat/aksi).
- **Berikutnya:** GEMINI_API_KEY → LIVE; integrasi API ekspedisi (opsional); notifikasi WA ke pelanggan saat berangkat/POD; audit training T9–T11.

## 2026-09-02 — Sesi #079: Tugas Sopir Hari Ini ✅
- BE: `GET /logistics/drivers` (manage) daftar akun peran `driver`; `POST /logistics/my-route {ids}` (update) → `route_order` 1..n hanya
  untuk pengiriman milik sopir (driver_user_id == actor); `GET /deliveries?mine=true` diurutkan route_order → ETA → terbaru.
- FE: `features/logistics/DriverTodayPanel.jsx` (hanya peran driver, di atas papan): nomor urut tujuan, pelanggan, alamat, status, ETA
  (+ lencana "Lewat ETA"), SJ, plat, posisi terakhir; tombol naik/turun menyimpan urutan; ringkasan "N tujuan aktif · M terkirim hari ini".
  `DeliveryCreateModal`: pilih akun sopir (`logistics-driver-select`) → mengisi `driver_user_id` + nama.
- **Berikutnya:** GEMINI_API_KEY → LIVE; notifikasi WA pelanggan; lacak resi API ekspedisi; audit training T9–T11.

## 2026-09-02 — Sesi #080: Navigasi ke tujuan (Google Maps) ✅
- FE saja: `logisticsApi.mapsUrl(address)` → `https://www.google.com/maps/dir/?api=1&travelmode=driving&destination=…`; tombol **Navigasi**
  (`driver-task-nav-{id}`) di tiap tugas sopir dan `logistics-detail-nav` di header detail pengiriman (target _blank → di ponsel membuka app Maps).
- Diverifikasi via screenshot (href benar, 2 tombol tugas + 1 di detail). Tidak ada perubahan backend.

## 2026-09-02 — Sesi #082: Audit UI/UX modul Logistik (tabel fungsional + navigasi silang) ✅
- `DeliveryTable.jsx` (dipisah dari LogisticsView): header **bisa diurutkan** (Nomor, Pesanan, Moda, ETA, Status; `logistics-sort-*`, aria-sort),
  baris klik/Enter/Spasi → detail, ETA terlambat merah "terlambat" (hanya status aktif), chevron, footer jumlah, **kartu di layar sempit**
  (`logistics-cards`, `logistics-card-*`). Empty state per filter + tombol `logistics-clear-filter`.
- **Navigasi silang** (`features/logistics/logisticsDeepLink.js` + `hooks/useLogisticsDeepLink.js`, terdaftar di `useDeepLinks`):
  nomor pesanan di tabel/detail (`logistics-open-order-*`, `logistics-detail-open-order`; admin/manager/sales/sales_admin) → Pesanan (SO)
  terpilih via `focusDoc`; dari Perjalanan Pesanan tombol `journey-logistics-open-*` → Logistik + detail otomatis terbuka
  (`LogisticsView.focusDelivery`). Escape menutup modal detail & buat.
- Testing agent iteration_290: 7/7 skenario FE lulus (admin/warehouse/sales/driver, mobile 390px). Catatan minor pra-eksisting:
  ETA date-input bawaan; foto seed hitam; 403 bootstrap `/uoms` & `/document-templates` peran sempit.

## 2026-09-02 — Sesi #083: AUDIT menyeluruh sesi ini (temuan DITAMPUNG, belum dikerjakan) 📋
- Testing agent iteration_291 (audit eksploratif, tanpa mengubah kode) + tinjauan kode → **`memory/AUDIT_TEMUAN_2026-09-02.md`**.
- Ringkas: **P0-1** tab Audit (Pengaturan) mem-blank seluruh SPA (`JSON.stringify(log.after).slice` pada log tanpa `after`; tanpa error
  boundary) · **P1-1** sopir bisa menulis ke pengiriman bukan tugasnya (tidak ada cek `driver_user_id == actor.id`) · **P1-2** Surat Jalan
  tidak menampilkan `logistics_number/status` · **P1-3** tidak ada jalur balik `loaded→prepared` · P2: tanggal "hari ini" UTC (bukan WIB),
  validasi lat/lng, `/my-route` id selesai, pencarian hanya Enter, empty state pencarian, konfirmasi hapus ilustrasi AI, badge NONAKTIF,
  tombol Uji koneksi Gemini, Escape modal galeri, label matriks izin/audit raw key, dll.
- **Berikutnya (prioritas sesi depan):** P0-1 → P1-1 → P1-2 → P1-3 → L-1 (zona waktu) → sisanya per tabel di berkas audit.

## 2026-09-02 — Sesi #081: Telepon / WA penerima di tugas sopir ✅
- BE: pengiriman punya `receiver_phone` + `receiver_name_hint` — otomatis saat dibuat dari `sales_orders.shipping_address.{recipient_name,phone}`
  → kontak utama `customers.contacts[is_primary]` → `customers.phone` (`_receiver_contact`); bisa diisi manual (create/PATCH `receiver_phone`).
  Pengiriman lama sudah di-backfill (5 dokumen).
- FE: `logisticsApi.telUrl/waUrl` (08xx → 628xx, pesan WA prefilled "sopir membawa kiriman LG-… sebentar tiba"); tombol **Telepon** & **WA**
  di tiap tugas sopir (`driver-task-call-*`, `driver-task-wa-*`, baris `driver-task-phone-*`) dan di header detail (`logistics-detail-call/wa/phone`);
  field "Telepon penerima" di modal buat & data detail (`logistics-receiver-phone`, `logistics-edit-receiver_phone`).



## 2026-09-02 — Sesi #084: Klon ulang repo avacadasa/kn + verifikasi & perbaikan AUDIT INDEPENDEN 2026-09-02 ✅
- Restore: `git clone` → `/app`, pip/yarn, `seed_reset.sh`, rebuild bundle statis. Gate 246 PASS.
- Dokumen pemilik `memory/AUDIT_KN_2026-09-02_owner.md` diverifikasi butir per butir; status lengkap di
  **`memory/AUDIT_KN_2026-09-02_STATUS.md`**.
- DIPERBAIKI (semua VALID, terbukti runtime): F-01 (pendapatan/HPP dijurnal saat dispatch & kwitansi AR),
  F-02 (backfill tahan periode tertutup, startup tak mati), F-03 (makloon dobel-posting + migrasi 6 JE
  reversal, Hutang −Rp 3.481.500), F-04 (`gl_posted` dihormati), F-05/F-06 (kwitansi AR: pemilik/entitas +
  pra-validasi & kompensasi), F-07 (simulate-payment pagar lebih-bayar + `paid_total`), F-08 (netting
  dibatasi & mengkonsumsi piutang balik; bawaan `transfer`), E-01 (stock-breakdown scoped+proyeksi),
  E-02 (cycle-count scoped), U-01 (remount saat ganti PT), U-02 (`/documents/{id}/print`),
  U-04/P0-1 (tab Audit + `ErrorBoundary`), gate `INV-CFG-01` (`__migrations__` diabaikan).
- Invarian baru di `verify_data_integrity.py`: INV-GL-DUP-01, INV-CASH-02, INV-AR-02, INV-GL-REV-01.
- Skrip: `scripts/migrate_reverse_duplicate_backfill_je.py --report|--apply`.
- Bukti: `backend/test_audit_2026_09_02_poc.py` (21/21), `test_reports/iteration_292.json` (17/17 + 3/3 UI).
- DITUNDA (perlu keputusan pemilik): B-01 kebijakan pengakuan pendapatan, F-09..F-17 (retur/PPN/GR/deposit/
  kontrabon), D-01 penomoran atomik, D-02/D-03, U-03/U-05, serta backlog P1-1..P1-3/L/G/X dari
  `AUDIT_TEMUAN_2026-09-02.md`.

## 2026-09-03 — Sesi #085: Backlog AUDIT_TEMUAN_2026-09-02 tuntas (P1 + seluruh P2) ✅
- Logistik: P1-1 pagar tulis sopir (403 pada pengiriman bukan tugasnya; lihat tetap boleh), P1-2 chip `LG-xxxxx · status` +
  "Buka di Logistik" pada baris Surat Jalan di detail pesanan, P1-3 "Bongkar" loaded→prepared (manage, alasan, konfirmasi),
  L-1 tanggal WIB (FE `todayWib()`, BE `today_wib()`), L-2 validasi koordinat + hapus posisi, L-3 my-route hanya aktif,
  L-4 pesan gabungan + tombol Terkirim disabled + prefill penerima, L-5 debounce + tombol Cari, L-6 empty state pencarian,
  L-7 hint di atas tombol/caption title/tap target 40px, L-8 hanya SJ dispatched, L-9 notifikasi terkirim/gagal ke sales,
  L-10 konfirmasi tahapan, L-11 Esc per lapisan (`useEscapeClose`).
- Galeri AI: G-1 konfirmasi hapus + jumlah komentar, G-2 badge NONAKTIF, G-3 uji koneksi Gemini (`verified_at`, LIVE hanya setelah lulus),
  G-4 banner demo besar + "Ukuran penuh", G-5 Esc modal galeri, G-6 badge komentar + hapus komentar sendiri + notifikasi desainer,
  G-7 resize ≤2048/timeout 60 s/JPEG, G-8 batas harian per desain + estimasi biaya.
- Lintas modul: X-1 label Indonesia matriks izin (`config/auditLabels.js`), X-2 label aksi audit + render bertahap 50,
  X-3 tooltip sidebar, X-4 lencana `FB-2`, X-5 divisi Logistik.
- Perbaikan ikutan (temuan iter293): `ConfirmHost` kini meneruskan `description`; bootstrap peran sempit tidak memanggil master tanpa izin.

## 2026-09-03 — Sesi #086: Penomoran atomik (D-01) · KNDatePicker ETA · Template PDF paritas sipro ✅
- **D-01**: `next_doc_number` mode bersama kini sequence atomik (`number_sequences` `__shared__`, disemai dari nomor tertinggi);
  `customers`/`crm`/`invoices` tidak lagi `count()+1`; index unik `(entity_id, doc_type)`; gate INV-NUM-01/02. 40× serempak → 40 unik.
- **ETA**: `components/KNDatePicker.jsx` (Popover+Calendar, locale `id`, ISO ↔ "Sel, 15 Sep 2026") dipakai di detail/buat pengiriman;
  tabel & panel sopir menampilkan tanggal Indonesia.
- **Template PDF** (analisis `memory/ANALISIS_TEMPLATE_PDF_SIPRO_2026-09-03.md`): lapisan `__default__` → override per jenis (diff saja),
  `GET/PUT/DELETE /api/pdf/templates[/{code}]`, `validate-script`, naskah pembuka/penutup ber-placeholder tervalidasi (400 bila asing),
  gaya tabel, bagian dokumen on/off, mode kop/footer (sistem/gambar/tanpa), nomor halaman, tempat-tanggal, meterai, cap, catatan sistem,
  branding + tagline/email/website/gambar kop/footer/cap, versi + audit. UI: tab Naskah (chip + peringatan hidup) & Tabel, info lapisan, reset via server.
- Bukti: `test_reports/iteration_294.json` (100% backend & UI), gate 248 PASS.

## 2026-09-03 — Sesi #087: Meja MD · Meja Admin Gudang · DetailModal `framed` · Jembatan WMS→Logistik ✅
- Peran baru `md` (md@) & `warehouse_admin` (wh.admin@) — `permissions_config`, `role_registry`, `roles.js` ROLE_NAV, `navMeta` ROLE_HOME.
  BE `GET /api/md/desk` (desain · sample · pr · acuan) & `GET /api/warehouse-admin/desk` (sj_belum_diangkut · outbound · inbound · spk ·
  persetujuan_gudang · logistik) di `work_desk_service.py`; FE `features/desks/RoleDesk.jsx` (testid `md-desk-*`, `wh-desk-*`).
- `DetailModal` prop `framed` (kartu putih untuk panel tanpa kartu sendiri) dipakai `ins-detail-modal` & `dsr-detail-modal`.
- Jembatan WMS→Logistik: tombol `create-delivery-{taskId}` setelah dispatch outbound & aksi `create_delivery` di Meja Admin Gudang →
  `openLogistics({createFromShipmentId})` → LogisticsView membuka `DeliveryCreateModal` dengan SJ terpilih (`preselectShipmentId`).
- Perbaikan pasca-uji (iteration_295): `md` masuk `FULL_VIEW_ROLES` di `routers/design_requests.py` (sebelumnya daftar 0 & detail 403);
  `openDocument` kini meneruskan `link.tab` → "Buka WMS" dari meja mendarat di tab Barang Keluar/Transfer/Stock Opname.
- Bukti: `test_reports/iteration_295.json` (BE 8/9 → 9/9 setelah fix, FE 100%), pytest `backend/tests/test_iter295_role_desks_bridge.py`.
- Data uji tersisa (boleh dihapus manual): KSC/SJ-00007, KSC/LG-00009..00011 (plat 'TEST …').
- **Backlog (opsional):** tinjau integrasi Pembelian → WMS Inbound seperti pola Outbound→Logistik.

## 2026-09-03 — Sesi #088: Tenggat DSR Indonesia · Pilih tugas otomatis · Jembatan PO→Gudang ✅
- DSR: `dsr-due-input` (modal Buat) & `dsr-assign-due` (Tugaskan) → `KNDatePicker`; tenggat detail/daftar/kanban via `formatDateId`;
  `DesignRequestsView` menerima `focusDoc` (auto-buka rincian dari Meja MD) dan `canCreate` += `md`.
- WMS: `OperationsView` menerima `focusDoc/onClearFocus` → `OutboundScanInterface.focusTaskId` (sorot + buka tugas, scroll) dan
  `InboundScanInterface.focusPoId` (pilih tugas penerimaan PO yang belum selesai). `openDocument` meneruskan `link.tab`.
- PO: tombol **Terima Barang di Gudang** (`receive-goods-button` di POCompactPanel, `popup-receive-goods-button` di PODetailPanel) untuk
  status pending/receiving/partial bila `wms.view` → Operasi Gudang tab Barang Masuk dengan tugas PO terpilih.
- KNDatePicker: tombol clear dipindah ke luar `PopoverTrigger` + `preventDefault` (akar masalah: pembungkus `<label>` meneruskan klik
  clear ke trigger → popover terbuka lalu tertutup di klik berikutnya). Kolom Tenggat di blok Tugaskan 140→190px.
- Bukti: `test_reports/iteration_296.json` FE 7/7 alur lulus; bug minor clear KNDatePicker diperbaiki & diverifikasi Playwright.
- Catatan state demo: KSC/DSR-00001 kini ditugaskan ke Sari Melati (efek uji); KSC/DSR-00005 dibatalkan (data uji).

## 2026-09-03 — Sesi #089: KEBIJAKAN PENDAPATAN (KEB-PDPT) · Lompat balik ke PO ✅
- **Kebijakan (disetujui pemilik, tahap 1):** pendapatan & HPP HANYA saat dikirim (`REVENUE_STATUSES`); pembayaran tidak memicu.
  Kwitansi untuk pesanan belum dikirim → `Dr Kas / Cr 2-1400 Uang Muka Pelanggan`, payments[].gl_bucket="advance".
  Saat kirim → JE sales_order + sales_cogs + **`advance_reclass`** (Dr 2-1400 / Cr 1-1200). Void kwitansi uang muka setelah reklas →
  `advance_reclass_reversal` (source_id `{order_id}:{receipt_id}`). Alokasi deposit (G-3) ke pesanan belum dikirim → tetap 2-1400.
  Historis dibiarkan (prospektif) — tampil di Meja Finance antrean `uang_muka_belum_kirim` badge "diakui (historis)".
  Helper: `gl_service.order_revenue_posted / order_advance_total / tag_advance_payment / order_advance_unrecognized / post_advance_reclass*`.
- UI: Meja Finance metrik `fin-desk-metric-advance` + antrean baru (row testid kini `fin-desk-{queueId}-row-{refId}` agar unik);
  journey `revenue_recognized`/`advance_unrecognized` → `journey-advance-note`; `so-compact-advance-note`.
- PO ↔ Gudang bolak-balik: `inbound-back-to-po-{taskId}` di panel Scan Receive → detail PO; `receive-goods-button` kembali ke Barang Masuk.
- `warehouse_admin` mendapat `supplier.view` (hilangkan 403 saat layar PO memuat pemasok).
- Bukti: `test_reports/iteration_298.json` (BE 18/18, FE 100%), pytest `test_iter297_revenue_policy.py` (layanan) &
  `test_iter298_kebpdpt_e2e.py` (E2E HTTP, mutating + self-clean). Efek: lubang penomoran SO-00131..133, AR-00010..14, FKT-00003 (terbit tak sengaja saat uji).
- **Tahap 2 (backlog):** pro-rata pendapatan untuk `partially_shipped` (per surat jalan). `test_fb02_logistics.py` 5 gagal karena seed drift (bukan regresi).

## 2026-09-03 — Sesi #090: KEB-PDPT tahap 2 (PRO-RATA per surat jalan) · Laporan Uang Muka ✅
- Pendapatan & HPP per SURAT JALAN: `post_shipment_revenue` / `post_shipment_cogs` (source_id = shipment id, `ref.order_id`),
  porsi = line_total × qty_kirim/qty_baris ÷ total nilai pesanan; SJ penutup (shipped/done) mengambil SISA → total tepat = grand.
  Reklas uang muka bertahap (`post_advance_reclass` posting selisih; source_id order_id lalu `{oid}:rc{n}`); pembalik = min(kwitansi, reklas netto).
  `post_sales_order`/`post_order_cogs` (per pesanan) hanya untuk pesanan TANPA surat jalan; pesanan lama ber-JE `sales_order` tidak disentuh.
  `reverse_order_journals` membalik juga shipment_*/advance_reclass. `_insert_entry(ref=...)` menyimpan tautan dokumen induk.
- Laporan Uang Muka Pelanggan: `services/advance_report_service.py`, `GET /api/ar/advance-report?entity_id=&q=` (izin accounting.view /
  penalty.issue — sales 403), view `advance-report` (menu Keuangan, `features/finance/AdvanceReportView.jsx`), totals mengikuti filter q.
- Journey: `revenue_recognized_total/pct` → `journey-revenue-prorata`. Meja Finance antrean uang muka: value = sisa uang muka (incl. partially_shipped).
- FKT-00003 sudah tidak ada (terhapus bersama SO uji); JE orphan KSC/JE-00102 (residu uji) dihapus.
- Bukti: iteration_299 (BE 40/41 → RBAC diperbaiki, FE 100%); pytest iter297 (5), iter298 (15), iter299 (21). Fixture E2E kini memulangkan roll
  (`tests/iter299_restore_orphan_rolls.py`). Nomor terpakai: SO-00137..142, AR-00021..26, SJ-00017/18.

## 2026-09-03 — Sesi #091: Audit MEJA KERJA — "satu klik, tiba tersaring" ✅
- `rowLink` mengirim `{focus_type, focus_id, number, tab}`; `openDocument` menyimpan number/tab di `focusDoc`. Layar tujuan menyaring
  tabel + membuka detail: OrdersView (search=nomor), SalesReturns, InternalRequests, Inspections (search + modal), PurchaseRequisitions,
  ARAgingView (search=nama + rincian pelanggan), PaymentPlansView (tab + q), CustomerList (search + Customer 360), RndSamples (focusDoc→focus),
  LogisticsView (`focusDelivery.search` / deliveryId via openLogistics), CycleCount (`focusSessionId`), TransferManagement (`focusTransferId`).
- Target Finance: siap_faktur_pajak & uang_muka → Pesanan; jatuh_tempo → AR Aging per pelanggan (row.customer_id); selisih/denda → tab Rencana Bayar.
  Meja Admin Gudang: PO → langsung Barang Masuk (WH_DESK_TARGET). Baris uang_masuk ber-`row_key` (testid unik per pesanan).
- Kualitas baris: PO inbound 'diterima X/Y · n baris · ETA'; PR judul bahan + subtitle qty/gudang/supplier; SPK label jenis + PO; DSR internal
  menyebut jenis; logistik 'SJ · moda · sopir · plat · ETA · alasan gagal'; outbound 'gudang · pelanggan · diambil X/Y'; opname 'n item · selisih · menunggu ACC';
  kamus BADGE_LABEL diperluas (packing/loading/delivered/failed/…).
- Bukti: iteration_300 (13/15 → 15/15 setelah CycleCount/Transfer focus); pytest iter295 lulus.
- Backlog kecil: pencarian OrdersView substring (SO-0001 ikut menarik SO-00010); `rnd-samples-search` tidak diisi; seed demo untuk antrean
  Finance yang kosong (selisih_bayar, denda_draft, hutang_jatuh_tempo); 403 noise /api/warehouses & /api/esign untuk peran md.

## 2026-09-03 — Sesi #092: Master Data UX · RBAC Pusat Persetujuan · IA Produk & Harga ✅
- Pencocokan nomor tepat: `build_search` mengenali q ber-tanda kutip ("SO-0001") → regex ^…$; lompatan dari meja mengisi `"nomor"`.
- RBAC: `approval-inbox` dicabut dari ROLE_NAV md & warehouse_admin; `approval.view` dicabut (permissions_config + bootstrap REVOKE) untuk
  md, finance, warehouse_admin → /api/approvals/* 403. Persetujuan domain mereka tetap di meja masing-masing.
- Modal selalu terpusat: FormModal body max-h calc(100dvh-12rem), DetailPopup calc(100dvh-9rem), DetailModal calc(100dvh-2rem).
- Ubah Data Master (produk): Impor/Ekspor dipindah ke kepala Records (`admin-toggle-import-export`); field berlabel + penjelasan;
  catatan "harga master = harga dasar pesanan BARU, SO lama tidak berubah"; blok "Isi roll standar produk ini (konversi khusus)".
- Satuan & Konversi: form UOM berlabel (dimensi KNSelect, tombol simpan tervalidasi, tutup & reset setelah sukses), baris informatif,
  tombol Update tanpa fungsi dihapus, UomConversionView tertanam di tab yang sama; tab "Konversi Satuan" dihapus.
- IA: Registri Domain → hub Pengaturan; "Harga per Badan Usaha" + "Harga per Pelanggan" berdampingan di Produk & Harga (menu
  "Daftar Harga per Pelanggan" dihapus; sales melihat Produk & Harga hanya tab Harga per Pelanggan). Template Varian diberi intro (`tpl-modal-intro`) + Esc-close.
- Kode internal (FASE/PS/D/MD/KN_18) dihapus dari UI: `utils/cleanText.js stripInternalCodes` untuk teks dinamis (registri, aturan UOM),
  literal JSX dibersihkan, note aturan UOM standar disinkron ke DEFAULT (idempotent) & DB dibersihkan.
- Bukti: iteration_301 (8/10 → fix), iteration_302 (BE 100%, FE 3/4 → sisa kode internal dibersihkan & diverifikasi regex 0 kecocokan),
  pytest test_iter301_rbac_pricing_uom.py (23+1 skip), test_iter302_approval_rbac.py (16).
- **Usulan IA lanjutan (menunggu keputusan pemilik):** zona sidebar KERJA SAYA / MODUL / ALAT / AKUN; lebur Beranda+Pusat Persetujuan ke Meja
  per peran; Logistik ke bawah Gudang & Logistik; penamaan menu 1–2 kata; alamat dokumen `?doc=`.

## 2026-09-03 — Sesi #093: Zona Sidebar · Logistik ke Gudang · Alamat Dokumen ?doc= ✅
- Sidebar 4 zona (`NAV_ZONES`/`zoneOf`/`withZones` di navigationConfig.js; render `nav-zone-*` di CoreWidgets): Kerja Saya (Beranda,
  Pusat Persetujuan, Meja-meja, Eskalasi) · Modul · Alat (Analytics, BI, Pusat Dokumen/Cetak, Pengaturan, Segera Hadir) · Akun (Profil).
- Logistik = item `logistics` di grup "Gudang & Logistik" (roles grup ditambah driver/sales/sales_admin/warehouse_admin/md); standalone dihapus.
- Alamat dokumen: `GET /api/documents/resolve?number=` (14 jenis, cocok tanpa awalan PT, tersaring izin modul peran) →
  `useViewDeepLink` (?doc=) → `docLinkTarget` (workDeskApi) → openDocument/openLogistics; toast bila tak ditemukan; URL dibersihkan.
  `utils/docLink.js` (docLink/waShareLink/copyDocLink); tombol `so-compact-copy-link` & `so-compact-share-wa` di panel ringkas SO.
  `useLogisticsDeepLink` meneruskan `search` (bug lama: baris shipment di meja tidak menyaring Logistik).
- Bukti: iteration_303 (BE 100%, FE 5/6) → iteration_304 (100/100); pytest test_iter303_doc_resolve.py 12/12.
- Backlog: tombol salin/WA juga di panel PO, SJ, DSR; nomor dokumen di WhatsApp otomatis jadi tautan (butuh domain publik).

## 2026-09-03 — Sesi #094: Nama menu singkat ✅ (self-test screenshot)
- 37 label menu diringkas 1–2 kata Indonesia (Kasir & Portal, Pesanan, Pelanggan, Pengadaan, Pesanan Pembelian, Hutang Pemasok,
  Bahan Cetak, Riset & Sampel, Operasi Gudang, Produksi, Lokasi Rak, Data Gudang, Pengiriman, RFID, Tag RFID, Perangkat, Monitor Gerbang,
  Piutang, Rencana Bayar, Kasus Keuangan, Saldo Kredit, Pajak, Buku Besar, Laporan, Tutup Buku, Buka Periode, Kas Kecil, SDM, Karyawan,
  Kehadiran, Penggajian, KPI, Analitik, Dasbor Penjualan, Dasbor Stok, Pengaturan, Meja Gudang).
- Singkatan/istilah teknis (POS, CRM, WMS, PO, AP, BOM, R&D, BI, Payroll) dipindah ke judul halaman: PAGE_META ditambah untuk
  orders, purchasing, operations, suppliers, makloons, amendments, costing, reorder, reports, documents, admin, escalations, home.

## 2026-09-03 — Sesi #095: Kolom nilai meja selalu berformat ✅ (self-test Playwright 3 meja)
- DeskQueueCard: `value_kind` money → `formatCurrency` (Rp, dipaksa Number); qty → angka + satuan; count → kosong per baris & ringkasan "n dokumen"
  (sebelumnya count dirender "Rp 0"). Baris outbound Meja Gudang kini membawa `unit` (yard) sehingga tampil "40 yard".
