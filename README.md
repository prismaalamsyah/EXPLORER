Berikut adalah versi **Markdown** dari dokumentasi lengkap **Deploy Aplikasi Explorer ke VPS dengan Docker + Nginx + HTTPS**:

---

# 🚀 Deploy Aplikasi Explorer ke VPS dengan Docker, Nginx Reverse Proxy, dan HTTPS

**Domain**: [`https://explorer.camat.my.id`](https://explorer.camat.my.id)
**Platform**: Vue.js + Docker + Nginx + Let's Encrypt

---

## ✅ 1. Build dan Jalankan Aplikasi Explorer dengan Docker

### A. Build Docker Image

```bash
docker build -t explorer .
```

### B. Jalankan Docker Container di Port 8099

```bash
docker run -d -p 8099:80 --name explorer-container explorer
```

---

## ✅ 2. Pasang dan Konfigurasi Nginx

### A. Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

### B. Buat Config File untuk Domain

```bash
sudo nano /etc/nginx/sites-available/explorer.camat.my.id
```

**Isi file:**

```nginx
# 1. Redirect HTTP ke HTTPS + Challenge untuk Certbot
server {
    if ($host = explorer.camat.my.id) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    server_name explorer.camat.my.id;

    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# 2. HTTPS Reverse Proxy ke Aplikasi Vue di Docker
server {
    listen 443 ssl http2;
    server_name explorer.camat.my.id;

    ssl_certificate /etc/letsencrypt/live/explorer.camat.my.id/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/explorer.camat.my.id/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:8099;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        try_files $uri $uri/ /index.html;
    }
}

# 3. Redirect www ke non-www
server {
    listen 80;
    server_name www.explorer.camat.my.id;
    return 301 https://explorer.camat.my.id$request_uri;
}
```

### C. Aktifkan Konfigurasi

```bash
sudo ln -s /etc/nginx/sites-available/explorer.camat.my.id /etc/nginx/sites-enabled/
```

### D. Cek dan Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## ✅ 3. Pasang HTTPS dengan Certbot + Let's Encrypt

### A. Install Certbot & Plugin Nginx

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### B. Jalankan Certbot

```bash
sudo certbot --nginx -d explorer.camat.my.id
```

> Ikuti petunjuk: isi email, setuju TOS, dan pilih redirect to HTTPS jika ditanya.

---

## ✅ 4. Verifikasi HTTPS Berfungsi

### Akses Aplikasi:

```
https://explorer.camat.my.id/
```

### Tes Langsung Vue Router (SPA):

```
https://explorer.camat.my.id/empeiria/staking
```

> Jika route ini bisa diakses langsung (tanpa 404), berarti konfigurasi `try_files` sukses untuk history mode Vue.

---

## 🧹 5. Tambahan & Maintenance

### Cek Status Nginx

```bash
sudo systemctl status nginx
```

### Lihat Log Nginx (Debugging)

```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

### Tes Renewal Certbot Secara Manual

```bash
sudo certbot renew --dry-run
```

> Certbot biasanya otomatis memperpanjang sertifikat via cron.

---

## ✅ Summary

* ✅ Aplikasi Explorer **online di VPS** dengan Docker
* ✅ Domain `explorer.camat.my.id` aktif dan aman (HTTPS)
* ✅ Mendukung **SPA routing langsung** (Vue Router history mode)
* ✅ Reverse proxy lancar dengan **Nginx**

---

Jika kamu ingin membuat dokumentasi versi publik (misalnya untuk GitHub README), tinggal paste markdown ini langsung! Jika ingin menambahkan ilustrasi/diagram deployment, saya juga bisa bantu buatkan.
