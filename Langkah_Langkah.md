# Langkah-langkah-Migrasi-Samba-Share-ke-HDD-Baru

````markdown
# Migrasi Samba Share ke HDD Baru di Debian

Dokumentasi ini berisi langkah-langkah memindahkan folder sharing Samba dari SSD kecil (128GB) ke HDD baru agar kapasitas lebih lega.  
Folder yang akan dipindahkan: `/srv/ctp`, `/srv/editing`, `/srv/karantina`, `/srv/render`.

---

## 1. Cek Harddisk Baru
Setelah memasang HDD baru, cek apakah sudah terbaca:
```bash
lsblk
````

Biasanya device baru akan muncul sebagai `/dev/sdb`.

---

## 2. Partisi dan Format HDD

Buat partisi baru:

```bash
sudo fdisk /dev/sdb
```

* Tekan `n` → `Enter` → pilih default
* Tekan `w` untuk simpan

Format ke **ext4**:

```bash
sudo mkfs.ext4 /dev/sdb1
```

---

## 3. Buat Mount Point

```bash
sudo mkdir /srv/data
```

---

## 4. Mount HDD Baru

Mount manual untuk tes:

```bash
sudo mount /dev/sdb1 /srv/data
```

Cek apakah sukses:

```bash
df -h | grep /srv/data
```

---

## 5. Pindahkan Data Lama ke HDD Baru

Gunakan `rsync` supaya permission ikut pindah:

```bash
sudo rsync -avh /srv/ctp /srv/data/
sudo rsync -avh /srv/editing /srv/data/
sudo rsync -avh /srv/karantina /srv/data/
sudo rsync -avh /srv/render /srv/data/
```

---

## 6. Backup Folder Lama & Buat Symlink

Backup folder lama:

```bash
sudo mv /srv/ctp /srv/ctp_backup
sudo mv /srv/editing /srv/editing_backup
sudo mv /srv/karantina /srv/karantina_backup
sudo mv /srv/render /srv/render_backup
```

Buat symlink agar tetap bisa diakses:

```bash
sudo ln -s /srv/data/ctp /srv/ctp
sudo ln -s /srv/data/editing /srv/editing
sudo ln -s /srv/data/karantina /srv/karantina
sudo ln -s /srv/data/render /srv/render
```

---

## 7. Tambahkan ke `/etc/fstab`

Cari UUID HDD:

```bash
sudo blkid /dev/sdb1
```

Edit `/etc/fstab`:

```fstab
UUID=abcd-1234-efgh-5678   /srv/data   ext4   defaults   0   2
```

Reload:

```bash
sudo mount -a
```

---

## 8. Update Konfigurasi Samba

Edit file konfigurasi Samba:

```bash
sudo nano /etc/samba/smb.conf
```

Sesuaikan path:

```ini
[ctp]
   path = /srv/data/ctp
   browseable = yes
   writable = yes
   valid users = @users

[editing]
   path = /srv/data/editing
   browseable = yes
   writable = yes
   valid users = @users

[karantina]
   path = /srv/data/karantina
   browseable = yes
   writable = yes
   valid users = @users

[render]
   path = /srv/data/render
   browseable = yes
   writable = yes
   valid users = @users
```

Restart Samba:

```bash
sudo systemctl restart smbd
```

---

## 9. Atur Permission

Pastikan semua folder share bisa diakses user Samba:

```bash
sudo chown -R root:users /srv/data/*
sudo chmod -R 2770 /srv/data/*
```

---

## 10. Uji Coba

Dari Windows client, coba akses:

```
\\192.168.10.250\ctp
\\192.168.10.250\editing
\\192.168.10.250\karantina
\\192.168.10.250\render
```

Cek log jika ada error:

```bash
tail -f /var/log/samba/log.smbd
```

---

✅ Sekarang semua data share Samba sudah dipindahkan ke HDD baru, sementara SSD tetap digunakan untuk sistem operasi.
