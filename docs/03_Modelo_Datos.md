# 03 — Modelo de Datos

## Google Sheets: deliverybot

**Spreadsheet ID:** `1hNSqj9dSELHYo_l9msficSonMdmhy5mvYlhzMZdDBD4`  
**Enlace:** [Abrir Google Sheets](https://docs.google.com/spreadsheets/d/1hNSqj9dSELHYo_l9msficSonMdmhy5mvYlhzMZdDBD4)

---

## Hoja: MENU (gid=0)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id_producto` | string | Identificador único del producto (ej: P001) |
| `nombre` | string | Nombre del producto |
| `descripcion` | string | Descripción corta |
| `precio` | number | Precio en pesos colombianos |
| `categoria` | string | Bebidas / Postres / Comidas / Almuerzos |
| `stock` | number | Unidades disponibles |

**Ejemplo de datos:**
```
id_producto | nombre        | descripcion          | precio | categoria | stock
P001        | Café          | Café tinto negro     | 2000   | Bebidas   | 50
P002        | Jugo de naranja| Jugo natural 350ml  | 3000   | Bebidas   | 30
P003        | Empanada      | Empanada de carne    | 2500   | Comidas   | 40
```

---

## Hoja: PEDIDOS (gid=792717060)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id_pedido` | string | ID único formato `PED-{chatId}-{timestamp}` |
| `id_usuario` | string | `telegram_id` del cliente |
| `detalle_pedido` | string | Items en formato `nombre x cantidad \| ...` |
| `total_pagar` | number | Total en pesos colombianos |
| `estado` | string | Recibido / Preparación / En camino / Entregado |
| `fecha` | string | Formato YYYY-MM-DD (zona UTC-5 Colombia) |
| `hora` | string | Formato HH:MM:SS (zona UTC-5 Colombia) |

**Ejemplo de datos:**
```
id_pedido              | id_usuario  | detalle_pedido             | total_pagar | estado   | fecha      | hora
PED-123456789-1717000  | 123456789   | Café x 2 | Empanada x 1   | 6500        | Recibido | 2026-06-04 | 10:30:00
```

---

## Hoja: USUARIOS (gid=1204490906)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `telegram_id` | string | ID único de Telegram |
| `nombre_completo` | string | Nombre del usuario |
| `puntos_lealtad` | number | Sistema de puntos (base 0) |

**Ejemplo de datos:**
```
telegram_id | nombre_completo  | puntos_lealtad
123456789   | Daniel Mayorga   | 0
987654321   | Ana García       | 15
```

---

## Hoja: SESSION (gid=1284889856)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `telegram_id` | string | FK hacia USUARIOS |
| `pantalla_actual` | string | Estado actual de la máquina |
| `carrito_temporal` | JSON string | Objeto con items acumulados y mapa de productos |
| `ultimo_cambio` | datetime | Timestamp de última modificación |

**Estados válidos de `pantalla_actual`:**
- `NUEVO`
- `VER_CATEGORIAS`
- `CARRITO`
- `SELECCIONANDO_CANTIDAD`
- `CONFIRMAR_PEDIDO`

---

## Estructura del carrito_temporal

### Estado: CARRITO (navegando productos)
```json
{
  "1": { "id": "P001", "nombre": "Café", "precio": 2000 },
  "2": { "id": "P002", "nombre": "Jugo", "precio": 3000 },
  "items": []
}
```

### Estado: SELECCIONANDO_CANTIDAD (ingresando cantidad)
```json
{
  "producto_seleccionado": {
    "id": "P001",
    "nombre": "Café",
    "precio": 2000
  },
  "mapa": {
    "1": { "id": "P001", "nombre": "Café", "precio": 2000 },
    "2": { "id": "P002", "nombre": "Jugo", "precio": 3000 }
  },
  "items": [
    { "id": "P001", "nombre": "Café", "precio": 2000, "cantidad": 2, "subtotal": 4000 }
  ]
}
```

### Estado: CONFIRMAR_PEDIDO (listo para confirmar)
```json
{
  "items": [
    { "id": "P001", "nombre": "Café", "precio": 2000, "cantidad": 2, "subtotal": 4000 }
  ],
  "total": 4000
}
```

---

## Diagrama de Relaciones

```
USUARIOS (telegram_id)
      │
      │ FK
      ▼
SESSION (telegram_id) ─── pantalla_actual ──→ estados de la máquina
                      └── carrito_temporal ──→ referencias a MENU.id_producto

MENU (id_producto) ←── referencias desde carrito_temporal y PEDIDOS
      │
      │ stock descuenta al confirmar pedido
      ▼
PEDIDOS (id_pedido) ──→ id_usuario → USUARIOS.telegram_id
```
