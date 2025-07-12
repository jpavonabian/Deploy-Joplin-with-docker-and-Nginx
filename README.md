# Deploy-Joplin-with-docker-and-Nginx
Este repositorio te permite desplegar fácilmente un servidor de notas personal con [Joplin Server](https://joplinapp.org/help/apps/server/), usando Docker y NGINX como proxy inverso con soporte HTTPS gracias a Certbot.

---

## Pasos de instalación

1. Clona este repositorio:

```bash
git clone https://github.com/jpavonabian/Deploy-Joplin-with-docker-and-Nginx
cd Deploy-Joplin-with-docker-and-Nginx
````

2. Copia el archivo que contiene las variables y edítalo:

```bash
cp example.env .env
nano .env
```

Rellena los datos SMTP, dominio y base de datos según tu configuración.

3. Levanta los contenedores:

```bash
docker compose up -d
```

Esto iniciará Joplin Server y PostgreSQL en una red interna de Docker.

4. Configura NGINX:

Copia el siguiente archivo en `/etc/nginx/sites-available/notas.tudominio.es` y enlázalo en `sites-enabled`.

```nginx
server {
    listen 80;
    server_name notas.tudominio.es;

    location / {
        proxy_pass http://127.0.0.1:22300;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Habilita el sitio y recarga NGINX:

```bash
sudo ln -s /etc/nginx/sites-available/notas.tudominio.es /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

5. Genera el certificado HTTPS con Certbot:

```bash
sudo certbot --nginx -d notas.tudominio.es
```

---

## Acceso por defecto

Una vez desplegado, accede a `https://notas.tudominio.es` con:

* **Usuario**: `admin@localhost`
* **Contraseña**: `admin`

Puedes (debes) cambiarla desde la interfaz web.

---

## Requisitos

* Docker y Docker Compose
* NGINX instalado en la máquina
* Certbot (`sudo apt install certbot python3-certbot-nginx`)
* Dominio apuntando correctamente al servidor
