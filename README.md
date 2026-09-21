# 🐳 TYPO3 Docker - CMS Profesional Autohospedado Enterprise-Grade

[![GitHub Stars](https://img.shields.io/github/stars/martin-helmich/docker-typo3?style=flat-square&logo=github)](https://github.com/martin-helmich/docker-typo3)
[![Docker Pulls](https://img.shields.io/docker/pulls/martinhelmich/typo3?style=flat-square&logo=docker)](https://hub.docker.com/r/martinhelmich/typo3)
[![License](https://img.shields.io/badge/License-GPL--3.0-blue?style=flat-square)](https://www.gnu.org/licenses/gpl-3.0.html)
[![TYPO3 Version](https://img.shields.io/badge/TYPO3-13.4%20LTS-orange?style=flat-square&logo=typo3)](https://typo3.org)

## 📋 Descripción general

**TYPO3 en Docker** es un CMS profesional autohospedado basado en PHP diseñado para empresas y portales grandes, que proporciona un sistema de gestión de contenidos robusto y flexible con editor WYSIWYG, multi-usuario con permisos granulares (por página, por usuario), menús dinámicos, media management, extensiones ilimitadas vía marketplace TER, staging/live workflows, backend poderoso, caché avanzado, soporte MySQL/MariaDB/PostgreSQL, todo ejecutándose en Docker bajo tu control sin licencias restrictivas.

La imagen Docker oficial es mantenida por **Martin Helmich** (martin-helmich/docker-typo3) con 117+ stars en GitHub, soporta versiones 6.2 → 13.4, y está lista para producción con 276+ commits recientes. TYPO3 cuenta con 117k+ instalaciones activas worldwide y 20+ años de historia enterprise.

## ✨ Características principales

- **Contenidos flexibles**: Estructura contenido personalizable, fields custom, tipos de contenido sin limitaciones
- **Multi-usuario**: Usuarios, grupos, roles con permisos granulares por página (editor, viewer, admin)
- **Editor WYSIWYG**: Rich-text editor integrado con formatting, media, links, drag-drop, user-friendly
- **Menús dinámicos**: Genera menús desde page tree automático, responsive, hierarchical, fácil
- **Media manager**: Library assets, subir imágenes/PDFs, organizar carpetas, metadata
- **Extensiones ilimitadas**: TER marketplace (miles), desarrollo custom, plugins, sin limitaciones
- **Staging/Live**: Workflow publishing, draft → staging → live, approval process, historia
- **Backend robusto**: Database abstraction, multi-DB support (MySQL, PostgreSQL, MariaDB), confiable
- **Caché avanzado**: PAGE cache, TYPO cache, HTTP cache, performance tuned, production-ready
- **Permisos granulares**: Access control per-page, user groups, roles, auditing, security
- **Multi-idioma**: i18n soporte, multiple languages, translations, fallback, global
- **Multi-site**: Single instalación, múltiples sites, compartir extensiones, escalable
- **REST API**: Unofficial REST API disponible
- **Logging/auditing**: Registro completo de actividades
- **Backup integration**: Herramientas de backup integradas

## 📋 Requisitos del sistema

- **Docker & Docker Compose v2+**
- **2 GB - 4 GB RAM** mínimo (PHP app ligera)
- **10 GB - 100+ GB** espacio disco (según contenidos + media)
- **Puerto TCP**: 80 (HTTP) o 443 (HTTPS reverse proxy)
- **PHP 8.1+** (bundled en imagen Docker)
- **MySQL 5.7+ / MariaDB 10.3+ / PostgreSQL 12+** (separado en compose)
- **Apache2 + mod_php** (bundled en imagen Docker)
- **GD Library, Imagick** (para procesamiento imágenes)
- **Opcional**: Redis (session storage, caching)
- **Opcional**: Elasticsearch (indexed search)
- ⚠️ **No para desarrollo solo**: Imagen NO recomendada producción sin hardening (SSL, security headers, etc). Usar reverse proxy nginx/Apache en frente con HTTPS

## 🐳 Instalación

### Paso 1: docker-compose.yml (TYPO3 13 LTS)

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

### Paso 3: Acceder install tool

```bash
# Abre en navegador:
http://localhost/typo3/install

# Sigue wizard de setup
```

### Acceder a TYPO3

| Acceso | URL |
|--------|-----|
| 🌐 **Frontend TYPO3** (sitio web) | `http://localhost` |
| 🔧 **Backend TYPO3** (admin) | `http://localhost/typo3` |

### Setup inicial (primer acceso)

1. Abre `http://localhost/typo3/install`
2. Install tool wizard → Database credentials (usa env variables)
3. Elige usuario admin (email + password)
4. Select TYPO3 distribution (Blank, Introduction Package, etc)
5. Finish → redirect Backend
6. Login con admin credentials
7. ¡Listo para gestionar contenidos!

💡 **Desde otros dispositivos**: Usa la IP de tu servidor:
- Frontend: `http://192.168.1.100`
- Backend: `http://192.168.1.100/typo3`
- Para obtener tu IP: `hostname -I`

## ⚙️ Configuración

1. **Variables de entorno principales** (en docker-compose.yml):
   - `TYPO3_DB_HOST`: Host de base de datos (db)
   - `TYPO3_DB_USERNAME`: Usuario DB (typo3)
   - `TYPO3_DB_PASSWORD`: Password DB
   - `TYPO3_DB_DATABASE`: Nombre base de datos (typo3)
   - `TYPO3_DB_DRIVER`: Driver (mysqli para MariaDB/MySQL, pgsql para PostgreSQL)

2. **Volúmenes persistentes**:
   - `typo3_fileadmin`: Archivos subidos (media, documentos)
   - `typo3_typo3conf`: Configuración local (LocalConfiguration.php, extensiones)
   - `typo3_typo3temp`: Archivos temporales, cache, assets procesados
   - `typo3_db`: Datos MariaDB/MySQL

3. **Base de datos**: Imagen permite MySQL, MariaDB, PostgreSQL. Composer incluye setup MariaDB standard. PostgreSQL requiere parámetro connection `TYPO3_DB_DRIVER=pgsql`

4. **Reverse Proxy (Producción)**: Usar nginx/Traefik/Caddy en frente con HTTPS, security headers, rate limiting

5. **Extensiones**: Instalables desde Backend → Admin Tools → Extension Manager

## 🚀 Primeros pasos

1. **Backend login**
   - Abre `http://localhost/typo3`
   - Username + password que configuraste en install
   - Dashboard TYPO3 aparece

2. **Crear página**
   - Backend → Page tree (izquierda) → select "Home" o root page
   - Click "+" para crear new page
   - Ingresa título página
   - Type: Standard (default)
   - Save → página creada

3. **Agregar contenido**
   - Page tree → selecciona página
   - Tab "Content" → click "Create content"
   - Type: Text, Text & Image, Headlines, etc
   - Escribe/pega contenido con editor WYSIWYG
   - Save

4. **Administrar usuarios**
   - Backend → System → Backend Users (o Web Users)
   - Click "+" para agregar usuario
   - Ingresa datos (username, email, password)
   - Asigna permissions (usergroup, página access)
   - Save → usuario puede loguear

5. **Crear menús**
   - Frontend → páginas auto-generan menu desde page tree
   - Template → setup TypoScript para posición menu
   - Menú dinámico renderiza automático

6. **Subir media**
   - Backend → File → Storage (fileadmin)
   - Click upload, selecciona imágenes/PDFs
   - Crea carpetas organiza assets
   - Reference desde content editors

7. **Publicar contenido**
   - Content → Edit page → Public checkbox
   - Si workflow: approve required
   - Frontend muestra página viva

## 💡 Casos de uso

- **Sitios web corporativos**: Multi-página, contenido flexible, usuario teams
- **Portales empresariales**: Escalable, multi-usuario, permisos granulares, workflow
- **Intranets**: Multi-site, multi-idioma, acceso control, media management
- **Media publishers**: Workflow publicación, staging/live, histórico, versioning
- **Comunidad/forums (extensiones)**: TER marketplace plugins agrega funcionalidad
- **E-commerce (extensiones)**: Integra shop extensions, payment gateways

## 🔒 Acceso remoto seguro

Para exponer TYPO3 de forma segura a Internet:

1. **Reverse Proxy obligatorio**: nginx, Traefik, Caddy con HTTPS (Let's Encrypt)
2. **Security headers**: HSTS, CSP, X-Frame-Options, Referrer-Policy
3. **Rate limiting**: Proteger login backend y install tool
4. **IP whitelist**: Restringir acceso a `/typo3/install` solo IPs de confianza
5. **Fail2ban**: Bloquear intentos de fuerza bruta
6. **VPN/Tailscale**: Acceso admin solo via VPN para máxima seguridad
7. **Authelia/Authelia**: SSO + 2FA delante del backend

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
# Cambiar imagen tag en compose (ej: martinhelmich/typo3:13 → :13.4)
docker compose pull
docker compose up -d
```

### Backup base datos
```bash
docker compose exec db mysqldump -u typo3 -ptypo3password123 typo3 > backup.sql
```

### Restore base datos
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

### Marketplace TER (TYPO3 Extension Repository)

TYPO3 soporta miles extensiones vía TER marketplace. Instalables desde backend.

**Extensiones populares:**
- **News (news)**: News/blog system con tags, categories. Must-have.
- **Powermail (powermail)**: Form builder, email integrations. Popular.
- **Mask (mask)**: Advanced content types builder. Powerful.
- **Commerce (commerc)**: E-commerce suite (payment, orders).
- **SEO (seo)**: SEO toolkit (sitemap.xml, robots.txt, metadata).

**Instalar extensión:**
Backend → Admin Tools → Extension Manager → search + install

## 📝 Licencia

**GPL-3.0 open source** - Software libre, puedes usar, modificar y distribuir bajo los términos de la licencia GPL-3.0.

---

> 📖 **Artículo original**: [Cómo instalar TYPO3 en Docker - CMS profesional autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-typo3-en-docker-cms.html)
> 
> 🐳 **Imagen Docker oficial**: [martinhelmich/typo3](https://hub.docker.com/r/martinhelmich/typo3) | [GitHub](https://github.com/martin-helmich/docker-typo3)
> 
> 📚 **Documentación oficial**: [TYPO3 Documentation](https://docs.typo3.org/)