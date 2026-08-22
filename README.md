# Zyx Authoring MCP

Repository ini adalah plugin client Zyx Authoring MCP untuk Codex dan Claude. Repository ini tidak menjalankan MCP server secara lokal.

Server MCP tetap berjalan pada endpoint remote berikut:

`https://staging.zyxacademy.com/api/mcp/authoring`

Endpoint saat ini adalah staging. Jangan menggantinya menjadi production tanpa instruksi eksplisit.

## Isi release plugin

- `.codex-plugin/plugin.json` adalah manifest Codex.
- `.claude-plugin/plugin.json` adalah manifest Claude.
- `.mcp.json` adalah konfigurasi MCP bersama.
- `claude-desktop-config.example.json` adalah contoh konfigurasi untuk Claude chat tab.
- `skills/` berisi empat skill authoring.
- `README.md` dan file reference skill adalah dokumentasi client.

Salinan server-side `lib/authoring` tidak termasuk release scope repository ini. Source authoritative tetap berada di repository `zyx-web`; snapshot lokal sebelumnya dikeluarkan agar tidak disalahartikan sebagai runtime server atau source of truth.

## Empat skill

- `zyx-source-pack` untuk membuat dan mengirim Source Pack.
- `zyx-idea-bundle` untuk membuat Idea Bundle dari Source Pack tervalidasi.
- `zyx-product-bundle` untuk membuat Product Bundle draft dari Idea yang sudah dipublikasikan.
- `zyx-question-bank` untuk membuat draft bank soal yang tertaut Idea published dalam workflow `quiz_bank`.

Skill adalah instruction layer. Skill tidak menggantikan enforcement layer di server MCP remote.

Semua skill mengarahkan agent untuk mendiskusikan titik keputusan penting dengan operator sebelum melanjutkan, lalu merekomendasikan langkah berikutnya setelah tahap selesai. Skill bersifat netral provider; selain dimuat sebagai plugin di Codex dan Claude, isi SKILL.md dapat ditempel utuh sebagai instructions di host lain seperti ChatGPT yang terhubung ke remote MCP connector.

## Autentikasi

Server remote menggunakan sesi admin Zyx melalui OAuth atau connection token sementara. Connection token tidak boleh ditulis ke `.mcp.json`, README, source code, atau repository.

Plugin ini tidak menjalankan secara lokal:

- database atau Drizzle;
- Better Auth, session, atau authorization;
- storage atau R2;
- MCP HTTP/stdio server;
- Next.js route atau transport server.

Semua fungsi tersebut tetap menjadi tanggung jawab server remote di `zyx-web`.

## Workflow dan enforcement

MCP remote tetap menyediakan dua workflow:

- `idea_product` untuk Idea Bundle lalu Product Bundle;
- `quiz_bank` untuk draft bank soal.

MCP remote adalah sumber kebenaran untuk authentication, scope, schema, checksum, provenance, quality gate, dependency, dan staging. MCP tidak memiliki publication tool. Submission masuk ke review; publication dilakukan admin di Zyx.

## Instalasi

Gunakan repository ini sebagai local plugin source pada host yang sesuai. Untuk Codex, jalankan plugin validator sebelum memasang plugin. Untuk Claude Code, muat `.claude-plugin/plugin.json` sebagai local development plugin atau melalui marketplace plugin, lalu reload plugin setelah perubahan.

Repository URL yang tercantum di kedua manifest berasal dari remote Git yang sudah ada di checkout ini. Remote tersebut tidak dibuat, diubah, atau dipublikasikan oleh perubahan ini.

## Batas validasi

Validasi lokal repository ini mencakup manifest JSON, path skill dan MCP, endpoint staging, serta ketiadaan runtime import ke `lib/authoring` atau module internal `zyx-web`. Validasi OAuth, database, storage, server runtime, deployment, dan koneksi remote belum dilakukan oleh plugin repository ini.
