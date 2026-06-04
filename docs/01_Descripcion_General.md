# 01 — Descripción General

## Problema

En entornos institucionales como oficinas, universidades o grandes centros de trabajo, la gestión de pedidos de cafetería suele ser ineficiente: filas largas, errores en la toma de pedidos manual y nula visibilidad del estado de la orden.

## Solución: DeliveryBot

Sistema de automatización basado en **n8n** que convierte a **Telegram** en una terminal de pedidos inteligente. Los usuarios consultan el menú, arman su carrito y reciben notificaciones en tiempo real sobre el estado de su pedido. La administración obtiene reportes diarios automáticos.

## Objetivos

1. Implementar un sistema de pedidos digital mediante interfaz conversacional en Telegram.
2. Automatizar el cálculo de totales y la generación de números de orden únicos.
3. Gestionar el ciclo de vida del pedido: **Recibido → Preparación → En camino → Entregado**.
4. Centralizar el inventario y menú en Google Sheets para fácil actualización por el administrador.
5. Generar reportes diarios de ventas y métricas de productos más vendidos.
6. Optimizar la comunicación cocina–cliente mediante notificaciones push automáticas.

## Módulos del Sistema

| Módulo | Archivo | Responsabilidad |
|--------|---------|----------------|
| 01 — Menú y Navegación | `01_Menu_Navegacion.json` | Entrada, registro de usuario, presentación de categorías |
| 02 — Carrito y Pedidos | `02_Carrito_Pedidos.json` | Selección de productos, acumulación de carrito, confirmación |
| 03 — Gestor de Estados | `03_Gestor_Estados.json` | Validación de stock, registro de pedido, cancelación |
| 04 — Reportes y Ventas | `04_Reportes_Ventas.json` | Reporte diario automático a las 7 AM |

## Resultados Esperados

- Cero pérdida de pedidos gracias al registro automático en la nube.
- Reducción de tiempos de espera en un 40% al permitir pedidos anticipados.
- Transparencia total del estado de la orden para el usuario.
- Reporte diario con: total vendido, producto estrella y hora pico de pedidos.

## Stack Tecnológico

| Componente | Tecnología |
|-----------|-----------|
| Automatización | n8n |
| Interfaz de usuario | Telegram Bot API |
| Base de datos | Google Sheets |
| Comunicación en tiempo real | Telegram inline keyboards + callbacks |
