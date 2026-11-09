# Portal IESTP Federico Uranga

Archivos para publicar en Hostinger:

- `portal-private/`: código PHP, vistas y datos. Va **fuera** de `public_html`, al mismo nivel.
- `public_html/`: raíz pública del dominio, con la página, la intranet, el CSS, el JavaScript y las imágenes.

El archivo `portal-private/config.php` (datos de la base de datos y clave de instalación) **no se guarda en GitHub**: el repositorio es público. Se crea a partir de `portal-private/config.example.php`.

El código es el del sistema v2 de `ranyuo/v2_flv` (commit `35fef57`, 9 de noviembre de 2026). Se conservaron el contenido inicial, el tema, el logo y las imágenes del IESTP Federico Uranga, y sus correcciones propias: textos sin datos pendientes, teléfono oculto cuando está vacío, concordancia de género en reclamos y quejas, y ventana emergente con imagen completa.

## Requisitos del hosting

- **PHP 8.1 o superior** con las extensiones `pdo_mysql`, `fileinfo`, `openssl`, `zlib`, `json` y `session` (Hostinger las trae activas). `mbstring` es recomendable.
- **MariaDB 10.4+ o MySQL 8** (en hPanel, «MySQL» es MariaDB). **Una base de datos por sitio:** las tablas usan el prefijo fijo `pisco_`.
- El sitio debe estar en la **raíz de un dominio o subdominio** (no en una subcarpeta).
- Carpetas con permiso de escritura: `portal-private/` (bloqueo de instalación, `backups/` y la configuración SMTP guardada desde la intranet) y `public_html/assets/cache/`.

## Instalación nueva

1. **Base de datos:** en hPanel › *Bases de datos › Bases de datos MySQL*, crea una base y un usuario con su contraseña. Anota los tres datos.
2. **Descarga el código:** en GitHub, *Code › Download ZIP*.
3. **Sube y extrae:** con el *Administrador de archivos*, sube el ZIP a la carpeta del dominio (la que contiene `public_html`) y extráelo. GitHub mete todo en una carpeta `fu-main/`; mueve `portal-private/` y `public_html/` desde ahí a la carpeta del dominio y reemplaza el `public_html` vacío.
4. **Configura:** copia `portal-private/config.example.php` como `portal-private/config.php` y completa:
   - `db_name`, `db_user` y `db_password`;
   - `install_key`: una clave privada aleatoria de 32 caracteres o más. Guárdala.
5. **Instala:** abre `https://tu-dominio/install.php`, escribe la `install_key` y crea la cuenta del administrador principal.
6. **Borra** `public_html/install.php`. Después de instalar ya no sirve: si lo vuelves a abrir responde «El portal ya está instalado» (código 410). Mientras el portal no esté instalado, la portada muestra un aviso que indica abrir `/install.php`.
7. **Permisos:** `public_html/assets/cache/` debe tener permisos **755**.
8. **Comprueba:**
   - Abre el sitio y entra a la intranet con la cuenta creada.
   - Revisa *Configuración*: datos institucionales, logo, distrito y región.
   - En *Mesa de partes › Áreas receptoras › Editar áreas*, carga las áreas reales con sus correos.
   - En *Usuarios y roles*, crea las cuentas del personal.

**Correo de notificaciones (recomendado):** en la intranet, *Configuración › Configurar correo notificador (SMTP)*, indica el servidor, la cuenta y su **contraseña de aplicación** (no la clave personal) y activa el envío. *Usar Gmail / Google Workspace* completa el servidor, el puerto y el cifrado. Después, en hPanel › *Avanzado › Trabajos cron*, crea una tarea **cada minuto** con `php /home/<usuario>/domains/<dominio>/portal-private/mail-worker.php` para los reintentos. En *Configuración › Estado de las notificaciones por correo* se comprueba que los avisos pasen a *Enviado*. Como alternativa, el bloque `smtp` de `config.php` (ver `config.example.php`) acepta la misma configuración; lo guardado desde la intranet tiene prioridad.

**Opcional, respaldos automáticos:** en hPanel › *Avanzado › Trabajos cron*, crea una tarea diaria con `php /home/<usuario>/domains/<dominio>/portal-private/cron.php`. Luego actívala en la intranet, en *Copias de seguridad*.

