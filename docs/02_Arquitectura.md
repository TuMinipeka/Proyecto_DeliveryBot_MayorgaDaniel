# 02 — Arquitectura del Sistema

## Principio de Diseño

El sistema es una **máquina de estados conversacional**. Cada mensaje del usuario ejecuta el workflow completo. El flujo lee `pantalla_actual` desde la hoja SESSION, determina en qué estado está el usuario, ejecuta la lógica correspondiente y actualiza el estado antes de responder.

## Diagrama de Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                        USUARIO                              │
│                     (Telegram App)                          │
└───────────────────────────┬─────────────────────────────────┘
                            │ mensaje / callback
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    TELEGRAM BOT API                         │
│              (Webhook → n8n Trigger)                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       n8n Engine                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Módulo 01   │  │  Módulo 02   │  │    Módulo 03     │  │
│  │  Menú y Nav  │→ │  Carrito y   │→ │  Gestor de       │  │
│  │              │  │  Pedidos     │  │  Estados         │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                             │
│                    ┌──────────────┐                         │
│                    │  Módulo 04   │                         │
│                    │  Reportes    │ ← Schedule (7 AM)       │
│                    └──────────────┘                         │
└───────────────┬──────────────────────────────┬──────────────┘
                │                              │
                ▼                              ▼
┌───────────────────────────┐    ┌─────────────────────────┐
│      Google Sheets        │    │     Telegram (salida)   │
│      deliverybot          │    │  mensajes al usuario    │
│  ┌─────────┬───────────┐  │    │  y al administrador     │
│  │  MENU   │  PEDIDOS  │  │    └─────────────────────────┘
│  ├─────────┼───────────┤  │
│  │ USUARIOS│  SESSION  │  │
│  └─────────┴───────────┘  │
└───────────────────────────┘
```

## Máquina de Estados

```
                    /start
                      │
              ┌───────▼────────┐
              │     NUEVO      │ ← usuario sin sesión
              └───────┬────────┘
                      │ registro + crear sesión
                      ▼
              ┌───────────────────┐
         ┌───▶│  VER_CATEGORIAS   │◀──────────────────┐
         │    └────────┬──────────┘                   │
         │             │ selecciona categoría          │
         │             ▼                               │
         │    ┌────────────────┐                       │
         │    │    CARRITO     │                       │
         │    └────────┬───────┘                       │
         │             │ escribe número de producto    │
         │             ▼                               │
         │    ┌───────────────────────┐                │
         │    │ SELECCIONANDO_CANTIDAD│                │
         │    └───────────┬───────────┘                │
         │                │ escribe cantidad            │
         │                ▼                            │
         │    ┌───────────────────┐                    │
         │    │  CONFIRMAR_PEDIDO │                    │
         │    └─┬──────┬──────────┘                    │
         │  CONFIRMAR SEGUIR   CANCELAR                │
         │      │      └──────────────────────────────►│
         │      │                                      │
         │      ▼                                      │
         │  Registrar pedido + descontar stock         │
         │      │                                      │
         └──────┘ (vuelta a categorías)                │
                                                       │
                                         ──────────────┘
```

## Flujo Principal Completo

```
Telegram Disparador
  └─→ Ajuste Variables (extrae chat_id, nombre_completo, texto_usuario)
        └─→ Leer estado Usuario (SESSION por telegram_id)
              └─→ If1 ¿texto_usuario == "/start"?
                    TRUE  └─→ If ¿ya existe en SESSION?
                                TRUE  └─→ Reset Session → Categorias
                                FALSE └─→ Registro Usuario → Crear session → Categorias
                    FALSE └─→ Router De Estado (Switch por pantalla_actual)
                                NUEVO             → (misma lógica /start)
                                VER_CATEGORIAS    → Leer MENU → Construir lista → Fusionar Carrito
                                                   → Mostrar Comidas → Guardar Estado (CARRITO)
                                CARRITO           → Procesar Selección → Validar
                                                   → Guardar Siguiente Rama → Pedir Cantidad
                                SELECCIONANDO_CANTIDAD → Procesar Cantidad → Validar
                                                        → Guardar Estado CONFIRMAR → Resumen
                                CONFIRMAR_PEDIDO  → Switch CONFIRMAR/SEGUIR/CANCELAR
                                                   CONFIRMAR → Leer stock → Generar pedido
                                                               → Registrar → Descontar stock
                                                               → Limpiar SESSION → Confirmar
                                                   SEGUIR    → Guardar historial → Categorías
                                                   CANCELAR  → Limpiar SESSION → Aviso
```
