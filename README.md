# Deploy-Joplin-with-docker-and-Nginx
This repository allows you to easily deploy a personal note server with [Joplin Server](https://joplinapp.org/help/apps/server/), using Docker and NGINX as a reverse proxy with HTTPS support thanks to Certbot.

---

## Installation steps

1. Clone this repository:

```bash
git clone https://github.com/jpavonabian/Deploy-Joplin-with-docker-and-Nginx
 cd Deploy-Joplin-with-docker-and-Nginx
````

2. Copy the file containing the variables and edit it:

````bash
cp example.env .env .env
nano .env
````

Fill in the SMTP, domain and database data according to your configuration.

3. Pull up the containers:

````bash
docker compose up -d
````

This will start Joplin Server and PostgreSQL on an internal Docker network.

4. Configure NGINX:

Copy the following file to `/etc/nginx/sites-available/notes.yourdomain.com` and link it to `sites-enabled`.

````nginx
server {
 listen 80;
 server_name notes.yourdomain.com;

    location / {
 proxy_pass http://127.0.0.1:22300;
 proxy_http_version 1.1;
 proxy_set_header Host $host;
 proxy_set_header X-Real-IP $remote_addr;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_set_header X-Forwarded-Proto $scheme;
 } }
}
`````

Enable the site and reload NGINX:

````bash
sudo ln -s /etc/nginx/sites-available/notes.yourdomain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
````

5. Generate the HTTPS certificate with Certbot:

````bash
sudo certbot --nginx -d notes.yourdomain.com
````

---

## Default access

Once deployed, access `https://notas.tudominio.es` with:

* **User**: `admin@localhost`
* **Password**: `admin`.

You can (should) change it from the web interface.

---

## Requirements

* Docker and Docker Compose
* NGINX installed on the machine
* Certbot (`sudo apt install certbot python3-certbot-nginx`)
* Domain pointing correctly to the server