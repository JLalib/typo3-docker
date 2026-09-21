# 🐳 TYPO3 Docker - CMS Profesional Autohospedado Enterprise-Grade

[![GitHub Stars](https://img.shields.io/github/stars/JLalib/typo3-docker?style=social)](https://github.com/JLalib/typo3-docker)
[![Docker Pulls](https://img.shields.io/docker/pulls/martinhelmich/typo3?label=Docker%20Pulls&logo=docker)](https://hub.docker.com/r/martinhelmich/typo3)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](https://opensource.org/licenses/GPL-3.0)
[![TYPO3 Version](https://img.shields.io/badge/TYPO3-13.4%20LTS-orange)](https://typo3.org/download/)
[![PHP Version](https://img.shields.io/badge/PHP-8.1%2B-purple)](https://www.php.net/)

---

## 📋 Descripción general

**TYPO3 en Docker** es una solución lista para producción que despliega el CMS enterprise-grade TYPO3 13 LTS (Long-Term Support) con MariaDB/PostgreSQL/MySQL en contenedores aislados. TYPO3 es un CMS profesional autohospedado basado en PHP diseñado para empresas y portales grandes, que proporciona un sistema de gestión de contenidos robusto y flexible con editor WYSIWYG, multi-usuario con permisos granulares (por página, por usuario), menús dinámicos, media management, extensiones ilimitadas vía marketplace TER, staging/live workflows, backend poderoso, caché avanzado y soporte multi-base de datos — todo ejecutándose en Docker bajo tu control sin licencias restrictivas.

Esta implementación utiliza la imagen oficial de Docker mantenida por **Martin Helmich** (`martinhelmich/typo3`), con 276+ commits recientes y soporte para versiones 6.2 → 13.4. Incluye configuración optimizada para producción con healthchecks, volúmenes persistentes y variables de entorno seguras.

> 📖 **Basado en:** [Cómo instalar TYPO3 en Docker - CMS profesional autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-typo3-en-docker-cms.html)

---

## ✨ Características principales

- **🏗️ Contenidos flexibles** — Estructura de contenido personalizable, fields custom, tipos de contenido sin limitaciones
- **👥 Multi-usuario enterprise** — Usuarios, grupos, roles con permisos granulares por página (editor, viewer, admin)
- **✏️ Editor WYSIWYG (RTE)** — Rich-text editor integrado con formatting, media, links, drag-drop, user-friendly
- **🌳 Menús dinámicos** — Generación automática desde page tree, responsive, jerárquicos, fáciles de mantener
- **📁 Media Manager** — Biblioteca de assets, subida de imágenes/PDFs, organización en carpetas, metadata
- **🔌 Extensiones ilimitadas** — TER marketplace (miles de extensiones), desarrollo custom, plugins, sin limitaciones
- **🔄 Staging/Live Workflow** — Publicación draft → staging → live, proceso de aprobación, historial de versiones
- **⚙️ Backend robusto** — Database abstraction, multi-DB support (MySQL, PostgreSQL, MariaDB), confiable
- **⚡ Caché avanzado** — PAGE cache, TYPO script cache, HTTP cache, performance tuned, production-ready
- **🔐 Permisos granulares** — Access control per-page, user groups, roles, auditoría, seguridad
- **🌍 Multi-idioma (i18n)** — Soporte múltiples idiomas, traducciones, fallback, alcance global
- **🏢 Multi-site** — Single instalación, múltiples sitios, compartir extensiones, escalable
- **📦 Versiones LTS Docker** — 13.4 (latest LTS, PHP 8.1+), 12.4 (previous LTS), legacy disponibles
- **🐳 Docker oficial** — Imagen `martinhelmich/typo3` con 117k+ instalaciones activas worldwide, 20+ años historia enterprise

---

## 📋 Requisitos del sistema

- **Docker & Docker Compose v2+** instalados
- **2 GB - 4 GB RAM** mínimo (PHP app ligera)
- **10 GB - 100+ GB** espacio disco (según contenidos + media)
- **Puerto TCP: 80** (HTTP) o **443** (HTTPS via reverse proxy)
- **PHP 8.1+** (bundled en imagen Docker)
- **MySQL 5.7+ / MariaDB 10.3+ / PostgreSQL 12+** (separado en compose)
- **Apache2 + mod_php** (bundled en imagen Docker)
- **GD Library, Imagick** (para procesamiento imágenes) — incluidos
- **Opcional: Redis** (session storage, caching)
- **Opcional: Elasticsearch** (indexed search)
- ⚠️ **No para desarrollo solo:** Imagen NO recomendada producción sin hardening (SSL, security headers, etc). Usar reverse proxy nginx/Apache en frente con HTTPS

---

## 🐳 Instalación

### Paso 1: Crear `docker-compose.yml` (TYPO3 13 LTS)

```yaml
version: '3.8'

services:
  typo3:
    image: martinhelmich/typo3:13
    container_name: typo3
    restart: unless-stopped
    environment:
      - TYPO3_DB_HOST=db
      - TYPO3_DB_USERNAME=typo3
      - TYPO3_DB_PASSWORD=typo3password123
      - TYPO3_DB_DATABASE=typo3
      - TYPO3_DB_DRIVER=mysqli
    ports:
      - "80:80"
    volumes:
      - typo3_fileadmin:/var/www/html/fileadmin
      - typo3_typo3conf:/var/www/html/typo3conf
      - typo3_typo3temp:/var/www/html/typo3temp
    depends_on:
      db:
        condition: service_healthy

  db:
    image: mariadb:11
    container_name: typo3-db
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword123
      - MYSQL_USER=typo3
      - MYSQL_PASSWORD=typo3password123
      - MYSQL_DATABASE=typo3
      - MYSQL_INITDB_ARGS=--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - typo3_db:/var/lib/mysql

volumes:
  typo3_fileadmin:
  typo3_typo3conf:
  typo3_typo3temp:
  typo3_db:
```

### Paso 2: Iniciar TYPO3

```bash
# Guardar como docker-compose.yml en directorio proyecto
docker compose up -d

# Espera ~10 segundos para que DB inicie
docker compose logs -f typo3

# Cuando veas "Apache 2.x is running" está listo
```

### Paso 3: Acceder al Install Tool

```
# Abre en navegador:
http://localhost/typo3/install

# Sigue wizard de setup
```

### Acceso a TYPO3

| Interfaz | URL |
|----------|-----|
| 🌐 **Frontend TYPO3** (sitio web) | `http://localhost` |
| 🔧 **Backend TYPO3** (admin) | `http://localhost/typo3` |

---

## ⚙️ Configuración

1. **Variables de entorno de base de datos** — Configura `TYPO3_DB_HOST`, `TYPO3_DB_USERNAME`, `TYPO3_DB_PASSWORD`, `TYPO3_DB_DATABASE`, `TYPO3_DB_DRIVER` (mysqli/pdo_pgsql)
2. **Contraseñas seguras** — Cambia `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD` por valores fuertes en producción
3. **Volúmenes persistentes** — `fileadmin` (media), `typo3conf` (configuración), `typo3temp` (cache), `db` (datos MariaDB)
4. **Healthcheck de BD** — MariaDB incluye healthcheck nativo para asegurar disponibilidad antes de iniciar TYPO3
5. **Driver de base de datos** — `mysqli` para MariaDB/MySQL, `pdo_pgsql` para PostgreSQL (cambiar imagen DB y driver)
6. **Puerto expuesto** — `80:80` por defecto; para HTTPS usar reverse proxy (nginx/Traefik) en puerto 443
7. **Character set** — `utf8mb4` con collation `utf8mb4_unicode_ci` configurado en `MYSQL_INITDB_ARGS`

---

## 🚀 Primeros pasos

1. **Backend login** — Abre `http://localhost/typo3`, ingresa username + password configurados en install tool, dashboard TYPO3 aparece
2. **Crear página** — Backend → Page tree (izquierda) → select "Home" o root page → Click "+" para crear new page → Ingresa título → Type: Standard (default) → Save → página creada
3. **Agregar contenido** — Page tree → selecciona página → Tab "Content" → click "Create content" → Type: Text, Text & Image, Headlines, etc → Escribe/pega contenido con editor WYSIWYG → Save
4. **Administrar usuarios** — Backend → System → Backend Users (o Web Users) → Click "+" para agregar usuario → Ingresa datos (username, email, password) → Asigna permissions (usergroup, página access) → Save → usuario puede loguear
5. **Crear menús** — Frontend → páginas auto-generan menú desde page tree → Template → setup TypoScript para posición menú → Menú dinámico renderiza automático
6. **Subir media** — Backend → File → Storage (fileadmin) → Click upload, selecciona imágenes/PDFs → Crea carpetas organiza assets → Reference desde content editors
7. **Publicar contenido** — Content → Edit page → Public checkbox → Si workflow: approve required → Frontend muestra página viva

---

## 💡 Casos de uso

- **🏢 Sitios web corporativos** — Multi-página, contenido flexible, equipos de usuarios colaborativos
- **🏛️ Portales empresariales** — Escalable, multi-usuario, permisos granulares, workflow de aprobación
- **🔐 Intranets** — Multi-site, multi-idioma, control de acceso, media management centralizado
- **📰 Media publishers** — Workflow publicación, staging/live, histórico, versioning de contenido
- **👥 Comunidad/foros (extensiones)** — TER marketplace plugins agregan funcionalidad social
- **🛒 E-commerce (extensiones)** — Integra shop extensions, payment gateways, catálogos productos

---

## 🔒 Acceso remoto seguro

> **⚠️ IMPORTANTE:** Para exposición a internet, **nunca** expongas el puerto 80 directamente. Usa siempre un reverse proxy con HTTPS.

### Opción A: Nginx Proxy Manager (recomendado para principiantes)
```yaml
# Añade a tu docker-compose.yml existente
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: npm
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - npm_data:/data
      - npm_letsencrypt:/etc/letsencrypt
    depends_on:
      - typo3
```
Accede a `http://tu-ip:81` (admin@example.com / changeme) → Add Proxy Host → `typo3:80` → Enable SSL → Let's Encrypt

### Opción B: Traefik (automático, para avanzados)
```yaml
  traefik:
    image: traefik:v3.0
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.email=tu@email.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "letsencrypt:/letsencrypt"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.typo3.rule=Host(`typo3.tudominio.com`)"
      - "traefik.http.routers.typo3.entrypoints=websecure"
      - "traefik.http.routers.typo3.tls.certresolver=letsencrypt"
      - "traefik.http.services.typo3.loadbalancer.server.port=80"
```
Añade labels al servicio `typo3` y despliega.

---

## 🛠️ Gestión y mantenimiento

### Ver estado
```bash
docker compose ps
```

### Ver logs
```bash
docker compose logs -f typo3
docker compose logs -f db
```

### Detener TYPO3
```bash
docker compose down
```

### Actualizar versión
```bash
# Cambiar imagen tag en compose (ej: martinhelmich/typo3:13 → :12 o :latest)
docker compose pull
docker compose up -d
```

### Backup base de datos
```bash
docker compose exec db mysqldump -u typo3 -ptypo3password123 typo3 > backup.sql
```

### Restore base de datos
```bash
docker compose exec -T db mysql -u typo3 -ptypo3password123 typo3 < backup.sql
```

### Acceder shell TYPO3
```bash
docker compose exec typo3 bash
```

### Monitorear consumo
```bash
docker stats typo3 db
# Típicamente:
# typo3: 200-400MB RAM
# db: 200-600MB RAM
```

---

## 📦 Marketplace TER (TYPO3 Extension Repository)

TYPO3 soporta miles de extensiones vía TER marketplace. Instalables desde backend.

| Extensión | Clave | Descripción |
|-----------|-------|-------------|
| **News** | `news` | News/blog system con tags, categories. Must-have. |
| **Powermail** | `powermail` | Form builder, email integrations. Popular. |
| **Mask** | `mask` | Advanced content types builder. Powerful. |
| **Commerce** | `commerce` | E-commerce suite (payment, orders). |
| **SEO** | `seo` | SEO toolkit (sitemap.xml, robots.txt, metadata). |

**Instalar extensión:** Backend → Admin Tools → Extension Manager → search + install

---

## 📊 Stack técnico

| Componente | Tecnología |
|------------|------------|
| **Backend** | PHP 8.1+ (bundled en imagen) |
| **Web Server** | Apache2 + mod_php (bundled) |
| **Database** | MySQL 5.7+ / MariaDB 10.3+ / PostgreSQL 12+ |
| **ORM** | Doctrine DBAL (database abstraction) |
| **Frontend** | Fluid templating (TypoScript) |
| **Caching** | PAGE cache, TYPO cache, HTTP cache |
| **Media Processing** | GD, Imagick para imágenes |
| **Licencia** | GPL-3.0 open source |

---

## ⚖️ Comparativa con alternativas

| vs | TYPO3 gana en | Alternativa gana en |
|----|---------------|---------------------|
| **WordPress** | Enterprise architecture, permisos granulares, estructura flexible, multi-site nativo | Simplicidad, vasto ecosistema plugins, curva aprendizaje fácil |
| **Drupal** | Mejor WYSIWYG, mejor UI/UX, backend más amigable | Personalización más potente, comunidad más grande |
| **Joomla** | Mejor performance, features enterprise | Más user-friendly para principiantes |

**Mejor para:** Empresas medianas/grandes, multi-usuario workflows, portales complejos, control fino, self-hosted.

---

## 📚 Referencias oficiales

- [TYPO3 Official Website](https://typo3.org/)
- [Docker TYPO3 GitHub - martinhelmich/typo3](https://github.com/martin-helmich/docker-typo3)
- [TYPO3 Documentation - Complete guide](https://docs.typo3.org/)
- [Reverse Proxy Container Configuration](https://docs.typo3.org/m/typo3/docs-typo3cms/master/en-us/Installation/Containers/Index.html)
- [TER - TYPO3 Extension Repository](https://extensions.typo3.org/)
- [Docker Hub - martinhelmich/typo3](https://hub.docker.com/r/martinhelmich/typo3)
- [TYPO3 Community Forum & Support](https://typo3.org/community/)

---

## 📝 Licencia

Este proyecto de configuración Docker se distribuye bajo licencia **MIT**. TYPO3 CMS es software libre bajo licencia **GPL-3.0-or-later**.

```
TYPO3 CMS - Copyright (C) 1998-2024 TYPO3 Association
Docker Image - Copyright (C) Martin Helmich
This Docker Compose configuration - MIT License
```

---

> 📌 **Post original:** [Cómo instalar TYPO3 en Docker - CMS profesional autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-typo3-en-docker-cms.html)  
> 🐙 **Repo:** [JLalib/typo3-docker](https://github.com/JLalib/typo3-docker)  
> ☕ **Apoya el canal:** [Ko-fi](https://ko-fi.com/genbyte) | [YouTube](https://youtube.com/@genbyte) | [Newsletter](https://genbyte.blogspot.com/newsletter)