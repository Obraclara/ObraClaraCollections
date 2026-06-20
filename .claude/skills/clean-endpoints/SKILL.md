---
name: clean-endpoints
description: Limpia y normaliza los endpoints de la colección Bruno (ObraClaraAPI) para que sigan STANDARD.md. Detecta automáticamente con git status qué archivos .yml de endpoints estás tocando (staged, modificados o sin rastrear) y los corrige. Úsala cuando el usuario pida "limpiar endpoints", "normalizar", "que sigan mi estándar", o tras crear/editar endpoints en collections/.
---

# Clean Endpoints — ObraClaraAPI

Normaliza los endpoints Bruno que estás tocando ahora mismo para que cumplan `STANDARD.md`, sin escanear toda la colección.

## 1. Detectar el alcance (git status, automático)

Ejecuta y quédate solo con los `.yml` bajo `collections/`:

```bash
git status --porcelain -z collections/ | tr '\0' '\n'
```

Reglas de selección:
- Incluye archivos **staged, modificados y sin rastrear** (`A`, `M`, `??`, `AM`, `MM`).
- Considera solo rutas que terminen en `.yml`.
- Si la lista está vacía, dilo y pregunta si quiere que limpie una carpeta concreta en vez de salir en silencio.
- Los renombrados (`R`) usan el destino.

Antes de tocar nada, **muestra al usuario la lista de archivos detectados** que vas a limpiar. No expandas el alcance a archivos que git no reporta como cambiados, salvo que el usuario lo pida.

## 2. Cargar el estándar

Lee `STANDARD.md` (raíz del repo) como fuente de verdad. No inventes convenciones; si algo no está cubierto, déjalo igual y avísalo.

## 3. Limpiar cada archivo

Para cada `.yml` detectado, deriva el contexto de su ruta:
- **Carpeta raíz del recurso** = primer folder bajo `collections/ObraClaraAPI/` → define el **tag** (§10) y el nombre del recurso para variables/URLs.
- **Sub-carpeta** hereda el tag del padre.

Aplica las correcciones del estándar. Las más comunes:

| Qué revisar | Regla (sección) |
|---|---|
| Nombre de archivo y `info.name` coinciden, Title Case con espacios | §0, §5 |
| **Archivos basura**: `*copy*.yml`, duplicados, temporales → proponer borrar | §0 |
| `info.seq` en orden CRUD | §6 |
| `tags` = carpeta raíz del recurso | §10 |
| `auth: inherit` (o `none` solo en los públicos listados) | §7 |
| URL con `{{baseUrl}}/api/v1/...`, plural, trailing slash, path params `:snake_case` | §0 |
| Body principal solo `type: json` (**sin la clave `data`**, ni siquiera vacía); estructura va en `examples` | §8.3 |
| `examples` con request + response y status correcto | §8 |
| List: query params mínimos `ordering, page, page_size, search` (disabled) | §8.1 |
| Create: `script.post-response` guarda `<recurso>_id`; test status 201 + `id` | §8.3, §9 |
| `settings` constantes (encodeUrl, timeout 0, followRedirects, maxRedirects 5) | §11 |
| `docs` presente y no vacío (mínimo una línea) | §0 docs |
| `script.tests` con las assertions del método | §9.2 |

Edita los archivos in-place para corregirlos. Para **archivos basura** (ej. `List copy.yml`) no los reescribas: propón eliminarlos (con `git rm`/`rm` según estén staged o no) y confirma antes de borrar.

**Regla crítica del body (POST/PUT/PATCH):** el `http.body` del request principal debe tener **solo** `type: json`, nunca la clave `data` (ni vacía `data: ""` ni con contenido). El motivo es que Bruno reescribe ese `data` cada vez que se editan valores en la UI, generando diffs y commits basura. La estructura de campos vive únicamente en `examples` (request y response), que sí queda fija. Si encuentras un `data` colado en el request principal, **elimínalo** y, si la estructura no estaba ya en el example, muévela ahí. En GET y DELETE no debe existir bloque `body` en absoluto.

Si un endpoint creó/usa un recurso nuevo, recuerda la regla de agregar `<recurso>_id` a **todos** los environments (§2) — avísalo aunque no esté en el git status.

## 4. Reporte

Al terminar, resume por archivo: qué se corrigió, qué se dejó igual y por qué, y qué requiere decisión del usuario (borrados, variables de environment pendientes). No hagas commit salvo que lo pidan.
