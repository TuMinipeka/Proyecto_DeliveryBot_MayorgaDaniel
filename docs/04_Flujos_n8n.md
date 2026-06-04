# 04 — Flujos n8n

## Módulo 01 — Menú y Navegación

**Archivo:** `Workflow/modulos/01_Menu_Navegacion.json`  
**Responsabilidad:** Entrada al sistema, autenticación/registro de usuarios, presentación de categorías.

### Nodos

| Nodo | Tipo | Descripción |
|------|------|-------------|
| `Telegram Disparador` | telegramTrigger | Webhook que escucha `message` y `callback_query` |
| `Ajuste Variables` | Set | Normaliza `chat_id`, `nombre_completo`, `texto_usuario` de ambos tipos de update |
| `Leer estado Usuario` | googleSheets (get) | Busca en SESSION por `telegram_id`. `returnFirstMatch: true`, `alwaysOutputData: true` |
| `If1` | IF | `texto_usuario == "/start"` |
| `If` | IF | `Leer estado Usuario.telegram_id` is not empty (sesión existente) |
| `Reset Session` | googleSheets (update) | `pantalla_actual=VER_CATEGORIAS`, `carrito_temporal={}`, matching por `telegram_id` |
| `Registro Usuario` | googleSheets (append) | Inserta en USUARIOS: `telegram_id`, `nombre_completo`, `puntos_lealtad=0` |
| `Crear session` | googleSheets (append) | Inserta en SESSION: `telegram_id`, `pantalla_actual=VER_CATEGORIAS`, `carrito_temporal={}`, `ultimo_cambio=$now` |
| `Categorias` | Telegram | Mensaje con inline keyboard: Bebidas, Postres, Comidas, Almuerzos |
| `Error Enviar /start` | Telegram | Mensaje: *"Para iniciar tu pedido, por favor envía el comando /start 🚀"* |
| `Router De Estado` | Switch | 5 ramas por `pantalla_actual ?? 'NUEVO'` |

### Decisiones técnicas clave

- `alwaysOutputData: true` en **Leer estado Usuario** garantiza que el flujo continúe aunque el usuario no exista en SESSION.
- `returnFirstMatch: true` evita que múltiples filas del mismo usuario rompan el flujo.
- `texto_usuario = $json.message?.text ?? $json.callback_query?.data` soporta mensajes de texto y callbacks de botones inline.

---

## Módulo 02 — Carrito y Pedidos

**Archivo:** `Workflow/modulos/02_Carrito_Pedidos.json`  
**Responsabilidad:** Navegación de productos, selección, acumulación en carrito, confirmación.

### Nodos

| Nodo | Tipo | Descripción |
|------|------|-------------|
| `Leer MENU` | googleSheets (get) | Filtra MENU por `categoria == texto_usuario`. `alwaysOutputData: true` |
| `Construir lista De Productos` | Code | Construye texto del menú + `mapa_productos` (JSON indexado 1..N) |
| `Fusionar Carrito` | Code | Combina `mapa_productos` nuevo con `items` existentes del carrito |
| `Mostrar Comidas` | Telegram | Envía el texto del menú en formato Markdown |
| `Guardar Estado (Carrito)` | googleSheets (update) | `pantalla_actual=CARRITO`, `carrito_temporal=carrito_fusionado` |
| `Procesar Selección Comida` | Code | Valida número escrito, extrae producto del mapa, construye `nuevo_carrito_temporal` |
| `Validar Seleccion` | IF | `$json.valido is true` |
| `Guardar Siguiente Rama` | googleSheets (update) | `pantalla_actual=SELECCIONANDO_CANTIDAD`, `carrito_temporal=nuevo_carrito_temporal` |
| `Comida seleccionada + Cantidad` | Telegram | *"Seleccionaste X, ¿Cuántas unidades?"* |
| `Error Número Inválido` | Telegram | Mensaje con el error del nodo anterior |
| `Procesar Cantidad` | Code | Valida entero positivo, agrega item al array, calcula subtotal y total, construye resumen |
| `Validar cantidad` | IF | `$json.valido is true` |
| `Error Cantidad Inválida` | Telegram | Mensaje con error |
| `Guardar Estado CONFIRMAR` | googleSheets (update) | `pantalla_actual=CONFIRMAR_PEDIDO`, `carrito_temporal=nuevo_carrito_temporal` |
| `Resumen Carrito` | Telegram | Resumen + 3 botones: ✅ Confirmar / ➕ Seguir comprando / ❌ Cancelar pedido |

