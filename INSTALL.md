# Installation

This file shows how to install and configure Mollysocket **on your system using a systemd service** or **Docker**.

## Install the binary with a dedicated user

First of all, you need to install Mollysocket on your system.

#### Create a dedicated account

The service will run with a dedicated account, so create it and switch to that user:

```console
# useradd mollysocket -M
```

#### Install the binary

You have 2 solutions to install the binary.

1. Use an already compiled binary: <https://github.com/mollyim/mollysocket/releases/>. Download it to `/usr/local/bin/` and link the executable: `ln -s /usr/local/bin/{REPLACE_WITH_DOWNLOADED_MS} /usr/local/bin/ms`

2. Use cargo. This method allows you to use cargo to maintain mollysocket up to date. First of all, you need to [install cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html) (you need at least version 1.59). Then, install mollysocket using cargo: `cargo install mollysocket`. *You probably need to install some system packages, like libssl-dev libsqlite3-dev*. Then copy the compile binary to your system: `cp ~/.cargo/bin/mollysocket /usr/local/bin/ms`.

## Install systemd services

Download the 2 systemd unit files [mollysocket.service](https://github.com/mollyim/mollysocket/raw/main/mollysocket.service) and [mollysocket-vapid.service](https://github.com/mollyim/mollysocket/raw/main/mollysocket-vapid.service) and place them in the right direction `/etc/systemd/system/`.

### Start the service

You should be able to see that service now `systemctl status mollysocket`.

You can enable it `systemctl enable --now mollysocket`, the service is now active (`systemctl status mollysocket`), and will be started on system boot.

## App configuration

*If you host your own Push server*, then explicitly add it to the allowed endpoints. In `/etc/mollysocket/conf.toml`, edit `allowed_endpoints = ['*', 'https://push.mydomain.tld']` (remove `'*'` if you will use your push server only). Then restart the service `systemctl restart mollysocket`.

## Install via docker
1. If you do not have Docker installed already, [install it](https://docs.docker.com/engine/install/)
2. Create a new directory and place [docker-compose.yml](https://github.com/azzy6001-sketch/mollysocket/blob/azzy6001-sketch-patch-1/docker-compose.yml) in it. You can also use [docker-compose-caddy.yml](https://github.com/azzy6001-sketch/mollysocket/blob/azzy6001-sketch-patch-1/docker-compose-caddy.yml) if you would like to use the webserver mode and do not already have a reverse proxy.
3. Run `sudo docker compose run --rm mollysocket vapid gen` and paste the output into the `MOLLY_VAPID_PRIVKEY` environment variable
4. If using air gapped mode, add `MOLLY_WEBSERVER=false` to environment variables. Then, run `docker compose up` and wait for a qr code, scan it in the Molly app by going to settings, notifications, delivery service, Unified Push, then tap scan qr code. After this, run `docker compose down`
5. If using webserver mode with the docker-compose-caddy file, create a file named `Caddyfile` in the working directory and paste the Docker Caddyfile example shown in the Proxy Server section, make sure to change the domain to your domain
6. If using air gapped mode, run `sudo docker compose run --rm mollysocket (parameters)` To get your parameters, go to settings, notifications, configure unified push, and tap "copy connection parameters". Once this is done, you can run `docker compose up -d`, your setup should now be complete
7. If using webserver mode, run `docker compose up -d` and go to your domain, you should see a qr code. Scan it in the Molly app by going to settings, notifications, delivery service, Unified Push, then tap scan qr code, after this, your setup should be complete
   
**Note** *If you host your own Push server*, then explicitly add it to the allowed endpoints. In your `Docker-compose.yml` file, edit `MOLLY_ALLOWED_ENDPOINTS=["*"]` and add your domain, you can remove `*` if you plan to only use your push server. Remember to then run `docker compose up -d`

## (Option A) Proxy server

You will need to proxy everything from `/` to `http://127.0.0.1:8020/` (8020 is the value define in the systemd unit file for `$ROCKET_PORT`, it can be changed if needed).

You also need to forward the `Host` header.

If you proxy from another path like `/molly/` instead of `/`, you also need to pass the original URL.

For Nginx, it looks like:

```
    location / {
        proxy_pass http://127.0.0.1:8020/;
        proxy_set_header            Host $host;
        proxy_set_header X-Original-URL $uri;
    }
```

For Caddy, it looks like 

```
example.com # replace with your domain {
	reverse_proxy http://127.0.0.1:8020
}
```

If you're using the docker-compose-caddy.yml file, you can use this Caddyfile

```
example.com # replace with your domain {
	reverse_proxy http://mollysocket:8020
}
```

## (Option B) Air gapped mode

To find the MollySocket QR code:

- If you can use port-forwarding through SSH to your server, then run the following command: `ssh -L 8020:localhost:8020 your_server`, then open http://localhost:8020 on your machine. You can ignore alerts if there are any. Then click on _airgapped mode_.

- If you can't use port-forwarding, change `webserver` to `false` in your config file (_/etc/mollysocket/conf.toml_) and restart your service:

```console
# systemctl restart mollysocket
# journalctl -u mollysocket
# # This should show a QR code
```

After scanning the QR code, you will have a command to copy to run on your server. You must run this command as user `mollysocket` with `MOLLY_CONF=/etc/mollysocket/conf.toml`.

For instance `sudo -su mollysocket MOLLY_CONF=/etc/mollysocket/conf.toml /usr/local/bin/ms connection add baab32b9-d60b-4c39-9e14-15d8f6e1527e 2 thisisrandom 'https://push.mydomain.tld/upthisisrandom?up'`.

## (Optional) More restrictive configuration

Once you have registered Molly (with option A or B), and you will be the only user using this service, you can restrict `allowed_uuids = ['baab32b9-d60b-4c39-9e14-15d8f6e1527e']` and `allowed_endpoints = ['https://push.mydomain.tld/upthisisrandom?up']` in the config file.

## Backup the VAPID privkey

If you wish to backup your VAPID privkey, you can run the following:

```console
# systemd-creds decrypt /etc/mollysocket/vapid.key
```
