# Zyx Authoring MCP

Repository ini adalah plugin client Zyx Authoring MCP untuk Codex dan Claude. Repository ini tidak menjalankan MCP server secara lokal.

Server MCP tetap berjalan pada endpoint remote berikut:

`https://staging.zyxacademy.com/api/mcp/authoring`

Endpoint saat ini adalah staging. Jangan menggantinya menjadi production tanpa instruksi eksplisit.

## Isi release plugin

- `.codex-plugin/plugin.json` adalah manifest Codex.
- `.claude-plugin/plugin.json` adalah manifest Claude.
- `.mcp.json` adalah konfigurasi MCP bersama.
- `claude-desktop-config.example.json` adalah contoh konfigurasi untuk Claude desktop / chat tab.
- `skills/` berisi lima skill authoring:
  - `zyx-authoring-pipeline` untuk mengorkestrasi Source Pack hingga Product Bundle secara resumable.
  - `zyx-source-pack` untuk membuat dan memvalidasi Source Pack.
  - `zyx-idea-bundle` untuk membuat Idea Bundle dari Source Pack tervalidasi.
  - `zyx-product-bundle` untuk membuat Product Bundle draft dari Idea yang sudah dipublikasikan.
  - `zyx-question-bank` untuk membuat draft bank soal yang tertaut Idea published dalam workflow `quiz_bank`.
- `README.md` dan file reference skill adalah dokumentasi client.

Salinan server-side `lib/authoring` tidak termasuk release scope repository ini. Source authoritative berada di repository `zyx-web`.

---

## Daftar Lengkap MCP Tools (Actual Codebase)

Berdasarkan implementasi server pada `lib/authoring/mcp-server.ts` di `zyx-web`, server Authoring MCP mengekspos **27 tools** yang terbagi dalam kategori fungsional:

### 1. Workflow & Scope Management
| Tool | Scope | Deskripsi |
|---|---|---|
| `workflow.list` | `authoring:read` | Mengembalikan pilihan workflow authoring (`idea_product`, `quiz_bank`). |
| `catalog.list_courses` | `authoring:read` | Mengembalikan daftar pilihan mata kuliah dengan key sementara yang aman (opaque key). |
| `catalog.list_chapters` | `authoring:read` | Mengembalikan daftar bab/chapter aktif untuk mata kuliah yang dipilih. |
| `source.ingest` | `authoring:read` | Menerima ZIP Source Pack (base64) & binary dokumen asli (`originals`), menjalankan verifikasi checksum, struktur, visual, dan menghasilkan `sourcePackToken`. |
| `workflow.start` | `authoring:read` | Mengunci mata kuliah, bab, dan workflow authoring. Menghasilkan `runToken` (berlaku 7 hari). |
| `workflow.get_run` | `authoring:read` | Mengembalikan metadata scope yang terkunci pada `runToken` tanpa membuka ID internal database. |
| `workflow.get_contract` | `authoring:read` | Mengembalikan aturan output, contract schema, enum guidance, dan compatibility gate untuk run aktif. |

### 2. Authoring Validation & Staging (Idea & Product)
| Tool | Scope | Deskripsi |
|---|---|---|
| `authoring.validate_idea_bundle` | `authoring:read` | Memeriksa integritas ZIP Idea Bundle V2 terhadap schema, deterministic checksum, provenance, kebersihan relasi siklus, dan quality report. |
| `authoring.submit_idea_bundle` | `authoring:stage` | Memvalidasi ulang dan menaruh Idea Bundle ke staging review inventory admin (`admin_review`). |
| `authoring.validate_product_bundle` | `authoring:read` | Memeriksa ZIP Product Bundle V2 (Artikel, Diktat, flashcard, soal ITB, dependency, provenance, dan quality metrics). |
| `authoring.submit_product_bundle` | `authoring:stage` | Memvalidasi ulang dan menaruh draft Product Bundle untuk review admin (`admin_review`). |

### 3. Assessment Authoring & Staging (Bank Soal)
| Tool | Scope | Deskripsi |
|---|---|---|
| `assessment.list_ideas` | `authoring:read` | Mengambil katalog Idea published pada scope terkunci, dengan pencarian teks, filter (`knowledgeKinds`, `instructionalRoles`, `difficultyLevels`), facets, dan pagination. |
| `assessment.get_idea` | `authoring:read` | Mengambil detail Idea published, relasi aktif (*prerequisite*, *related*, *extends*, dll.), dan ringkasan provenance sumber. |
| `assessment.validate_quiz_draft` | `authoring:read` | Memeriksa draft soal `question-bank-draft.v2` (struktur soal, validasi bobot relasi Idea = 1, coverage gap, pembahasan). |
| `assessment.submit_quiz_draft` | `authoring:stage` | Memvalidasi ulang dan menyimpan draft soal canonical berstatus `generated` untuk review admin. |

