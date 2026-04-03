# Deploying KingdomTools on your VPS

This guide assumes:

- Your VPS runs **Linux** with **YunoHost** installed (which owns ports 80/443 via nginx)
- **Foundry VTT** runs separately via pm2
- You want to deploy at **kingdom.alpguler.dev**

---

## Prerequisites checklist

Before you start, make sure you have:

- [ ] SSH access to your VPS
- [ ] Your VPS IP address
- [ ] Access to your DNS provider (wherever `alpguler.dev` is managed)
- [ ] The repo pushed to GitHub/GitLab (so you can clone it on the VPS)

---

## Step 1 — Connect to your VPS

Open a terminal on your local machine and SSH into your VPS:

```bash
ssh your-username@YOUR_VPS_IP
```

> **Tip:** Replace `your-username` with your actual Linux username and `YOUR_VPS_IP`
> with your server's IP address (e.g. `ssh admin@203.0.113.50`).
>
> If you set up SSH keys, you won't need a password. Otherwise, type your password
> when prompted (it won't show characters as you type — that's normal).

From here on, **all commands are run on the VPS** unless stated otherwise.

---

## Step 2 — Point your domain to the VPS

Go to your DNS provider (Cloudflare, Namecheap, etc.) and add an **A record**:

| Type | Name    | Value          | TTL  |
|------|---------|----------------|------|
| A    | kingdom | `YOUR_VPS_IP`  | Auto |

This makes `kingdom.alpguler.dev` resolve to your server.

> **How to check if it works** (run this on the VPS or your local machine):
>
> ```bash
> ping kingdom.alpguler.dev
> ```
>
> You should see responses from your VPS IP. It can take 5-30 minutes to propagate.

---

## Step 3 — Register the domain in YunoHost

Tell YunoHost about the new subdomain so it manages the SSL certificate:

```bash
sudo yunohost domain add kingdom.alpguler.dev
```

> You'll be asked for your admin password. Type it and press Enter.

