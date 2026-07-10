# 🔄 Infraestructura de Sincronización Privada y Segura con Syncthing y Docker

Este repositorio contiene la infraestructura como código (IaC) para desplegar un nodo de **Syncthing** de manera automatizada, portátil y segura utilizando **Docker Compose**. 

Está diseñado para usuarios que desean mantener el control absoluto sobre sus datos personales, eliminando la dependencia de servicios en la nube de terceros y facilitando la sincronización segura de archivos entre dispositivos (móviles, ordenadores y servidores). Personalmente lo uso para sincronizar mi teléfono con mi pc para almacenar y organizar mis fotos y archivos en mi disco duro, por lo que no necesito de servicios en la nube de Google Drive o DropBox que cuentan con un almacenamiento limitado y a su vez no son tan seguros y privados como Syncthing.

---

## 🏗️ Arquitectura del Servicio

*   **Orquestador:** Docker Compose.
*   **Imagen Base:** `lscr.io/linuxserver/syncthing:latest` (Mantenida activamente, optimizada para arquitectura multi-plataforma y con altos estándares de seguridad).
*   **Método de Sincronización:** Protocolo de Bloques Compartidos (BEP) peer-to-peer con cifrado de extremo a extremo.

---

## 🔒 Capa de Seguridad

Este despliegue ha sido diseñado siguiendo las mejores prácticas de seguridad informática para contenedores:

1.  **Ejecución sin Privilegios de Root:**
    El contenedor se ejecuta bajo un identificador de usuario (`PUID`) y grupo (`PGID`) específicos del sistema host. Esto previene que una eventual vulnerabilidad en Syncthing permita al atacante escalar privilegios y tomar el control del sistema host (`root`).
2.  **Protección del Panel de Administración (Web GUI):**
    Por defecto, el puerto `8384` está enlazado a la dirección loopback local (`127.0.0.1:8384`). Esto significa que la interfaz web de administración **solo** es accesible desde la propia máquina que aloja el contenedor.
    *   *Si necesitas habilitar el acceso remoto (ej. desde una red local), cambia el mapeo en `compose.yaml` a `8384:8384` e inmediatamente configura un usuario administrador con contraseña fuerte y activa la opción "Usar HTTPS para la GUI" en los ajustes de Syncthing.*
3.  **Aislamiento y Exclusión de Secretos:**
    Los archivos de identidad criptográfica generados automáticamente por Syncthing en el directorio `config/` (`key.pem`, `cert.pem`, `https-key.pem` y `https-cert.pem`) contienen claves privadas del dispositivo. Estos archivos y bases de datos locales están estrictamente excluidos en el archivo `.gitignore` para evitar filtraciones accidentales en repositorios públicos, al igual que las rutas absolutas de los directorios de sincronización, nombre de Host, zona horaria y los ID de procesos, almacenados en el archivo `.env` usadas como variables de entorno.
4.  **Cifrado P2P en Tránsito:**
    Toda la comunicación de datos y metadatos entre dispositivos utiliza TLS con cifrado fuerte para evitar ataques de intermediarios (Man-in-the-Middle) en redes públicas.

---

## 📁 Estructura del Proyecto

A continuación se detalla la función de cada archivo y cuáles no deben incluirse en el control de versiones:

```text
├── compose.yaml          # Definición de servicios e infraestructura Docker.
├── .env.example          # Plantilla de variables de entorno locales (Rutas, TZ, Hostname, etc).
├── .gitignore            # Filtro para excluir credenciales, base de datos y configuraciones locales.
├── README.md             # Documentación principal del repositorio.
└── config/               # [NO SUBIR A GITHUB] Carpeta generada en ejecución que contiene
                          # la base de datos de archivos y la identidad criptográfica (claves privadas).
```

---

## 🚀 Instalación y Uso Rápido

### Prerrequisitos
*   Tener instalado **Docker** y **Docker Compose**.
*   Conocer el `PUID` y `PGID` de tu usuario en la máquina host (se obtienen ejecutando el comando `id` en la terminal).

### Paso 1: Configurar las Variables de Entorno
Copia la plantilla `.env.example` para crear tu archivo `.env` personal (este archivo está ignorado en Git):

```bash
cp .env.example .env
```

Edita el archivo `.env` según tu sistema:
```ini
PUID=1000
PGID=1000
TZ=America/Bogota
SYNCTHING_HOSTNAME=mi-nodo-syncthing
SYNCTHING_DATA_DIR=/ruta/a/tus/datos/a/sincronizar
```

### Paso 2: Desplegar el Contenedor
Inicia el servicio en segundo plano:

```bash
docker compose up -d
```

### Paso 3: Acceder al Panel de Administración
Si mantuviste la configuración segura por defecto, accede desde el navegador local de la máquina host en:
👉 `http://127.0.0.1:8384`

---

## ⚙️ Puertos Utilizados

| Puerto | Protocolo | Tipo de Tráfico | Descripción |
| :--- | :--- | :--- | :--- |
| `8384` | TCP | HTTP / HTTPS | Interfaz Web de Administración (GUI). |
| `22000` | TCP | Datos (BEP) | Protocolo de Sincronización principal. |
| `22000` | UDP | Datos (BEP) | Protocolo de Sincronización rápido |
| `21027` | UDP | Descubrimiento | Descubrimiento local de dispositivos. |

---

## 💡 Mantenimiento recomendado
*   **Copias de Seguridad:** Respalda periódicamente el archivo `config/config.xml` y las claves `key.pem` y `cert.pem`. Si pierdes estas claves, tus dispositivos vinculados dejarán de reconocer este nodo y deberás reconfigurar los enlaces de confianza.
*   **Actualizaciones:** La imagen base se actualiza de manera continua. Puedes actualizar el contenedor ejecutando:
    ```bash
    docker compose pull
    docker compose up -d
    ```