**Retención de la auditoría:** el historial de auditoría se conserva 24 meses y lo más antiguo se elimina una vez al día, desde `mail-worker.php` o `cron.php` (basta con que una de las dos tareas esté programada). Para cambiar el plazo, define `'audit_retention_months'` en `config.php` (mínimo 6; `0` lo conserva sin límite).

## Actualizar una instalación existente

Sube `portal-private/` y `public_html/` encima de las carpetas actuales y acepta **sobrescribir**. **No borres las carpetas antes:** así se conservan `config.php`, `backups/`, `install.lock` y la configuración SMTP guardada (`.smtp-settings.json`). Después, borra `public_html/install.php`.

La base de datos se actualiza sola la primera vez que se abre el sitio con la versión nueva: crea la cola de correo `pisco_mail` y da al Responsable académico permiso de edición en Configuración académica y a Dirección permiso de consulta. No se pierden datos ni cuentas.

## Qué trae el sistema v2

- **Diseños del portal:** en *Temas y apariencia › Diseño del portal* se elige entre **Clásico institucional** (el actual), **Moderno**, **Universitario**, **Editorial**, **Impacto**, **Impacto institucional**, **Horizonte** y **Campus**. Los colores de cada diseño se ajustan en *Temas y apariencia › Colores del diseño*; si no se cambian, siguen el tema activo (azul y dorado de Federico Uranga).
- **Programas de estudio en carrusel** (portada y página de programas). Para cambiar a cuadrícula: *Vista en la web* en *Programas de estudio*.
- **Noticias con galería:** imagen principal y hasta 4 fotos adicionales, con visor ampliado.
- **Notificaciones por correo (SMTP)** de trámites, reclamos, Bolsa laboral, derivaciones y cambios de estado, con cola de reintentos.
- **Configuración académica general:** programas, módulos, unidades didácticas, asignaciones docentes, ambientes, turnos y periodo; certificación modular, grados y títulos, constancias y certificados con registro de verificación.
- **Enlaces propios de los servicios:** `/mesa-de-partes.php`, `/libro-de-reclamaciones.php`, `/bolsa-laboral.php` y `/certificados.php` abren cada servicio sin el menú del portal, para enlazarlos desde otras webs.
- **Mesa de partes:** código y dígitos del correlativo configurables (`MPV-IESTPFU`), y feriados nacionales del Perú (incluida Semana Santa) calculados solos cada año.
- **Pruebas automáticas** en `tests/` y en GitHub Actions (`.github/workflows/qa.yml`). Ver `tests/README.md`.

## Dónde se edita

- **CSS y JavaScript del portal:** `public_html/assets/app/` (`portal.css` y `portal-1.js` a `portal-4.js`).
- **Armazón de la página:** `portal-private/views/portal.php`.
- **Contenido inicial** que carga la instalación: `portal-private/data/portal.json`.

## Imágenes y documentos

Los archivos públicos (logo, carrusel, fotos, PDF) van en `public_html/assets/fu/`. El logo provisional es `public_html/assets/fu/logo.png`; reemplázalo por el oficial con el mismo nombre o súbelo desde *Configuración*.

## Estado del contenido

Cargado con la información del Oficio N.º 105-2026-IESTP"FU"-MFGQ/D: presentación, reseña histórica, misión y visión, organigrama, director, docentes de Asistencia Administrativa, programas de estudio (con plan de estudios de Mecánica Agrícola y módulo 1 de Asistencia Administrativa), admisión, matrícula, horario 2026-II, fotos de la nueva infraestructura, logo y banner.

Se completó con **información estándar** (revisar y reemplazar cuando haya datos oficiales): valores institucionales, perfil y campo laboral de Industrias de Alimentos y Bebidas, campo laboral de los otros programas, requisitos de admisión, servicios, mensaje de la Dirección, beneficios de la portada, hitos institucionales (tomados de la reseña), Formulario Único de Trámite (`assets/fu/FUT-formulario-unico-de-tramite.pdf`) y una silueta genérica (`assets/fu/avatar-personal.png`) como foto del personal.

Pendiente: RUC, teléfono, redes sociales, dominio, responsables de las jefaturas, fotos del personal, TUPA, documentos de gestión (PEI, PAT, RI), vacantes y cronograma de admisión 2027.
