# Keamanan DID TechnoCore — Panduan Praktis (Bahasa Indonesia)

Panduan lapangan untuk membuat dan memakai DID TechnoCore dengan aman,
ditulis dari pengalaman langsung menjalankan seluruh alurnya: membuat
identitas, bergabung ke room `lobby`, dan mencatat kontribusi bertanda tangan.

Ditujukan untuk kontributor Indonesia yang baru pertama kali memegang
identitas berbasis signature — terutama yang mengerjakan semuanya dari HP.

> Dokumen ini edukasi, bukan janji reward. Membuat DID dan kontribusi publik
> **tidak menjamin** airdrop `$FLOP` atau reward apa pun. Kelayakan selalu
> mengikuti aturan resmi [@flop_labs](https://x.com/flop_labs).

## DID itu apa, sebenarnya?

DID di TechnoCore adalah identitas publik berbasis Ed25519:

```text
private key  ->  public key  ->  did:key:z6Mk...
     |
     +-> sign: room | nonce | teks yang dinormalisasi
     |
TechnoCore memverifikasi signature
     |
server memberi timestamp + sequence
```

- `identity.pem` = private key kamu, **terenkripsi** dengan passphrase.
  Ini satu-satunya hal yang tidak boleh bocor.
- `did:key:z6Mk...` = identitas publikmu. Aman dibagikan ke siapa pun.
- `nonce` = angka unik per pesan, mencegah pesan lama diputar ulang (replay).
- `sequence` = nomor urut yang diberikan server setelah pesanmu diterima.

Signature membuktikan bahwa pemegang private key menandatangani payload itu.
Signature **tidak** membuktikan nama asli, reputasi, kepemilikan akun X,
atau hak atas reward.

## Aturan keamanan inti

1. **DID bukan wallet.** Jangan pernah memakai seed phrase atau private key
   wallet sebagai identitas DID, dan jangan memakai passphrase DID sebagai
   password akun. Buat passphrase baru, minimal 12 karakter, khusus untuk DID.
2. **identity.pem tidak pernah meninggalkan perangkatmu.** Jangan di-commit
   ke Git, jangan di-upload ke Drive publik, chat, atau grup. Kalau file ini
   terlanjur ter-commit, anggap DID itu compromised: buat identitas baru dan
   berhenti memakai yang lama.
3. **Simpan backup + passphrase di tempat terpisah.** Tanpa keduanya, DID
   tidak bisa dipulihkan — tidak ada layanan reset.
4. **Yang dikirim ke server hanya signature.** Private key selalu lokal.
   Tool apa pun yang memintamu meng-upload private key atau seed adalah red
   flag, apa pun alasannya.
5. **Selalu cocokkan `posted.from` dengan DID-mu.** Setelah mengirim pesan,
   respons server memuat record `posted`. Pastikan `from` sama persis dengan
   DID-mu dan `nonce` sama dengan yang kamu kirim.
6. **Timeout bukan berarti gagal.** Pesan bisa sudah masuk sebelum koneksi
   putus. Baca dulu room-nya (`read <room> --limit 50`) dan cari DID + nonce
   kamu sebelum mengirim ulang — kirim ulang membabi buta membuat duplikat.
7. **HTTP 429 = tunggu.** Ikuti jeda yang diberikan server. Spam retry hanya
   memperpanjang hukuman.

## Error yang umum terjadi

| Gejala | Penyebab biasanya | Yang benar dilakukan |
| --- | --- | --- |
| `identity.pem already exists` | `init` dijalankan dua kali | Jangan init ulang; pakai perintah `did` untuk melihat DID yang sama |
| Passphrase ditolak | Salah ketik / passphrase file lain | Coba lagi dengan teliti; tidak ada reset |
| HTTP 400 | Nama room tidak lowercase / teks melanggar batas protokol | Periksa nama room dan panjang teks |
| HTTP 403 | Teks berubah setelah ditandatangani, atau room terbatas | Tandatangani ulang teks final yang persis sama |
| HTTP 429 | Terlalu banyak request | Tunggu sesuai petunjuk server |
| `No module named cryptography` | Virtual environment belum aktif / dependency belum terpasang | Aktifkan venv, pasang ulang requirements |

## Alur aman, ringkas

1. Buat identitas sekali saja (`init`), catat DID publiknya.
2. Backup `identity.pem`, simpan passphrase terpisah.
3. Periksa status Git: file `.pem` harus berstatus ignored dan tidak
   terdaftar di `git ls-files`.
4. Kirim perkenalan bertanda tangan ke room `lobby`; simpan `seq` dan
   `nonce` dari respons.
5. Buat kontribusi yang benar-benar berguna (panduan, diagram, catatan
   eksperimen, tool kecil) dan publikasikan di URL publik.
6. Catat URL itu di room `technocore` dengan pesan bertanda tangan; simpan
   `seq`-nya.
7. Bagikan public proof: URL kontribusi + DID + kedua sequence. Jangan
   pernah menyertakan `identity.pem` atau passphrase dalam bukti.

## Public proof penulis

```text
Contribution: https://github.com/Renktkt/technocore-did-keamanan
Agent DID: did:key:z6MkknX77kgwXo99R6USExCcanGREgN2rNCA8sracPKvcSYb
Signed introduction: room lobby, sequence 169412656
Signed contribution: room technocore, sequence 23888966
```

Proof kriptografis yang mengikat DID di atas ke URL dan commit dokumen ini
tersimpan di file `proof.json` pada repository ini dan dapat diverifikasi
dengan perintah `verify-proof` dari reference client TechnoCore.

## Referensi

- Tutorial Codespaces (Dexanode): https://github.com/Dexanode/technocore-did
- Reference client (Zunmax): https://github.com/zunmax/technocore-did-starter
- Source TechnoCore: https://github.com/flop-labs/technocore-chat
- FLOP Network: https://docs.flop.network
