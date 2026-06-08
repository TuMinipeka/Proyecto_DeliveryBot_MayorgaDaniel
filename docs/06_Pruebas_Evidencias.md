# 06 — Pruebas y Evidencias

## Módulo 01 — Menú y Navegación

### Casos de prueba

| # | Acción | Resultado esperado | Estado |
|---|--------|-------------------|--------|
| 1.1 | Usuario nuevo envía `/start` | Bot registra usuario en USUARIOS, crea sesión en SESSION, muestra 4 categorías (Bebidas - Postres - Comidas - Almuerzos) | ✅ Aprobado |
| 1.2 | Usuario existente envía `/start` | Bot resetea la sesión (`pantalla_actual=VER_CATEGORIAS`, `carrito_temporal={}`) y muestra 4 categorías | ✅ Aprobado |
| 1.3 | Usuario envía texto aleatorio sin sesión | Bot responde: *"Para iniciar tu pedido, envía /start 🚀"* | ⬜ Pendiente |

### Evidencias
![DEcision de flujo al pulsar /start ](/docs/assets/capturas_modulo_01/flujostartn8n.png)

Mensaje reflejado en el Bot de Telegram

![Bot de Telegram respondiendo al /start](assets/capturas_modulo_01/bottelegramstart.png)

Se guarda dentro del google Sheets, Junto a la sesion guardada

![Evidencia de info guardada dentro de la tabla de Google Sheets](assets/capturas_modulo_01/googlesheetsconusuario.png)

![Evidencia de Session modificado al usuario solo escribir /start (u cualquier otro mensaje con el ChatBot)](assets/capturas_modulo_01/session1.png)

## Módulo 02 — Carrito y Pedidos

### Casos de prueba

| # | Acción | Resultado esperado | Estado |
|---|--------|-------------------|--------|
| 2.1 | Usuario selecciona categoría "Bebidas" | Bot muestra lista numerada de bebidas del menú (ej: 1. Coca Cola $6500, 2. Jugo Natural $4000) | ✅ Aprobado |
| 2.2 | Usuario ingresa número de producto válido (ej: 1) | Bot confirma "✅ Seleccionaste: Coca Cola — Precio: $6500" y solicita cantidad | ✅ Aprobado |
| 2.3 | Usuario ingresa número fuera de rango | Bot muestra error de selección inválida | ⬜ Pendiente |
| 2.4 | Usuario ingresa cantidad válida (ej: 3) | Bot muestra resumen del carrito "Coca Cola x3 = $19500 — Total: $19500" con 3 botones (Confirmar / Seguir comprando / Cancelar) | ✅ Aprobado |
| 2.5 | Usuario ingresa cantidad inválida (ej: "abc") | Bot muestra error de cantidad inválida | ✅ Aprobado |
| 2.6 | Usuario agrega productos de 2 categorías distintas | El carrito acumula ambos productos correctamente | ✅ Aprobado |

### Evidencias
Flujo de trabajo en n8n al seleccionar una categoría:

![Flujo de trabajo al escoger bebida en el catalogo](assets/capturas_modulo_02/flujon8ncategoria.png)

Al escoger la categoría Bebidas, el bot lista los productos disponibles:

![Pick al catalogo del bot de telegram](assets/capturas_modulo_02/escogercatalogo.png)

Al ingresar el número 1, el bot confirma "Coca Cola" y solicita la cantidad:

![Mensaje telegram confirmando la selección del producto y solicitando cantidad](assets/capturas_modulo_02/mensajetelegramcomida.png)

Flujo de trabajo en n8n al procesar la selección del producto:

![Flujo de trabajo formado por n8n al momento de escoger la bebida](assets/capturas_modulo_02/seleccioncomidaflujon8n.png)

Al ingresar la cantidad (3), el bot muestra el resumen del carrito con los 3 botones de acción:

![Mensaje de mi carrito para confirmar, seguir comprando o cancelar mi pedido](assets/capturas_modulo_02/tucarritotelegram.png)


## Módulo 03 — Gestor de Estados

### Casos de prueba

