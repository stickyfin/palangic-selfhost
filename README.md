# palangic-selfhost
Self hosted website for palang IC

# Palang IC (Self-Hosted)

A privacy-first, 100% browser-local MyKad cross-out and watermark web tool. All image processing—including HEIC conversion, interactive canvas watermarking, and A4 top/bottom layout generation—runs strictly inside the user's browser with zero server uploads.

---

## Features

- **100% Client-Side Privacy:** Images never leave your device or touch a backend server.
- **HEIC / JPG / PNG Support:** Automatically converts iPhone `.heic` / `.heif` photos locally via `heic2any`.
- **Interactive Watermark Controls:** Drag watermarks directly on the Front and Back MyKad canvases, or adjust **Text**, **Font Size (`px`)**, **Rotation (`°`)**, **Opacity**, and **Color** via sliders or exact number inputs.
- **URL Hash Pre-fill:** Open `#text=YOUR_PURPOSE` (e.g., `http://localhost:8085/#text=FOR%20LOAN%20APPLICATION%20ONLY`) to pre-populate the watermark text.
- **Multiple Export Formats:**
  - Individual Front PNG / Back PNG
  - Tight-cropped Combined PNG
  - **Live A4 Preview & Resize (`1240 × 1754 px` @ 150 DPI):** Places **Front IC on Top** and **Back IC on Bottom** with adjustable IC scale (`%`), vertical gap (`px`), and top offset (`px`) before exporting.

---

## Project Structure

```text
palang-ic/
├── index.html          # Complete single-file web application
├── docker-compose.yml  # Docker deployment config (NGINX Alpine)
└── README.md           # Deployment documentation
```

---

## Step 1: Prepare the Project Directory

Create a folder on your server or local machine and place your `index.html` inside it:

```bash
mkdir -p ~/docker/palang-ic
cd ~/docker/palang-ic
# Save your index.html file in this directory
```

---

## Step 2: Deploy Using Docker Compose (Recommended)

1. Create a `docker-compose.yml` file in the same directory:

```yaml
services:
  palang-ic:
    image: nginx:alpine
    container_name: palang-ic
    restart: unless-stopped
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html:ro
    ports:
      - "8085:80"
```

2. Start the container in detached mode:

```bash
docker compose up -d
```

3. Verify the container is running:

```bash
docker ps --filter "name=palang-ic"
```

4. Access the app in your browser at:
   - `http://localhost:8085` (local)
   - `http://<YOUR_SERVER_IP>:8085` (LAN)

---

## Step 3: Alternative Deployment Methods

### Option A: Single-Command Docker Run (No Compose File)

```bash
docker run -d \
  --name palang-ic \
  --restart unless-stopped \
  -p 8085:80 \
  -v $(pwd)/index.html:/usr/share/nginx/html/index.html:ro \
  nginx:alpine
```

### Option B: Standalone NGINX Host (Bare Metal / VM)

1. Copy `index.html` to your web root:

```bash
sudo mkdir -p /var/www/palang-ic
sudo cp index.html /var/www/palang-ic/index.html
```

2. Add a server block in `/etc/nginx/sites-available/palang-ic`:

```nginx
server {
    listen 80;
    server_name palang.yourdomain.com;

    root /var/www/palang-ic;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

3. Enable the site and reload NGINX:

```bash
sudo ln -s /etc/nginx/sites-available/palang-ic /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## Step 4: Reverse Proxy & HTTPS (Optional)

If exposing behind **NGINX Proxy Manager**, **Caddy**, or **Traefik**:

- **Scheme:** `http`
- **Forward Hostname / IP:** `palang-ic` (if on the same Docker network) or your host LAN IP
- **Forward Port:** `80` (Docker internal) or `8085` (host mapped port)
- **SSL:** Enable Force SSL / HTTPS (`heic2any` and Canvas APIs work over both HTTP and HTTPS).

### Security Headers (Recommended)

Because the app processes sensitive ID documents purely in browser memory, add a strict Content Security Policy (CSP) header in your reverse proxy to block unauthorized network requests:

```text
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' [https://cdn.jsdelivr.net](https://cdn.jsdelivr.net); style-src 'self' 'unsafe-inline'; img-src 'self' blob: data:; connect-src 'none';
```

> **Note:** Setting `connect-src 'none'` guarantees at the browser level that the page cannot transmit `fetch` or `XHR` payloads anywhere.

---

## Updating & Maintenance

- **Update the app:** Replace `index.html` on the host. Because it is bind-mounted read-only (`:ro`), changes take effect immediately upon refreshing your browser (`Ctrl + Shift + R`).
- **Update NGINX image:**
  ```bash
  docker compose pull && docker compose up -d
  ```
- **Stop and remove:**
  ```bash
  docker compose down
  ```
