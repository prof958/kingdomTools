# Bypassing YunoHost SSO for kingdom.alpguler.dev

YunoHost's SSO (Single Sign-On) intercepts all requests to domains it manages
and redirects to its login portal. You need to disable this for
`kingdom.alpguler.dev` so the app handles its own authentication.

---

## Method 1 — Edit ssowat config (quick)

### Step 1: Check current config

```bash
sudo cat /etc/ssowat/conf.json.persistent
```

### Step 2: Edit the file

```bash
sudo nano /etc/ssowat/conf.json.persistent
```

If the file is empty or just `{}`, replace everything with:

```json
{
    "unprotected_urls": [
        "kingdom.alpguler.dev/"
    ]
}
```

If it already has content, add `"kingdom.alpguler.dev/"` to the `unprotected_urls`
array. Example with existing entries:

```json
{
    "unprotected_urls": [
        "some-other-app.alpguler.dev/",
        "kingdom.alpguler.dev/"
    ]
}
```

Save: `Ctrl + O`, `Enter`, `Ctrl + X`.

### Step 3: Reload nginx

```bash
sudo systemctl reload nginx
```

### Step 4: Test

```bash
curl -I https://kingdom.alpguler.dev
```

If it still redirects to SSO, try Method 2.

---

## Method 2 — Set domain permission via YunoHost CLI

```bash
# List current permissions
sudo yunohost user permission list

# Allow visitors (unauthenticated users) on the domain
sudo yunohost domain config set kingdom.alpguler.dev -a default_app=""
```

Then reload:

```bash
sudo systemctl reload nginx
```

---

## Method 3 — Configure via nginx directly

If Methods 1 and 2 don't work, it's possible YunoHost's main nginx server block
for `kingdom.alpguler.dev` includes SSO Lua scripts that run before your
`kingdomtools.conf` location block.

### Step 3a: Check the main server block

```bash
sudo cat /etc/nginx/conf.d/kingdom.alpguler.dev.conf
```

Look for lines like:

```nginx
include conf.d/yunohost_sso*.conf;
```

or

```nginx
access_by_lua_file /usr/share/ssowat/access.lua;
```

### Step 3b: Override SSO in your location block

Edit your proxy config:

```bash
sudo nano /etc/nginx/conf.d/kingdom.alpguler.dev.d/kingdomtools.conf
```

Add the `access_by_lua` override at the top of the location block:

```nginx
location / {
    # Disable YunoHost SSO for this location
    access_by_lua_file /dev/null;

    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

If `access_by_lua_file /dev/null;` causes an error, try:

```nginx
    access_by_lua '';
```

or:

```nginx
    access_by_lua_block {
        return
    }
```

### Step 3c: Test and reload

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## Method 4 — Remove and re-add domain without SSO

As a last resort, remove the domain from YunoHost and re-add it without SSO:

```bash
# Remove the domain (keeps DNS and cert)
sudo yunohost domain remove kingdom.alpguler.dev

# Re-add it
sudo yunohost domain add kingdom.alpguler.dev
```

After re-adding, YunoHost creates a fresh server block. Then:

1. Re-create your nginx proxy config (see DEPLOY.md Step 9)
2. Add the domain to `unprotected_urls` (Method 1 above) BEFORE reloading nginx
3. Reload: `sudo systemctl reload nginx`

---

## Debugging

### Check what SSO is doing

```bash
# See the SSO configuration YunoHost generated
sudo cat /etc/ssowat/conf.json | python3 -m json.tool | grep -A5 kingdom
```

### Check nginx config for the domain

```bash
# Main server block
sudo cat /etc/nginx/conf.d/kingdom.alpguler.dev.conf

# Your custom config
sudo cat /etc/nginx/conf.d/kingdom.alpguler.dev.d/kingdomtools.conf

# All configs for this domain
ls -la /etc/nginx/conf.d/kingdom.alpguler.dev*
```

### Check if the redirect is from nginx or the app

```bash
# -L follows redirects, -v shows headers
curl -v https://kingdom.alpguler.dev 2>&1 | head -30
```

If you see `Location: https://kingdom.alpguler.dev/yunohost/sso/` in the response
headers, the redirect is from nginx/SSO. If you see a 200 response, SSO has been
bypassed.

### Regenerate SSO config

After editing `conf.json.persistent`, force YunoHost to regenerate:

```bash
sudo yunohost app ssowatconf
sudo systemctl reload nginx
```
