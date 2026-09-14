# 🐳 TYPO3 Docker - CMS Profesional Autohospedado Enterprise-Grade

[![GitHub Stars](https://img.shields.io/github/stars/martin-helmich/docker-typo3?style=social)](https://github.com/martin-helmich/docker-typo3)
[![Docker Pulls](https://img.shields.io/docker/pulls/martinhelmich/typo3)](https://hub.docker.com/r/martinhelmich/typo3)
[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.html)
[![TYPO3 Version](https://img.shields.io/badge/TYPO3-13.4%20LTS-orange)](https://typo3.org/download/)
[![PHP Version](https://img.shields.io/badge/PHP-8.1%2B-purple)](https://www.php.net/)

## 📋 Descripción general

**TYPO3 en Docker** es una solución completa para desplegar el CMS profesional enterprise-grade más potente del mercado en contenedores. Basado en la imagen oficial de **Martin Helmich** (276+ commits, 117k+ instalaciones activas mundiales), proporciona un sistema de gestión de contenidos robusto y flexible con arquitectura multi-usuario, permisos granulares por página, editor WYSIWYG integrado, menús dinámicos, media management, marketplace de extensiones ilimitado (TER), workflows staging/live, caché avanzado y soporte nativo para MySQL/MariaDB/PostgreSQL.

> 🎯 **Propuesta clave**: Enterprise CMS self-hosted sin licencias restrictivas, bajo tu control total, production-ready con 20+ años de historia enterprise.

## ✨ Características principales

- **Contenidos flexibles** — Estructura personalizable, custom fields, tipos de contenido ilimitados
- **Multi-usuario enterprise** — Usuarios, grupos, roles (admin, editor, viewer), permisos granulares por página
- **Editor WYSIWYG (RTE)** — Rich-text editor integrado con formatting, media, links, drag-drop
- **Menús dinámicos** — Generación automática desde page tree, responsive, jerárquicos
- **Media Manager** — Library de assets, subida imágenes/PDFs, organización carpetas, metadata
- **Extensiones ilimitadas** — TER marketplace (miles de extensiones), desarrollo custom, sin limitaciones
- **Staging/Live Workflow** — Publicación draft → staging → live, proceso aprobación, historial versiones
- **Backend robusto** — Database abstraction, multi-DB support (MySQL, PostgreSQL, MariaDB)
- **Caché avanzado** — PAGE cache, TYPO script cache, HTTP cache, performance tuned production-ready
- **Permisos granulares** — Access control per-page, user groups, roles, auditing, security
- **Multi-idioma (i18n)** — Soporte múltiples idiomas, traducciones, fallback, alcance global
- **Multi-site** — Single instalación, múltiples sites, compartir extensiones, escalable
- **Template system** — Fluid templating + TypoScript
- **Logging/Auditing** — Trazabilidad completa de acciones
- **Backup integration** — Compatible con estrategias de backup estándar

## 📋 Requisitos del sistema

- **Docker & Docker Compose v2+**
- **RAM**: 2 GB - 4 GB mínimo (PHP app ligera)
- **Disco**: 10 GB - 100+ GB (según contenidos + media)
- **Puertos**: TCP 80 (HTTP) o 443 (HTTPS via reverse proxy)
- **PHP**: 8.1+ (bundled en imagen Docker)
- **Base de datos**: MySQL 5.7+ / MariaDB 10.3+ / PostgreSQL 12+ (separado en compose)
- **Web Server**: Apache2 + mod_php (bundled en imagen)
- **Procesamiento imágenes**: GD Library, Imagick (incluidos)
- **Opcional**: Redis (session storage, caching)
- **Opcional**: Elasticsearch (indexed search)

> ⚠️ **Nota producción**: Imagen NO recomendada para producción sin hardening (SSL, security headers, etc). Usar reverse proxy nginx/Apache en frente con HTTPS.

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

```
# Abre en navegador:
http://localhost/typo3/install

# Sigue wizard de setup
```

### Acceso a TYPO3

| Componente | URL |
|------------|-----|
| 🌐 **Frontend TYPO3** (sitio web) | `http://localhost` |
| 🔧 **Backend TYPO3** (admin) | `http://localhost/typo3` |

## ⚙️ Configuración

1. **Variables de entorno DB** — Configura `TYPO3_DB_HOST`, `TYPO3_DB_USERNAME`, `TYPO3_DB_PASSWORD`, `TYPO3_DB_DATABASE`, `TYPO3_DB_DRIVER` (mysqli/pdo_pgsql)
2. **Credenciales MariaDB** — Define `MYSQL_ROOT_PASSWORD`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE`
3. **Charset DB** — `MYSQL_INITDB_ARGS=--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci`
4. **Healthcheck DB** — Verifica conectividad antes de iniciar TYPO3 (interval: 10s, timeout: 5s, retries: 5)
5. **Volúmenes persistentes** — `fileadmin` (media), `typo3conf` (config), `typo3temp` (cache), `db` (datos)
6. **Puerto expuesto** — `80:80` (cambiar a 8080:80 si hay conflicto)
7. **PostgreSQL (opcional)** — Cambia driver a `pdo_pgsql` y usa imagen `postgres:16` con variables `POSTGRES_*`

## 🚀 Primeros pasos

1. **Backend login** — Abre `http://localhost/typo3`, ingresa username + password del install tool, dashboard TYPO3 aparece
2. **Crear página** — Backend → Page tree (izquierda) → select "Home" o root page → click "+" → ingresa título → Type: Standard → Save
3. **Agregar contenido** — Page tree → selecciona página → Tab "Content" → click "Create content" → Type: Text, Text & Image, Headlines → escribe con editor WYSIWYG → Save
4. **Administrar usuarios** — Backend → System → Backend Users (o Web Users) → click "+" → ingresa datos (username, email, password) → asigna permissions (usergroup, página access) → Save
5. **Crear menús** — Frontend → páginas auto-generan menu desde page tree → Template → setup TypoScript para posición menu → menu dinámico renderiza automático
6. **Subir media** — Backend → File → Storage (fileadmin) → click upload, selecciona imágenes/PDFs → crea carpetas organiza assets → reference desde content editors
7. **Publicar contenido** — Content → Edit page → Public checkbox → Si workflow: approve required → Frontend muestra página viva

## 💡 Casos de uso

- **Sitios web corporativos** — Multi-página, contenido flexible, equipos de usuarios colaborativos
- **Portales empresariales** — Escalable, multi-usuario, permisos granulares, workflow de aprobación
- **Intranets** — Multi-site, multi-idioma, control de acceso, media management centralizado
- **Media publishers** — Workflow publicación staging/live, histórico, versioning, editorial process
- **Comunidad/foros (extensiones)** — TER marketplace plugins agregan funcionalidad social
- **E-commerce (extensiones)** — Integra shop extensions, payment gateways, catálogos productos

## 🔒 Acceso remoto seguro

Para acceso externo seguro, **usa siempre un reverse proxy** con HTTPS:

```yaml
# Ejemplo con nginx-proxy + letsencrypt (docker-compose override)
services:
  typo3:
    environment:
      - VIRTUAL_HOST=typo3.tudominio.com
      - LETSENCRYPT_HOST=typo3.tudominio.com
      - LETSENCRYPT_EMAIL=admin@tudominio.com
    expose:
      - "80"
    # Eliminar ports: - "80:80" cuando uses reverse proxy
```

**Nunca expongas el puerto 80/443 directamente a Internet** sin TLS termination y security headers.

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| **Ver estado** | `docker compose ps` |
| **Ver logs TYPO3** | `docker compose logs -f typo3` |
| **Ver logs DB** | `docker compose logs -f db` |
| **Detener** | `docker compose down` |
| **Actualizar versión** | Cambiar tag en compose → `docker compose pull` → `docker compose up -d` |
| **Backup DB** | `docker compose exec db mysqldump -u typo3 -ptypo3password123 typo3 > backup.sql` |
| **Restore DB** | `docker compose exec -T db mysql -u typo3 -ptypo3password123 typo3 < backup.sql` |
| **Shell TYPO3** | `docker compose exec typo3 bash` |
| **Monitorear recursos** | `docker stats typo3 db` |

**Consumo típico**:
- `typo3`: 200-400MB RAM
- `db`: 200-600MB RAM

### Marketplace TER (TYPO3 Extension Repository)

Instalables desde **Backend → Admin Tools → Extension Manager**:

| Extensión | Clave | Descripción |
|-----------|-------|-------------|
| **News** | `news` | Sistema news/blog con tags, categories. Must-have. |
| **Powermail** | `powermail` | Form builder, email integrations. Popular. |
| **Mask** | `mask` | Advanced content types builder. Powerful. |
| **Commerce** | `commerce` | E-commerce suite (payment, orders). |
| **SEO** | `seo` | SEO toolkit (sitemap.xml, robots.txt, metadata). |

## 📝 Licencia

**GPL-3.0 Open Source** — [Ver licencia completa](https://www.gnu.org/licenses/gpl-3.0.html)

TYPO3 es software libre: puedes redistribuirlo y/o modificarlo bajo los términos de la GNU General Public License versión 3.

---

> 📖 **Guía completa en el blog**: [Cómo instalar TYPO3 en Docker - CMS profesional autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-typo3-en-docker-cms.html)
>
> 🐳 **Imagen Docker oficial**: [martinhelmich/typo3](https://hub.docker.com/r/martinhelmich/typo3) | [GitHub](https://github.com/martin-helmich/docker-typo3)
>
> 📚 **Documentación oficial**: [TYPO3 Documentation](https://docs.typo3.org/)