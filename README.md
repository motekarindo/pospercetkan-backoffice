# sidigit

## Dockerized workflows

### Local development
- Copy `.env.example` to `.env` and adjust anything you need for development (defaults target PHP 8.4 on the official PHP FPM image).
- Use `make help` to discover common Docker targets.
- Start the stack with `make up` (or `docker compose -f docker-compose.local.yml up --build`).
- Install PHP dependencies inside the container: `make composer-install` (or `docker compose -f docker-compose.local.yml exec app composer install`).
- Run database migrations: `docker compose -f docker-compose.local.yml exec app php artisan migrate`.
- Handle frontend assets on the host machine: run `npm install` and `npm run dev` locally to keep Vite watching for changes.
- Your API is available on `http://localhost:8080` and the host-side Vite server runs on `http://localhost:5173`.
- Point the `AWS_*` values in `.env` at your existing MinIO/S3 endpoint (the compose file defaults to `http://host.docker.internal:9000`; on Linux set this to the accessible host or network alias).

### Production image & deployment
- Review `docker/.env.production` and replace the placeholder values with your real production secrets before building.
- Build the images with `docker compose -f docker-compose.prod.yml build`.
- Push them to your registry (for example `docker tag sidigit-app:latest registry.example.com/sidigit-app:latest`).
- Run the stack where you deploy: `docker compose -f docker-compose.prod.yml up -d`.
- After the containers start, run `docker compose -f docker-compose.prod.yml exec app php artisan key:generate --force` (first run), `php artisan migrate --force`, and `php artisan storage:link` if you need public storage.
- Make sure `docker/.env.production` references the correct external MinIO/S3 endpoint and credentials before deploying.

## Modul Struk Thermal
- Ditambahkan modul struk thermal untuk order pada route: `/orders/{order}/receipt` (`orders.receipt`).
- Akses route struk diproteksi permission `order.view` melalui mapping explicit di `config/route_permissions.php`.
- Struk bisa dibuka publik internal aplikasi (halaman preview) atau langsung cetak thermal via query `?print=1`.
- Opsi ukuran thermal didukung via query `paper`:
  - `paper=80` (default)
  - `paper=58`
- Contoh: `/orders/{order}/receipt?payment_id=10&paper=58&print=1`
- Jika order punya beberapa pembayaran, struk bisa dipilih per pembayaran melalui parameter `payment_id`.
- Integrasi tombol:
  - daftar order: aksi `Print Struk` (muncul jika order sudah punya pembayaran),
  - halaman order (edit/lihat): tombol `Print Struk`,
  - halaman input pembayaran: tombol `Print Struk Terakhir`.
- Layout struk didesain untuk kertas thermal 80mm di `resources/views/admin/orders/receipt.blade.php`.
