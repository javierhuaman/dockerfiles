# 🐘 PostgreSQL con Docker + Compose

Entorno de **PostgreSQL 16** dockerizado, parametrizable por **`.env`**, con persistencia de datos, healthcheck y comandos útiles para desarrollo.

---

## 🧱 Estructura sugerida

```
postgres/
├─ Dockerfile
├─ docker-compose.yml
├─ .env                 # variables (no subir credenciales reales)
└─ data/                # volumen persistente (se crea al levantar)
```

---

## ⚙️ Variables (.env)

Ejemplo de `.env`:

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin123
POSTGRES_DB=mi_base
PG_PORT=5432
TZ=America/Lima
```

> **Tip:** No subas credenciales reales al repo. Usa `.gitignore` y variables distintas por entorno.

---

## 🚀 Uso

```bash
# Construir la imagen (lee automáticamente el .env)
docker compose build

# Levantar en segundo plano
docker compose up -d

# Ver logs en tiempo real
docker compose logs -f

# Conectarse al contenedor (psql)
docker exec -it postgres16 psql -U $POSTGRES_USER -d $POSTGRES_DB
```

---

### 🔄 Parar / reiniciar

```bash
# Detener
docker compose down

# Detener y borrar volumen (¡eliminas datos!)
docker compose down -v

# Reiniciar con cambios
docker compose up -d --build
```

---

## 🧪 Verificación de estado (healthcheck)

El servicio incorpora `pg_isready`. Puedes comprobar:

```bash
docker inspect --format='{{json .State.Health}}' postgres16 | jq
```

O revisar los logs:

```bash
docker compose logs -f
```

---

## 📦 Persistencia de datos

El volumen `./data:/var/lib/postgresql/data` guarda la data fuera del contenedor.  
- **No** se borra al recrear el contenedor.  
- **Sí** se borra si haces `docker compose down -v`.

---

## 💾 Backups y restore rápidos

```bash
# Backup (dump) desde el host
docker exec -t postgres16 pg_dump -U $POSTGRES_USER -d $POSTGRES_DB > backup.sql

# Restore (¡borra y crea!)
cat backup.sql | docker exec -i postgres16 psql -U $POSTGRES_USER -d $POSTGRES_DB
```

> **Sugerencia:** Versiona tus scripts de migración en otra carpeta (ej. `/migrations`) y ejecútalos manualmente o con herramientas (Flyway/Liquibase).

---

## 🧰 Comandos útiles

```bash
# Entrar al shell del contenedor
docker exec -it postgres16 bash

# Ver bases disponibles
docker exec -it postgres16 psql -U $POSTGRES_USER -c "\l"

# Ver tablas de la DB por defecto
docker exec -it postgres16 psql -U $POSTGRES_USER -d $POSTGRES_DB -c "\dt"
```

---

## 🔒 Seguridad y buenas prácticas

- Cambia `POSTGRES_PASSWORD` (usa contraseñas fuertes o secretos).
- Limita el puerto publicado (usa redes internas si tu app está en el mismo Compose).
- No expongas `5432` públicamente en producción (usa firewall, VPN o reverse proxy con ACL).
- Haz backups periódicos y prueba el **restore**.

---

## 🏭 Consejos para producción

- Usa almacenamiento rápido (NVMe) para `./data`.
- Ajusta parámetros según carga (`shared_buffers`, `work_mem`, etc.) con un `postgresql.conf` custom.
- Monitorea con `pg_stat_statements` y métricas (Prometheus + Grafana).
- Define recursos (CPU/RAM) y `shm_size` adecuado (ya se incluye `1g`).

---

## ➕ (Opcional) Agregar pgAdmin 4

Si quieres administración visual, agrega al `docker-compose.yml`:

```yaml
  pgadmin:
    image: dpage/pgadmin4:8
    container_name: pgadmin4
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@local
      PGADMIN_DEFAULT_PASSWORD: admin123
      TZ: ${TZ}
    ports:
      - "8081:80"
    depends_on:
      - postgres
```

Accede en `http://localhost:8081` y conecta con:  
- **Host**: `postgres` (nombre del servicio)  
- **Port**: `5432`  
- **User/Pass**: los del `.env`  

---

## ❗️ Troubleshooting

- **“Connection refused”**: espera unos segundos; revisa `docker compose logs -f`.  
- **Error de permisos en `./data`**: elimina la carpeta y deja que Docker la cree, o corrige permisos (`sudo chown -R $USER:$USER data`).  
- **Cambiaste credenciales y no se reflejan**: recuerda que el init solo corre la **primera vez**; si ya hay datos, no se regeneran usuarios/DB. Usa `docker compose down -v` para reinicializar.  

---

✍️ **Autor:** Javier Huamán Huayllani
