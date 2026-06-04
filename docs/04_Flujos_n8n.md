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

## Módulo 03 — Workflows Secundarios

**Archivo:** `Workflow/modulos/03_Gestor_Estados.json`

Este archivo contiene **tres flujos independientes** que se importan en el mismo canvas de n8n. Cada uno tiene su propio trigger y puede activarse o desactivarse por separado sin afectar a los demás:

| Flujo | Trigger | Responsabilidad |
|-------|---------|----------------|
| Confirmación del pedido | Callback de Telegram (`CONFIRMAR / SEGUIR / CANCELAR`) | Validación de stock, registro y despacho del pedido *(documentado en Módulo 02)* |
| Workflow Cocinero | Mensaje de Telegram (`/estado`, `/ayuda`) | Actualizar estado de pedidos y notificar al cliente |
| Reporte Diario | Schedule Trigger (7:00 AM diario) | Generar y enviar métricas del día al administrador |

---

### Flujo 2 — Workflow Cocinero

**Tipo:** Workflow independiente — no conectado al Telegram Disparador principal.  
**Responsabilidad:** Permite al personal de cocina actualizar el estado de un pedido via Telegram. Al actualizar, notifica automáticamente al cliente.

#### Trigger

`Telegram Disparador Cocinero` — Webhook ID: `cb5135a2-2dcc-4f9d-8615-4b696728eb59`, escucha solo `message`.

#### Comandos disponibles

| Comando | Uso |
|---------|-----|
| `/estado [ID] [NuevoEstado]` | Actualiza el estado de un pedido |
| `/ayuda` | Muestra la guía de uso |

**Estados válidos:** `Recibido` / `Preparación` / `En camino` / `Entregado`

#### Flujo completo

```
Telegram Disparador Cocinero
  └─→ Detectar Comando (Switch por primera palabra)
        Output 0 (/estado) → Parsear comando/estado
                              → Validar comando (IF: valido == true)
                                  TRUE  → Buscar pedido (Get row PEDIDOS por id_pepido)
                                            → EL pedido existe? (IF: id_pepido not empty)
                                                TRUE  → Actualizar Estado PEDIDOS
                                                          → Preparar Notificaciones
                                                          → Actualizacion Pedido (→ cliente)
                                                          → Notificar al cliente (→ admin confirma)
                                                FALSE → Pedido No Encontrado (→ admin)
                                  FALSE → Error Formato Inválido (→ admin)
        Output 1 (/ayuda)  → Guia de Uso (→ cocinero)
```

#### Nodos

| Nodo | Tipo | Descripción |
|------|------|-------------|
| `Telegram Disparador Cocinero` | telegramTrigger | Webhook que escucha `message`. Mismo bot, lógica separada |
| `Detectar Comando` | Switch | Detecta la primera palabra del texto (`split(' ')[0]`). Rama 0 = `/estado`, rama 1 = `/ayuda` |
| `Parsear comando/estado` | Code | Divide el mensaje, extrae `id_pedido` (partes[1]) y `nuevo_estado` (partes[2..].join(' ')), valida que el estado esté en la lista válida |
| `Validar comando` | IF | `$json.valido is true` |
| `Buscar pedido` | googleSheets (get) | Filtra PEDIDOS por `id_pepido == String($json.id_pedido).trim()`. `alwaysOutputData: true` |
| `EL pedido existe?` | IF | `$json.id_pepido is not empty` |
| `Actualizar Estado PEDIDOS` | googleSheets (update) | Actualiza PEDIDOS por `id_pepido`, solo modifica el campo `estado` |
| `Preparar Notificaciones` | Code | Construye `mensaje_cliente` con emoji según estado, construye `mensaje_admin` de confirmación, lee `id_usuario` del nodo Buscar pedido |
| `Actualizacion Pedido` | Telegram | Envía `mensaje_cliente` al `id_usuario` (el cliente) |
| `Notificar al cliente` | Telegram | Envía `mensaje_admin` al cocinero como confirmación |
| `Pedido No Encontrado` | Telegram | Envía al admin: *"❌ Pedido X no encontrado"* |
| `Error Formato Inválido` | Telegram | Envía al admin el `mensaje_error` del parser |
| `Guia de Uso` | Telegram | Envía instrucciones completas del comando `/estado` al cocinero |

#### Emojis de estado (Preparar Notificaciones)

| Estado | Emoji |
|--------|-------|
| Recibido | 📋 |
| Preparación | 👨‍🍳 |
| En camino | 🛵 |
| Entregado | ✅ |

#### Texto de la guía (`/ayuda`)

```
👨‍🍳 Guía Rápida - Cocina

Para cambiar el estado de un pedido escribe:
/estado [ID] [Estado]

Ejemplo: /estado PED-123-456 Preparación

Estados válidos:
- Recibido
- Preparación
- En camino
- Entregado

El ID del pedido lo encuentras en la hoja PEDIDOS de Google Sheets.
Escribe /ayuda para ver esto de nuevo.
```

---

### Flujo 3 — Reporte Diario

**Tipo:** Workflow independiente — trigger por Schedule, no por Telegram.  
**Responsabilidad:** Genera y envía automáticamente un reporte de ventas del día al administrador todos los días a las 7:00 AM.

#### Flujo completo

```
Schedule Trigger (7 AM diario)
  └─→ Leer Pedidos (Get all rows PEDIDOS sin filtro)
        └─→ Code in JavaScript (calcula métricas del día)
              └─→ Actualizacion Pedido1 (Telegram → admin chat_id: 8907728238)
```

#### Nodos

| Nodo | Tipo | Descripción |
|------|------|-------------|
| `Schedule Trigger` | scheduleTrigger | Cron: `triggerAtHour: 7`, todos los días |
| `Leer Pedidos` | googleSheets (getAll) | Lee todas las filas de PEDIDOS sin filtro. `alwaysOutputData: true` |
| `Code in JavaScript` | Code | Filtra pedidos del día (`new Date().toISOString().split('T')[0]`), calcula `totalVendido`, `productoEstrella` (parsea `detalle_pedido` formato `nombre x cantidad \| ...`), `horaPico` (agrupa por hora) |
| `Actualizacion Pedido1` | Telegram | Envía el reporte al admin (`chat_id: 8907728238`) |

#### Lógica del Code node

1. Filtra pedidos donde `fecha == hoy` (ISO date string).
2. Si no hay pedidos: envía mensaje *"No hubo pedidos hoy"*.
3. `totalVendido` — suma de `total_pagar` de los pedidos del día.
4. `productoEstrella` — parsea `detalle_pedido` (`nombre x cantidad | ...`), suma cantidades por producto, devuelve el más vendido.
5. `horaPico` — agrupa pedidos por la hora (`hora.split(':')[0]`), devuelve la hora con más pedidos.

#### Formato del reporte

```
📊 Reporte Diario — YYYY-MM-DD

💰 Total vendido: $X
📦 Pedidos procesados: N
⭐ Producto estrella: NombreProducto (X unidades)
⏰ Hora pico: HH:00 (N pedidos)

Generado automáticamente por DeliveryBot
```