### Decisiones técnicas clave

- **Fusión del carrito:** `carrito_temporal` en estado CARRITO contiene el mapa de productos (keys numéricas) y el array `items` de categorías anteriores. `Fusionar Carrito` hace un spread del mapa nuevo preservando los `items` existentes.

---

## Módulo 03 — Gestor de Estados

**Archivo:** `Workflow/modulos/03_Gestor_Estados.json`  
**Responsabilidad:** Procesamiento de confirmación, validación de stock, registro del pedido, descuento de inventario, cancelación.

### Nodos

| Nodo | Tipo | Descripción |
|------|------|-------------|
| `Switch` | Switch | 3 ramas: `CONFIRMAR`, `SEGUIR`, `CANCELAR` por valor de `texto_usuario` |
| `Lectura stock Actual` | googleSheets (getAll) | Lee todas las filas de MENU sin filtro |
| `Confirmar Stock + Generar Pedido` | Code | Lee carrito de SESSION, valida stock por item, genera `id_pedido = PED-{chatId}-{timestamp}`, calcula fecha/hora Colombia (UTC-5) |
| `Validar Stock` | IF | `$json.valido is true` |
| `Registrar Pedidos` | googleSheets (append) | Inserta en PEDIDOS con todos los campos del pedido |
| `Stock Descuento` | Code | Mapea cada item a `{id_producto, stock_nuevo, row_number}` |
| `Descontar los stoks` | googleSheets (update) | Actualiza MENU por `id_producto` con nuevo stock |
| `Limpiar Carrito SESSION` | googleSheets (update) | `pantalla_actual=VER_CATEGORIAS`, `carrito_temporal={}` |
| `Confirmar al cliente` | Telegram | Mensaje al cliente con resumen del pedido en Markdown |
| `Stock Insuficientes` | Telegram | Mensaje de error de stock al usuario |
| `Guardar HIstorial` | googleSheets (update) | `pantalla_actual=VER_CATEGORIAS`, preserva `carrito_temporal` actual |
| `Mostrar categoría de Nuevo` | Telegram | Los 4 botones de categoría |
| `Update SESSION` | googleSheets (update) | `pantalla_actual=VER_CATEGORIAS`, `carrito_temporal={}` |
| `Aviso Cancelacion` | Telegram | *"Pedido cancelado. Escribe /start para hacer un nuevo pedido"* |

### Decisiones técnicas clave

- **Generación de ID:** `PED-{chatId}-{timestamp}` garantiza unicidad sin autoincrement.
- **Zona horaria Colombia:** `Date.now() - 5 * 60 * 60 * 1000` para fecha/hora correcta.
- **Descuento de stock por lotes:** `Stock Descuento` convierte el array en múltiples items; `Descontar los stoks` procesa cada uno como update separado por `id_producto`.

---

## Módulo 04 — Reportes y Ventas

**Archivo:** `Workflow/modulos/04_Reportes_Ventas.json`  
**Responsabilidad:** Reporte diario automático a las 7 AM con métricas del día.

### Nodos

| Nodo | Tipo | Descripción |
|------|------|-------------|
| `Schedule Trigger` | scheduleTrigger | Cron: todos los días a las 7:00 AM |
| `Leer Pedidos` | googleSheets (getAll) | Lee todas las filas de PEDIDOS sin filtro |
| `Métricas Del dia` | Code | Filtra pedidos del día actual (UTC-5), calcula total vendido, producto estrella y hora pico |
| `Actualizacion Reporte` | Telegram | Envía reporte al admin (`chat_id: 8907728238`) con métricas formateadas |

### Formato del reporte al administrador

```
📊 Reporte Diario — [FECHA]

💰 Total vendido: $XX.XXX
⭐ Producto estrella: [nombre] (N unidades)
⏰ Hora pico: HH:00 - HH:59 (N pedidos)
📦 Total de pedidos: N
```
