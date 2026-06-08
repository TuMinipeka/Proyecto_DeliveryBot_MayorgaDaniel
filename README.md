# DeliveryBot — Automatización de Pedidos Institucional

> Sistema de pedidos para cafetería construido sobre **n8n**, **Telegram** y **Google Sheets**.  
> Sin apps extras. Sin código de servidor. Solo automatización.

---

## Contenido

- [Descripción](#descripción)
- [Stack tecnológico](#stack-tecnológico)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Base de datos](#base-de-datos)
- [Módulos](#módulos)
- [Instalación rápida](#instalación-rápida)
- [Documentación](#documentación)

---

## Descripción

DeliveryBot convierte Telegram en una terminal de pedidos inteligente para empleados y estudiantes.  
El flujo completo va desde la selección del menú hasta la notificación de entrega, sin intervención manual del administrador.

**Qué puede hacer el sistema:**

- El cliente consulta el menú, arma su carrito y confirma el pedido desde Telegram
- El bot valida stock en tiempo real antes de registrar el pedido
- El cocinero actualiza el estado del pedido con un comando y el cliente recibe la notificación automáticamente
- El administrador recibe un reporte de ventas diario a las 7:00 AM

---

## Stack tecnológico

| Componente | Tecnología |
|---|---|
| Motor de automatización | n8n |
| Interfaz de usuario | Telegram Bot API |
| Base de datos | Google Sheets |

---

## Estructura del proyecto

```
proyecto/
├── README.md
├── .gitignore
│
├── Workflow/
│   ├── DeliveryBot_Completo.json        ← Workflow unificado (entregable principal)
│   ├── flujodelcocinero.json            ← Workflow cocinero (exportado de n8n)
│   ├── reportediario.json               ← Workflow reporte diario (exportado de n8n)
│   │
│   └── modulos/                         ← Descripciones por módulo
│       ├── 01_Menu_Navegacion.json      ← Entrada, registro de usuario, categorías
│       ├── 02_Carrito_Pedidos.json      ← Selección de producto, carrito, totales
│       ├── 03_Gestor_Estados.json       ← Stock, registro de pedido, cancelación
│       ├── 04_Reportes_Ventas.json      ← Contexto del módulo de reportes
│       ├── 05_Flujo_Cocinero.json       ← Descripción del workflow del cocinero
│       └── 06_Reporte_Diario.json       ← Descripción del workflow de reporte diario
│
└── docs/
    ├── 01_Descripcion_General.md
    ├── 02_Arquitectura.md
    ├── 03_Modelo_Datos.md
    ├── 04_Flujos_n8n.md
    ├── 05_Guia_Configuracion.md
    ├── 06_Pruebas_Evidencias.md
    └── assets/
        ├── capturas_modulo_01/          ← /start, registro, sesión
        ├── capturas_modulo_02/          ← catálogo, selección, carrito
        ├── capturas_modulo_03/          ← confirmación, stock, cocinero
        └── capturas_modulo_04/          ← flujo y mensaje del reporte
```

---

## Base de datos

**Google Sheets — `deliverybot`**

| Hoja | Propósito |
|---|---|
| `MENU` | Productos, categorías, precios y stock |
| `PEDIDOS` | Registro histórico de todos los pedidos |
| `USUARIOS` | Datos de clientes registrados |
| `SESSION` | Estado actual de la conversación por usuario |

---

## Módulos

| # | Nombre | Tipo | Trigger |
|---|---|---|---|
| 01 | Menú y Navegación | Workflow principal | Telegram — mensaje entrante |
| 02 | Carrito y Pedidos | Workflow principal | Telegram — mensaje entrante |
| 03 | Gestor de Estados | Workflow principal | Telegram — mensaje entrante |
| 04 | Reportes y Ventas | Contexto del módulo | — |
| 05 | Flujo del Cocinero | **Workflow independiente** | Telegram — bot de cocina |
| 06 | Reporte Diario | **Workflow independiente** | Schedule Trigger — 7:00 AM |

### Módulo 05 — Flujo del Cocinero

El personal de cocina gestiona los pedidos con comandos desde su propio bot de Telegram:

- `/estado [ID] [NuevoEstado]` — actualiza el estado en PEDIDOS y notifica al cliente
- `/ayuda` — muestra la guía de comandos
- **Estados:** `Recibido` → `Preparación` → `En camino` → `Entregado`

### Módulo 06 — Reporte Diario

Genera y envía al administrador un resumen automático cada día a las 7:00 AM con:
total vendido, pedidos procesados, producto estrella y hora pico.

---

## Instalación rápida

Ver [`docs/05_Guia_Configuracion.md`](docs/05_Guia_Configuracion.md) para instrucciones detalladas.

1. Importar `Workflow/DeliveryBot_Completo.json` en n8n
2. Importar `Workflow/flujodelcocinero.json` y `Workflow/reportediario.json` como workflows separados
3. Configurar credenciales de Telegram y Google Sheets en los tres workflows
4. Activar los workflows y enviar `/start` al bot del cliente

---

## Documentación

| Documento | Contenido |
|---|---|
| [01 — Descripción General](docs/01_Descripcion_General.md) | Visión general del sistema |
| [02 — Arquitectura](docs/02_Arquitectura.md) | Diagrama y decisiones de diseño |
| [03 — Modelo de Datos](docs/03_Modelo_Datos.md) | Estructura de las hojas de Google Sheets |
| [04 — Flujos n8n](docs/04_Flujos_n8n.md) | Detalle de cada nodo y rama |
| [05 — Guía de Configuración](docs/05_Guia_Configuracion.md) | Instalación paso a paso |
| [06 — Pruebas y Evidencias](docs/06_Pruebas_Evidencias.md) | Casos de prueba y capturas |

---

**Autor De Este Proyectazo:** Daniel Mayorga — [danielsantiagomayorga03@gmail.com](mailto:danielsantiagomayorga03@gmail.com)
