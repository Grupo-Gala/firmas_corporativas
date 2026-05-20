# Proyecto: Generador de firmas HTML corporativas — Grupo GALA

Este fichero es contexto persistente para Claude Code. Al abrir el repositorio en VS Code, Claude lo carga automáticamente.

---

## Objetivo

Script Node.js que, a partir de un Google Sheet con los datos de los empleados y 4 plantillas HTML, genera la firma personalizada de cada empleado lista para que IT/RRHH se la envíe y el empleado la pegue en Gmail/Outlook.

## Decisiones cerradas (no re-discutir, ya validadas con el usuario)

- **Lenguaje del script:** Node.js.
- **Fuente de datos:** Google Sheet publicado como CSV (sin Google API ni credenciales).
- **Plantillas:** 4 HTMLs combinando marca × QR.
- **Recursos (assets):** este mismo repo, https://github.com/Grupo-Gala/firmas_corporativas, servido vía GitHub Pages en `grupo-gala.github.io/firmas_corporativas/`. Ya tiene `logos/`, `iconos/` y 16 QRs pre-generados en `qr/` con patrón `qr-{nombre}-{apellido}.png`.
- **Entrega:** copy-paste por el empleado en Gmail/Outlook (lo genera y envía IT/RRHH).
- **Estructura HTML de las plantillas:** las 4 plantillas comparten la estructura compacta tipo "autoescuela" (una sola tabla con `<tr><td colspan>` para el aviso legal, sin tabla externa envolvente, sin `<div>` separado para el aviso legal). `font-family` unificado a `Montserrat, Arial, sans-serif`. Aviso legal: el largo (incluye la frase sobre derechos de acceso, rectificación y supresión).

## Estado actual del repo

```
firmas_corporativas/
├── CLAUDE.md            ← este fichero
├── README.md
├── logos/               ← 3 logos
├── iconos/              ← 7 redes sociales 50px
├── plantillas/          ← 4 plantillas normalizadas y armonizadas con placeholders {{...}}
└── qr/                  ← 16 QRs ya generados
```

Aún no existen: `script/`, `salida/`.

Las 4 plantillas originales (pre-normalización) viven fuera del repo en la máquina del trabajo: `C:\Users\Usuario\Downloads\v{1,3}-Plantilla_Firma_Email_Corporativo-{Formacion,Autoescuela}-2026-{con,sin}-qr.html`. Ya no se necesitan; conservar como histórico si se quiere, no commitear.

## Modelo del Google Sheet (1 fila por empleado)

| Columna | Tipo | Notas |
|---|---|---|
| `nombre` | texto | Nombre y apellidos tal cual se mostrarán en la firma (capitalización normal, ej. `Sara Casillas`). |
| `cargo` | texto | Formato "Cargo · Departamento" |
| `telefono` | texto | Visible y usado en la firma |
| `email` | texto | Visible y usado en el `mailto:` |
| `direccion` | texto | Dirección postal (sede central si compartida, propia si no) |
| `maps_url` | texto | URL a Google Maps de esa dirección |
| `marca` | enum | `formacion` o `autoescuela` |

**No hay columna `qr_png`.** El nombre del fichero QR se deriva del campo `nombre` (slugificado: minúsculas, espacios → guiones, sin acentos, ej. `José Manuel Santiago` → `jose-manuel-santiago`). El script comprueba si `qr/qr-{slug}.png` existe en el repo. Si sí → plantilla con QR; si no → plantilla sin QR.

## Reglas de selección de plantilla

- `marca = formacion` + QR existe → `plantillas/formacion-con-qr.html`
- `marca = formacion` + QR no existe → `plantillas/formacion-sin-qr.html`
- `marca = autoescuela` + QR existe → `plantillas/autoescuela-con-qr.html`
- `marca = autoescuela` + QR no existe → `plantillas/autoescuela-sin-qr.html`

## Placeholders a usar al normalizar las plantillas

`{{nombre}}`, `{{cargo}}`, `{{telefono}}`, `{{email}}`, `{{direccion}}`, `{{maps_url}}`, `{{qr_filename}}`

(`{{qr_filename}}` sólo aparece en las plantillas "con-qr". Se sustituye por el nombre del fichero PNG, ej. `qr-sara-casillas.png`. La URL completa la monta la plantilla con `https://grupo-gala.github.io/firmas_corporativas/qr/{{qr_filename}}`.)

## Cambios obligatorios al normalizar las plantillas originales

1. **QR siempre por empleado en ambas marcas.** En Formación el QR original era fijo (`qr-gala-formacion.png`); ese fichero no existe y el caso se elimina — todas las firmas usan QR personal o ninguno.
2. **Mailto = `{{email}}`**, no genérico. La plantilla original de Formación apuntaba a `info@galaformacion.com`; cambiar.
3. **Dirección y maps_url siempre desde el Sheet** (`{{direccion}}`, `{{maps_url}}`). La plantilla de Formación traía la dirección a fuego; cambiar.
4. **Logo Autoescuela:** usar `64px-logo-gala-autoescuela.png` (no la `.jpg`). PNG soporta transparencia y se ve mejor en clientes con fondos no-blancos.

## Próximos pasos en orden

1. ~~**Normalizar las 4 plantillas**~~ ✅ Hecho. Las 4 viven en `plantillas/` con nombres limpios (`formacion-con-qr.html`, `formacion-sin-qr.html`, `autoescuela-con-qr.html`, `autoescuela-sin-qr.html`) y comparten estructura compacta (ver "Decisiones cerradas").
2. **Crear el Google Sheet** con el encabezado de la tabla de "Modelo del Google Sheet" (lo hace el usuario; pendiente para la próxima sesión). Publicarlo como CSV (Archivo → Compartir → Publicar en la web → CSV) y dejar la URL en una constante del script (o en un fichero `.env`).
3. **Escribir `script/generar-firmas.js`** (dentro del repo):
   - Descarga el CSV de la URL pública.
   - Para cada fila: slugifica `nombre`, comprueba si existe `qr/qr-{slug}.png` localmente en el repo, elige plantilla por `marca` + presencia de QR, sustituye placeholders, escribe `salida/{slug}.html`.
   - Dependencias mínimas (intentar `node` puro o un parser de CSV ligero como `csv-parse`).
   - Añadir `salida/` al `.gitignore` para no commitear firmas generadas.
4. **Actualizar `README.md`** con las instrucciones de uso para IT/RRHH: "para añadir un nuevo empleado: 1) añade fila al Sheet, 2) (si lleva QR) sube su PNG a `qr/` con el patrón `qr-{slug}.png` y haz commit, 3) `npm install && node script/generar-firmas.js`, 4) envía el `.html` resultante por chat al empleado".

**Punto de retomada (próxima sesión):** el usuario llega con el Google Sheet creado y publicado como CSV; arrancar por el paso 3 (escribir el script) usando esa URL.

## Cosas a tener en cuenta al colaborar

- Hablar **siempre en castellano** con el usuario.
- Prefiere **conversación en texto plano**, evitar menús de opciones múltiples salvo decisiones genuinamente binarias.
- Maneja **dos marcas**: GALA Formación (galaformacion.com) y Autoescuela GALA (autoescuelagala.com).
- Org GitHub: `Grupo-Gala` (mayúsculas en URL), pero github.io es lowercase `grupo-gala`.