### 4. Knowledge Base Inspection
Tersedia untuk semua workflow guna meninjau materi, sumber, dan produk yang telah ada di dalam scope:
| Tool | Scope | Deskripsi |
|---|---|---|
| `knowledge.search_ideas` | `authoring:read` | Mencari dan memfilter Idea published dalam scope mata kuliah dan bab terkunci. |
| `knowledge.get_idea` | `authoring:read` | Mengambil detail lengkap Idea published beserta relasi aktif dan ringkasan provenance. |
| `knowledge.list_products` | `authoring:read` | Mengembalikan ringkasan produk (artikel, diktat, set flashcard, soal) yang sudah ada dalam scope chapter, dihitung per tipe dan per Idea. |
| `knowledge.get_product` | `authoring:read` | Mengambil detail satu produk spesifik (`article`, `diktat`, `flashcard_set`, atau `question`) beserta link Idea dan relasinya. |
| `knowledge.list_sources` | `authoring:read` | Mengembalikan daftar sumber materi aktif dalam scope bab beserta jumlah chunk masing-masing. |
| `knowledge.get_source` | `authoring:read` | Mengambil detail sumber materi beserta seluruh chunk (start/end offset, sectionLocator, checksum). |
| `knowledge.search_source_chunks` | `authoring:read` | Mencari chunk sumber materi berdasarkan query teks (potongan teks sumber yang menjelaskan topik tertentu). |

### 5. Content & Coverage Analysis
| Tool | Scope | Deskripsi |
|---|---|---|
| `analysis.get_coverage` | `authoring:read` | Mengembalikan matriks cakupan Idea terhadap sumber, artikel, flashcard, dan soal dalam scope run. Menunjukkan Idea mana yang sudah tertutup dan yang masih uncovered. |

### 6. Assessment & Question Bank Analysis
| Tool | Scope | Deskripsi |
|---|---|---|
| `assessment.list_questions` | `authoring:read` | Menampilkan daftar soal published dalam scope run dengan filter `difficulty`, `cognitiveLevel`, `questionType`, `origin`, query teks, dan pagination. |
| `assessment.get_question` | `authoring:read` | Mengambil detail satu soal beserta snapshot opsi, explanation, dan link Idea. |
| `assessment.analyze_bank` | `authoring:read` | Menganalisis distribusi bank soal berdasarkan difficulty, cognitive level, origin, Idea coverage, uncovered Ideas, dan over-represented Ideas (>20%). |
| `assessment.find_similar` | `authoring:read` | Mencari soal serupa dalam scope bab berdasarkan tingkat kesulitan, cognitive level, tipe soal, dan kemiripan teks prompt untuk mencegah duplikasi. |

---

## Autentikasi dan Otorisasi

Server remote menggunakan otentikasi sesi admin Zyx:
- **OAuth 2.0 / MCP connector**: Digunakan secara interaktif oleh host yang mendukung.
- **Connection Token**: Token sementara berumur pendek via `POST /api/mcp/authoring/connection` untuk host tanpa interaksi OAuth.
- **Scope**:
  - `authoring:read`: Diperlukan untuk membaca katalog, menginspeksi materi, dan melakukan validasi draft/bundle.
  - `authoring:stage`: Diperlukan untuk staging submission (`authoring.submit_*` dan `assessment.submit_quiz_draft`).

> [!IMPORTANT]
> Connection token tidak boleh ditulis ke `.mcp.json`, README, source code, atau repositori Git.

---

## Workflow dan Staging Enforcement

MCP remote menyediakan dua workflow utama:
1. `idea_product`: Untuk pembuatan Idea Bundle dari Source Pack, lalu dilanjutkan pembuatan Product Bundle dari Idea published.
2. `quiz_bank`: Untuk pembuatan draft bank soal dari Idea published.

**Sifat Staging (No Direct Publication):**
MCP remote **tidak memiliki publication tool**. Seluruh tool submission (`submit_idea_bundle`, `submit_product_bundle`, `submit_quiz_draft`) menyimpan konten ke antrean review berstatus draft/staging/generated (`admin_review`). Publikasi akhir ke platform pembelajaran dilakukan oleh admin Zyx melalui dashboard web.

---

## Batas Ukuran Payload (Limits)

- **Source Pack ZIP**: Maksimum 64 MB (`AUTHORING_MCP_MAX_SOURCE_PACK_BYTES`).
- **Bundle ZIP (Idea/Product)**: Maksimum 32 MB (`AUTHORING_MCP_MAX_BUNDLE_BYTES`).
- **Binary Asli (Originals)**: Maksimum 32 MB per file, total maksimum 96 MB (`AUTHORING_MCP_MAX_ORIGINAL_BYTES`).
- **Draft JSON Bank Soal**: Maksimum 8 MB, maksimal 500 soal per draft.
