# Infraestructura de Sincronización Segura con Syncthing y Docker

   ## Descripción
   Este proyecto contiene la infraestructura como código (IaC) para desplegar un nodo de Syncthing de manera automatizada, aislada y segura utilizando Docker Compose. Está diseñado para eliminar la dependencia de nubes de terceros y mantener el control total sobre la privacidad de los archivos.

   ## Arquitectura
   *   **Orquestación:** Docker Compose
   *   **Imagen base:** `linuxserver/syncthing` (Elegida por sus actualizaciones constantes y estándares de seguridad).

   ## Enfoque de Seguridad
   *   **Aislamiento:** El servicio se ejecuta en un entorno en contenedores, separando las dependencias de la aplicación del sistema host.
   *   **Gestión de Permisos:** Configurado para ejecutarse con un usuario sin privilegios (PUID/PGID específicos) para mitigar vectores de ataque y evitar la ejecución como `root`.
   *   **Protección de Secretos:** Las llaves criptográficas (`.pem`) y las bases de datos locales están estrictamente excluidas del control de versiones mediante `.gitignore`.
   *   **Cifrado:** Syncthing utiliza TLS nativo para asegurar la transferencia de datos punto a punto.

   ## Uso Rápido
   1. Clonar el repositorio.
   2. Ajustar el `PUID` y `PGID` en el archivo `compose.yaml` para coincidir con el usuario propietario de la máquina host.
   3. Ejecutar: `docker compose up -d`
