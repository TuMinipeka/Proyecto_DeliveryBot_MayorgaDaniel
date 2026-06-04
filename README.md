# Proyecto_DeliveryBot_MayorgaDaniel

Sistema de automatización de pedidos de cafetería institucional basado en **n8n** y **Telegram**, con base de datos en **Google Sheets**.

## Descripción

DeliveryBot convierte a Telegram en una terminal de pedidos inteligente. Los empleados o estudiantes consultan el menú, arman su carrito y reciben notificaciones en tiempo real sobre el estado de su pedido. El personal de cocina gestiona los estados de los pedidos con comandos desde su propio bot. La administración recibe reportes diarios automáticos de ventas.

## Estructura del Proyecto

```
proyecto/
├── README.md
├── .gitignore
├── Workflow/
│   ├── modulos/
│   │   ├── 01_Menu_Navegacion.json      ← Entrada, registro, categorías
│   │   ├── 02_Carrito_Pedidos.json      ← Selección, carrito, confirmación
│   │   ├── 03_Gestor_Estados.json       ← Stock, registro pedido, cancelación
│   │   └── 04_Reportes_Ventas.json      ← Reporte diario 7 AM al admin
│   └── DeliveryBot_Completo.json        ← Workflow unificado (entregable)
└── docs/
    ├── 01_Descripcion_General.md
    ├── 02_Arquitectura.md
    ├── 03_Modelo_Datos.md
    ├── 04_Flujos_n8n.md
    ├── 05_Guia_Configuracion.md
    ├── 06_Pruebas_Evidencias.md
    └── assets/
        ├── diagrama_arquitectura.png
        ├── capturas_modulo_01/
        ├── capturas_modulo_02/
        ├── capturas_modulo_03/
        └── capturas_modulo_04/
```

## Stack Tecnológico

| Componente | Tecnología |
|-----------|-----------|
| Automatización | n8n |
| Interfaz de usuario | Telegram Bot API |
| Base de datos | Google Sheets |

## Base de Datos

**Google Sheets — deliverybot**  
[Abrir Spreadsheet](https://docs.google.com/spreadsheets/d/1hNSqj9dSELHYo_l9msficSonMdmhy5mvYlhzMZdDBD4)

| Hoja | Propósito |
|------|-----------|
| MENU | Productos, categorías, stock y precios |
| PEDIDOS | Registro histórico de todos los pedidos |
| USUARIOS | Datos de clientes registrados |
| SESSION | Estado actual de la conversación por usuario |

## Módulos

| # | Módulo | Tipo | Trigger | Nodos principales |
|---|--------|------|---------|------------------|
| 01 | Menú y Navegación | Workflow principal | Telegram (mensaje) | Telegram Disparador, Ajuste Variables, Leer estado Usuario, Router De Estado |
| 02 | Carrito y Pedidos | Workflow principal | Telegram (mensaje) | Leer MENU, Construir lista, Fusionar Carrito, Procesar Selección, Resumen Carrito |
| 03 | Workflow Cocinero | **Independiente** | Telegram Disparador Cocinero | Detectar Comando, Parsear comando/estado, Validar comando, Buscar pedido, Actualizar Estado PEDIDOS, Notificar al cliente |
| 04 | Reporte Diario | **Independiente** | Schedule Trigger (7 AM diario) | Leer Pedidos, Code in JavaScript, Actualizacion Pedido1 |

### Módulo 03 — Workflow Cocinero

Workflow independiente para el personal de cocina. Permite actualizar el estado de un pedido vía Telegram y notifica automáticamente al cliente.

- **Comandos:** `/estado [ID] [NuevoEstado]`, `/ayuda`
- **Estados válidos:** `Recibido` → `Preparación` → `En camino` → `Entregado`

### Módulo 04 — Reporte Diario

Workflow independiente con Schedule Trigger. Genera y envía automáticamente un reporte de ventas del día al administrador todos los días a las 7:00 AM.

## Instalación Rápida

Ver [docs/05_Guia_Configuracion.md](docs/05_Guia_Configuracion.md) para instrucciones completas.

1. Importar `Workflow/DeliveryBot_Completo.json` en n8n.
2. Configurar credenciales de Telegram y Google Sheets.
3. Activar el workflow y enviar `/start` al bot.

## Documentación

- [Descripción General](docs/01_Descripcion_General.md)
- [Arquitectura del Sistema](docs/02_Arquitectura.md)
- [Modelo de Datos](docs/03_Modelo_Datos.md)
- [Flujos n8n](docs/04_Flujos_n8n.md)
- [Guía de Configuración](docs/05_Guia_Configuracion.md)
- [Pruebas y Evidencias](docs/06_Pruebas_Evidencias.md)

## Autor

**Daniel Mayorga** — [danieltomas0320@gmail.com](mailto:danieltomas0320@gmail.com)