Then install the SSL certificate (free, via Let's Encrypt):

```bash
sudo yunohost domain cert install kingdom.alpguler.dev
```

> If this fails with a DNS error, wait a few more minutes for DNS to propagate and retry.

**Verify it worked:**

```bash
sudo yunohost domain list
```

You should see `kingdom.alpguler.dev` in the list.

---

## Step 4 — Install Docker

Check if Docker is already installed:

```bash
docker --version
```

If you see a version number (e.g. `Docker version 24.x`), skip to Step 5.

**If Docker is NOT installed:**

YunoHost runs on Debian, which doesn't have the Docker Compose plugin in its default
repos. You need to add Docker's official repository first:

```bash
# Update packages and install prerequisites
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the Docker repository to apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package list again (now includes Docker's repo)
sudo apt update

# Install Docker and the Compose plugin
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Make Docker start automatically on boot
sudo systemctl enable --now docker

# Let your user run Docker without sudo
sudo usermod -aG docker $(whoami)
```

> **Important:** After the last command, you need to log out and back in for the
> group change to take effect:
>
> ```bash
> exit
> ```
>
> Then SSH back in:
>
> ```bash
> ssh your-username@YOUR_VPS_IP
> ```

**Verify Docker works:**

```bash
docker run hello-world
```

You should see "Hello from Docker!" in the output.

**Verify Docker Compose works:**

```bash
docker compose version
```

You should see something like `Docker Compose version v2.x.x`.

---

## Step 5 — Clone the project

Create a directory and clone your repo:

```bash
# Create the project folder
sudo mkdir -p /opt/kingdomtools

# Make your user the owner (so you don't need sudo for everything)
sudo chown $(whoami):$(whoami) /opt/kingdomtools

# Go into it
cd /opt/kingdomtools

# Clone your repo (replace with your actual repo URL)
git clone https://github.com/prof958/kingdomTools.git .
```

> **Note:** The `.` at the end means "clone into the current directory" instead of
> creating a subfolder.
>
> If your repo is private, you'll need to authenticate. The easiest way is an
> HTTPS personal access token:
>
> ```
> https://YOUR_TOKEN@github.com/YOUR_USERNAME/kingdomTools.git
> ```
>
> Or set up an SSH key on the VPS:
>
> ```bash
> ssh-keygen -t ed25519 -C "your-email@example.com"
> cat ~/.ssh/id_ed25519.pub
> # Copy the output and add it to GitHub → Settings → SSH keys
> ```
>
> Then clone with SSH:
>
> ```
> git clone git@github.com:YOUR_USERNAME/kingdomTools.git .
> ```

---

## Step 6 — Create the `.env` file

This file holds your secrets. **Never commit it to git.**

### 6a — Generate the secret values

Run these commands one at a time and **write down / copy each result**:

```bash
# Generate a password for the database
openssl rand -base64 24
ZnzrUFVHg8oft0nvd5ilq3z8b9Q30qL8
```

Copy the output. This is your `POSTGRES_PASSWORD`.

```bash
# Generate a session secret
openssl rand -base64 32
Rtl2hSgAU57LD3xhRPJ5L8u/dPnmUuilHQxN8xRvob0=
```

Copy the output. This is your `SESSION_SECRET`.

### 6b — Generate the app login password hash

You need a bcrypt hash of the password you'll use to log into the app. You can generate
it using Node.js (which is already on your VPS since Foundry uses it):

```bash
node -e "const b=require('bcryptjs'); b.hash('YOUR_CHOSEN_PASSWORD', 10).then(h=>console.log(h))"
node -e "const b=require('bcryptjs'); b.hash('kingmaker', 10).then(h=>console.log(h))"

```

> **Replace `YOUR_CHOSEN_PASSWORD`** with the actual password you want to use to log in.
>
> If you get an error about bcryptjs not found, install it temporarily:
>
> ```bash
> npm install -g bcryptjs
> node -e "const b=require('bcryptjs'); b.hash('YOUR_CHOSEN_PASSWORD', 10).then(h=>console.log(h))"
> ```

Copy the output (it looks like `$2b$10$xxxxxxx...`). This is your `APP_PASSWORD_HASH`.
$2b$10$Hf852cCImcZvhchxEJF4/u3IvyuKlSrUC.zigC14OPvDBpk4hGSNK

### 6c — Create the file

```bash
cd /opt/kingdomtools
nano .env
```

> `nano` is a simple text editor. It opens inside your terminal.

Paste the following, replacing the placeholder values with what you generated above:

> **IMPORTANT:** Bcrypt hashes contain `$` signs (e.g. `$2b$10$Hf85...`).
> Docker Compose treats `$` as a variable reference. You **must double every `$`** in
> the hash to escape it. For example:
>
> - Original hash: `$2b$10$Hf852cCImcZvhchxEJF4/u3Ivy...`
> - Escaped for .env: `$$2b$$10$$Hf852cCImcZvhchxEJF4/u3Ivy...`
>
> If you see warnings like `The "Hf852c..." variable is not set`, it means the `$`
> signs were not escaped.

```env
# Database
POSTGRES_USER=kingdomtools
POSTGRES_PASSWORD=paste-your-generated-postgres-password-here

# App
SESSION_SECRET=paste-your-generated-session-secret-here
APP_PASSWORD_HASH=$$2b$$10$$paste-your-escaped-bcrypt-hash-here
```

> **How to use nano:**
>
> 1.  Type or paste the content (right-click to paste in most terminals)
> 2.  Press `Ctrl + O` (the letter O) to save → press `Enter` to confirm the filename
> 3.  Press `Ctrl + X` to exit

Lock down the file so only you can read it:

```bash
chmod 600 .env
```

**Verify the file looks correct:**

```bash
cat .env
```

> **Note:** You do NOT need a `DATABASE_URL` in the `.env` file — it is constructed
> automatically inside `docker-compose.yml` from `POSTGRES_USER` and `POSTGRES_PASSWORD`.

---

## Step 7 — Edit `docker-compose.yml`

Since YunoHost's nginx already handles HTTPS on ports 80/443, you need to **remove the
Caddy service** from the compose file. The app will only listen on `127.0.0.1:3000`
(localhost, not exposed to the internet).

```bash
cd /opt/kingdomtools
nano docker-compose.yml
```

**Delete everything** in the file (`Ctrl + K` repeatedly deletes one line at a time),
then paste this:

```yaml
services:
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: kingdomtools
      POSTGRES_USER: ${POSTGRES_USER:-kingdomtools}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "127.0.0.1:5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-kingdomtools}"]
      interval: 5s
      timeout: 5s
      retries: 5

  app:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER:-kingdomtools}:${POSTGRES_PASSWORD}@db:5432/kingdomtools
      APP_PASSWORD_HASH: ${APP_PASSWORD_HASH:?Set APP_PASSWORD_HASH in .env}
      SESSION_SECRET: ${SESSION_SECRET:?Set SESSION_SECRET in .env}
      NODE_ENV: production
    ports:
      - "127.0.0.1:3000:3000"

volumes:
  pgdata:
```

Save and exit (`Ctrl + O`, `Enter`, `Ctrl + X`).

> **What changed:** Removed the `caddy` service, its volumes (`caddy_data`,
> `caddy_config`), and the port bindings for 80/443. Everything else stays the same.

---

## Step 8 — Check for port conflicts

Before starting, make sure nothing else is using ports 3000 or 5432:

```bash
sudo ss -tlnp | grep -E '3000|5432'
```

- **If port 3000 is in use** (e.g. by Foundry): Change the app's port in
  `docker-compose.yml` from `"127.0.0.1:3000:3000"` to `"127.0.0.1:3001:3000"`.
  Then use port `3001` in the nginx config (Step 9).

- **If port 5432 is in use** (e.g. by another Postgres): Change the db port from
  `"127.0.0.1:5432:5432"` to `"127.0.0.1:5433:5432"`. The internal `DATABASE_URL`
  doesn't change — only the host-side mapping.

> **Tip:** Foundry VTT typically runs on port 30000, so port 3000 should be free.

---

## Step 9 — Configure YunoHost nginx to proxy to the app

You need to tell nginx to forward `kingdom.alpguler.dev` traffic to port 3000 (or 3001
if you changed it in Step 8).

```bash
# Create the config file
sudo nano /etc/nginx/conf.d/kingdom.alpguler.dev.d/kingdomtools.conf
```

> If you get an error that the directory doesn't exist:
>
> ```bash
> sudo mkdir -p /etc/nginx/conf.d/kingdom.alpguler.dev.d
> sudo nano /etc/nginx/conf.d/kingdom.alpguler.dev.d/kingdomtools.conf
> ```

Paste this content:

```nginx
location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # WebSocket support (future-proofing)
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    # Timeouts
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

> **If you changed the port to 3001 in Step 8**, change `proxy_pass` to
> `http://127.0.0.1:3001;`

Save and exit (`Ctrl + O`, `Enter`, `Ctrl + X`).

**Test the nginx config for syntax errors:**

```bash
sudo nginx -t
```

You should see:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**If there are errors**, re-open the file and check for typos.

**Reload nginx to apply:**

```bash
sudo systemctl reload nginx
```

---

## Step 10 — Build and start the app

```bash
cd /opt/kingdomtools

# Build and start everything (this will take a few minutes the first time)
docker compose up -d --build
```

> **What happens:**
>
> 1. Docker downloads the Postgres 16 image (~80 MB)
> 2. Docker builds your Next.js app from the Dockerfile (installs deps, builds, etc.)
> 3. Postgres starts and waits to be healthy
> 4. The app runs `prisma migrate deploy` (creates database tables)
> 5. The Next.js server starts on port 3000

**Watch the logs to see if everything is working:**

```bash
docker compose logs -f
```

> Press `Ctrl + C` to stop watching logs (the containers keep running).

**You should see:**

- Postgres: `database system is ready to accept connections`
- App: Prisma migration messages, then the server listening

**If you see errors**, check the troubleshooting section at the bottom.

---

## Step 11 — Verify the deployment

**Test locally on the VPS:**

```bash
curl -I http://127.0.0.1:3000
```

You should see `HTTP/1.1 200 OK` (or `307` redirect to login — both are fine).

**Test through nginx:**

```bash
curl -I https://kingdom.alpguler.dev
```

You should see `HTTP/2 200` or a redirect response.

**Finally, open in your browser:**

```
https://kingdom.alpguler.dev
```

You should see the KingdomTools login page. Log in with the password you chose in Step 6b.

---

## Step 12 — Seed the database (optional)

If you want to populate the database with initial data (PF2e items, etc.):

```bash
docker compose exec app npx prisma db seed
```

> If this fails, the seed script may need dev dependencies. As a workaround you can
> run the seed from your local machine through an SSH tunnel:
>
> ```bash
> # On your LOCAL machine (not the VPS), open a tunnel:
> ssh -L 5432:127.0.0.1:5432 your-username@YOUR_VPS_IP
>
> # Then in another terminal on your local machine:
> DATABASE_URL="postgresql://kingdomtools:YOUR_PG_PASSWORD@localhost:5432/kingdomtools" npx prisma db seed
> ```

---

## Everyday commands reference

Run these from `/opt/kingdomtools` on the VPS:

```bash
# Always start by going to the project directory
cd /opt/kingdomtools
```

### Updating the app after code changes

```bash
git pull
docker compose up -d --build
```

### Viewing logs

```bash
# All services
docker compose logs -f

# Only the app
docker compose logs -f app

# Only the database
docker compose logs -f db

# Last 100 lines only
docker compose logs --tail 100 app
```

### Restarting

```bash
# Restart just the app (keeps database running)
docker compose restart app

# Restart everything
docker compose restart
```

### Stopping

```bash
# Stop everything (data is preserved)
docker compose down

# Start again
docker compose up -d
```

### Database access

```bash
# Open a SQL shell
docker compose exec db psql -U kingdomtools

# Inside psql, useful commands:
#   \dt          — list tables
#   \q           — quit
#   SELECT * FROM "User" LIMIT 5;
```

### Running Prisma commands

```bash
docker compose exec app npx prisma migrate deploy
docker compose exec app npx prisma db seed
```

### Checking what's running

```bash
docker compose ps
```

### Checking disk usage

```bash
docker system df
```

---

## Troubleshooting

### "Port already in use"

```bash
sudo ss -tlnp | grep 3000
```

Find what's using the port and either stop it or change KingdomTools to a different port
(see Step 8).

### App container keeps restarting

```bash
docker compose logs app
```

Common causes:

- **Missing environment variables** → Check your `.env` file: `cat .env`
- **Database connection refused** → The db container might not be healthy yet. Wait and
  check: `docker compose ps`
- **Prisma migration error** → Something wrong with the database. Check db logs:
  `docker compose logs db`

### "502 Bad Gateway" in the browser

This means nginx can't reach the app. Check:

1. Is the app running? `docker compose ps` — the app should show `Up`
2. Is it on the right port? `curl http://127.0.0.1:3000` — should respond
3. Is the nginx config correct? `sudo cat /etc/nginx/conf.d/kingdom.alpguler.dev.d/kingdomtools.conf`

### SSL certificate issues

```bash
# Check certificate status
sudo yunohost domain cert status kingdom.alpguler.dev

# Force renewal
sudo yunohost domain cert install kingdom.alpguler.dev --force
```

### "Permission denied" errors

```bash
# If Docker commands fail without sudo
sudo usermod -aG docker $(whoami)
# Then log out and back in

# If file operations fail
ls -la /opt/kingdomtools/
# Check owner — should be your user
```

### Need to completely start over

```bash
cd /opt/kingdomtools

# Stop and remove containers + volumes (THIS DELETES THE DATABASE)
docker compose down -v

# Rebuild from scratch
docker compose up -d --build
```

### Check YunoHost hasn't overwritten your nginx config

After a YunoHost update:

```bash
cat /etc/nginx/conf.d/kingdom.alpguler.dev.d/kingdomtools.conf
```

If it's empty or missing, re-create it (see Step 9).
