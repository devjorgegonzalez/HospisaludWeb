# Esquema de Base de Datos — HospisaludWeb

El modelo de datos normalizado para PostgreSQL 16+ se encuentra completamente especificado en:
👉 **[docs/data-model.md](file:///h:/Repos/HospisaludWeb/docs/data-model.md)**

### Resumen de Entidades Principales
1. `branches`: Sedes operativas físicas (Hospital, Centro, Petrucci, Guanipa).
2. `components`: Principios activos / componentes médicos para filtros y búsqueda.
3. `products`: Catálogo general con precios en USD y los 4 flags fijos (`is_oferta`, `is_receta_medica`, `is_venta_controlada`, `is_destacado`).
4. `product_components`: Relación N:M entre productos y principios activos.
5. `branch_inventories`: Stock independiente por cada una de las 4 sedes.
6. `stock_reservations`: Bloqueo de stock por 15 minutos en checkout.
7. `exchange_rates`: Histórico y valor diario de la tasa oficial BCV.
8. `users`: Roles administrativos (`SUPER_ADMIN`, `BRANCH_ADMIN`, `OPERATOR`) y clientes (`CLIENT`).
9. `user_addresses`: Libreta de direcciones con zonas urbana/interurbana.
10. `orders`: Cabecera de pedidos (Pickup o Delivery, pagos manuales, estados Kanban).
11. `order_items`: Detalle de productos por pedido.
12. `order_status_logs`: Auditoría y trazabilidad de cambios de estado.
