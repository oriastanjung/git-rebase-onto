# Git Rebase --onto Sintaks

## Ringkasan

Perintah `git rebase --onto` memungkinkan Anda untuk me-rebase serangkaian commit dari satu branch ke branch lain, secara efektif memindahkan commit ke base yang berbeda.

### Sintaks

```bash
git rebase --onto <mau tempel kemana> <titk commit mulai terpisah> <branch yang di rebase>
```

**Parameter:**
- `<mau tempel kemana>`: Branch/commit target tempat commit akan di-rebase
- `<titk commit mulai terpisah>`: Commit tempat range rebase dimulai (eksklusif)
- `<branch-yang-di-rebase>`: Branch yang berisi commit yang akan di-rebase

## Skenario Kasus

**Keadaan Awal:**
```
staging: C1 ── C2 ── C3 ── C4 ── C5 ── C6
                    └─ C2.a ── C2.b ── C2.c  <branch feat/a>
```

## Proses Langkah demi Langkah

### 1. Buat Backup (Direkomendasikan)

Sebelum melakukan rebase, buat backup dari feature branch:

```bash
git switch feat/a
git branch backup/feat-a-pre-onto
git push origin backup/feat-a-pre-onto
```

### 2. Lakukan Rebase

Rebase commit dari `feat/a` (setelah C2) ke atas `staging` terbaru:

```bash
git rebase --onto origin/staging C2 origin/feat/a
```

**Pendekatan alternatif** (kurang fleksibel):
```bash
git rebase --onto <hash_C6> C2
```

### 3. Selesaikan Konflik (jika ada)

Jika terjadi konflik selama rebase:

1. **Selesaikan konflik** di editor Anda
2. **Stage file yang sudah diselesaikan:**
   ```bash
   git add <file-yang-selesai>
   ```
3. **Lanjutkan rebase:**
   ```bash
   git rebase --continue
   ```

**Untuk membatalkan rebase:**
```bash
git rebase --abort
```

### 4. Force Push Hasil

Push branch yang sudah di-rebase ke remote:

```bash
git push --force-with-lease origin HEAD:feat/a
```

## Hasil yang Diharapkan

**Setelah me-rebase `feat/a` ke atas `staging`:**
```
staging: C1 ── C2 ── C3 ── C4 ── C5 ── C6           (tidak berubah)
feat/a:  C1 ── C2 ── C3 ── C4 ── C5 ── C6 ── C2.a' ── C2.b' ── C2.c'
```

### 5. Fast-Forward Merge (Opsional)

Untuk memajukan `staging` agar mencakup perubahan `feat/a` tanpa merge commit:

```bash
git switch staging
git merge --ff-only feat/a
```

**Hasil akhir:**
```
staging: C1 ── C2 ── C3 ── C4 ── C5 ── C6 ── C2.a' ── C2.b' ── C2.c'
```

## Verifikasi

Untuk memvisualisasikan riwayat commit:

```bash
git log --oneline --graph --decorate --all --boundary
```

## Praktik Terbaik

1. **Selalu buat backup** sebelum melakukan operasi destruktif
2. **Gunakan `--force-with-lease`** daripada `--force` untuk force push yang lebih aman
3. **Test rebase** pada salinan lokal sebelum push ke remote
4. **Koordinasi dengan anggota tim** saat force-push ke branch yang dibagikan
5. **Gunakan pesan commit yang deskriptif** untuk membuat operasi rebase lebih jelas