| # | Acción | Resultado esperado | Estado |
|---|--------|-------------------|--------|
| 3.1 | Usuario presiona "✅ Confirmar" con stock disponible | Bot confirma al usuario ("¡Pedido confirmado! N° PED-xxx") y envía notificación al cocinero ("🔔 Nuevo pedido"); pedido se registra en PEDIDOS | ✅ Aprobado |
| 3.2 | Usuario presiona "✅ Confirmar" con stock insuficiente | Bot responde "⚠️ Stock insuficiente: ❌ [producto]: solo quedan 0 unidades (pediste 2). Modifica tu carrito o escribe /start para reiniciar." | ✅ Aprobado |
| 3.3 | Usuario presiona "➕ Seguir comprando" | Bot muestra el resumen del carrito actual y luego las 4 categorías nuevamente (Bebidas - Postres - Comidas - Almuerzos) | ✅ Aprobado |
| 3.4 | Usuario presiona "❌ Cancelar pedido" | Bot limpia el carrito, muestra aviso de cancelación | ⬜ Pendiente |
| 3.5 | Verificar hoja PEDIDOS tras confirmación | Se registra fila con: `id_pedido`, `id_usuario`, `detalle_pedido` (ej: Coca Cola x3), `total_pagar` (ej: 19500), `estado=Recibido`, `fecha`, `hora` | ✅ Aprobado |
| 3.6 | Verificar hoja SESSION al presionar "✅ Confirmar" | `pantalla_actual=CONFIRMAR_PEDIDO`, `carrito_temporal` contiene los ítems del pedido activo | ✅ Aprobado |
| 3.7 | Cocinero envía `/estado PED-xxx-xxx Preparación` | Bot actualiza estado en PEDIDOS, cliente recibe notificación con emoji 👨‍🍳, cocinero recibe confirmación | ⬜ Pendiente |
| 3.8 | Cocinero envía `/estado` con ID inexistente | Bot responde "❌ Pedido X no encontrado" al cocinero | ✅ Aprobado |
| 3.9 | Cocinero envía `/ayuda` | Bot muestra la guía completa de comandos al cocinero | ✅ Aprobado |

### Evidencias
Al darle click al boton CONFIRMAR:

![Despues de oprimir el boton confirmar en telegram este el mesaje de pedido confirmado](assets/capturas_modulo_03/chattelegramconfirmar.png)

Registra el pedido dentro del Google Sheets dentro de la hoja Usuario.

![Tabla de datos del google sheets guardado](assets/capturas_modulo_03/pedidogooglesheets.png)

Junto la pantalla actual en la que se encuentra en la hoja session:

![Google sheets donde se ve la pantalla actual de session junto sus otros valores](assets/capturas_modulo_03/sheetssession.png)

Si quieres segir comprando, te dara el catalogo para que lo agregues a tu carrito:

![Ouput telegram al seguir comprando](assets/capturas_modulo_03/seguircomprando.png)

Si no hay suficientes Coca Colas dentro del Stock.

![Mensaje que envia el flujo por medio del bot de telegram al aver unidades insufucientes dentro del stock](assets/capturas_modulo_03/salida0stock.png)

Al cancelar carrito, este solo te dira que el pedido ha sido canselado.

Flujo de trabajo cocinero :



---

## Módulo 04 — Reportes y Ventas

### Casos de prueba

| # | Acción | Resultado esperado | Estado |
|---|--------|-------------------|--------|
| 4.1 | Ejecución del Schedule Trigger | Admin recibe mensaje de reporte con fecha, total vendido, pedidos procesados, producto estrella y hora pico | ✅ Aprobado |
| 4.2 | Reporte con 0 pedidos del día | Admin recibe mensaje indicando sin actividad | ⬜ Pendiente |
| 4.3 | Reporte con múltiples pedidos | Total vendido: $99500, Pedidos procesados: 2, Producto estrella: Coca Cola (3 unidades), Hora pico: 21:00 (2 pedidos) | ✅ Aprobado |

### Evidencias
Workflow De reporte diario en n8n

![Flujo principal del reporte del dia en n8n](assets/capturas_modulo_04/flujotrabajon8nreportesdiarios.png)

Mensaje del reoprte del dia

![Mensaje reporte diario con el bor de telegram](assets/capturas_modulo_04/reportetelegram.png)

---