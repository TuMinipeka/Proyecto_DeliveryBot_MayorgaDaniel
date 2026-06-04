# 05 — Guía de Configuración

## Requisitos Previos

- Cuenta de n8n (self-hosted o cloud)
- Bot de Telegram creado via [@BotFather](https://t.me/BotFather)
- Cuenta de Google con acceso a Google Sheets
- Google Sheets del proyecto configurado con las 4 hojas

---

## Paso 1 — Configurar Google Sheets

1. Abrir el archivo [deliverybot](https://docs.google.com/spreadsheets/d/1hNSqj9dSELHYo_l9msficSonMdmhy5mvYlhzMZdDBD4).
2. Verificar que existan las 4 hojas: **MENU**, **PEDIDOS**, **USUARIOS**, **SESSION**.
3. Agregar productos de prueba en la hoja **MENU** con los campos: `id_producto`, `nombre`, `descripcion`, `precio`, `categoria`, `stock`.
4. Dejar las hojas **PEDIDOS**, **USUARIOS** y **SESSION** vacías (solo los encabezados).

**Encabezados requeridos por hoja:**

- **MENU:** `id_producto | nombre | descripcion | precio | categoria | stock`
- **PEDIDOS:** `id_pedido | id_usuario | detalle_pedido | total_pagar | estado | fecha | hora`
- **USUARIOS:** `telegram_id | nombre_completo | puntos_lealtad`
- **SESSION:** `telegram_id | pantalla_actual | carrito_temporal | ultimo_cambio`

---

## Paso 2 — Configurar Credenciales en n8n

### Credencial de Telegram
1. En n8n, ir a **Credentials → New**.
2. Buscar **Telegram API**.
3. Ingresar el token del bot obtenido de @BotFather.
4. Guardar con el nombre `Telegram account`.
   - **Credential ID asignado:** `wlHAIXNKATZGf4dx`

### Credencial de Google Sheets
1. En n8n, ir a **Credentials → New**.
2. Buscar **Google Sheets OAuth2 API**.
3. Autenticar con la cuenta de Google que tiene acceso al Spreadsheet.
4. Guardar con el nombre descriptivo.
   - **Credential ID asignado:** `F1mDXLGolfF5triT`

---

## Paso 3 — Importar los Workflows

### Opción A — Workflow completo (recomendado para producción)
1. En n8n, ir a **Workflows → Import from file**.
2. Seleccionar `Workflow/DeliveryBot_Completo.json`.
3. Activar el workflow.

### Opción B — Módulos separados (recomendado para revisión)
1. Importar los **3 archivos** de `Workflow/modulos/` en orden (no 4 — el Reporte Diario vive dentro del módulo 03):
   - `01_Menu_Navegacion.json`
   - `02_Carrito_Pedidos.json`
   - `03_Gestor_Estados.json` ← contiene 3 flujos: confirmación del cliente + Workflow Cocinero + Reporte Diario

> **Nota — Workflow Cocinero:** usa el mismo bot de Telegram que el flujo principal, pero su trigger (`Telegram Disparador Cocinero`) escucha únicamente mensajes de texto (`message`), no callbacks de botones inline. Ambos triggers pueden coexistir en el mismo bot sin conflicto.

> **Nota — Reporte Diario:** el Schedule Trigger **no se activa automáticamente** al importar. Después de importar `03_Gestor_Estados.json`, abrir el canvas en n8n, localizar el nodo `Schedule Trigger (7 AM diario)` y activar el workflow manualmente para que comience a ejecutarse cada día a las 7:00 AM.

---

## Paso 4 — Configurar el Webhook de Telegram

1. Activar el workflow en n8n (el nodo `Telegram Disparador` genera la URL del webhook).
2. El **Webhook ID** del proyecto es: `cb5135a2-2dcc-4f9d-8615-4b696728eb59`
3. Verificar que el bot responde enviando `/start` en Telegram.

---

## Paso 5 — Verificar la Configuración

| Verificación | Esperado |
|-------------|---------|
| Enviar `/start` al bot | Aparecen los 4 botones de categoría |
| Seleccionar una categoría | Aparece la lista de productos |
| Ingresar un número de producto | El bot pide la cantidad |
| Ingresar una cantidad | Aparece el resumen del carrito con 3 botones |
| Presionar "✅ Confirmar" | El bot confirma el pedido y se registra en PEDIDOS |
| A las 7 AM | El admin recibe el reporte diario |

---

## Referencias Rápidas

| Dato | Valor |
|------|-------|
| Spreadsheet ID | `1hNSqj9dSELHYo_l9msficSonMdmhy5mvYlhzMZdDBD4` |
| Admin chat_id | `8907728238` |
| Webhook ID | `cb5135a2-2dcc-4f9d-8615-4b696728eb59` |
| Telegram credential ID | `wlHAIXNKATZGf4dx` |
| Google Sheets credential ID | `F1mDXLGolfF5triT` |
