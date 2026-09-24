# Spec: Geolocalización, Contexto de Sede y Enrutamiento Inteligente

**Módulo:** 03 · **Estado:** Aprobado · **Fecha:** 2026-09-24  
**Referencia:** [GEMINI.md](file:///h:/Repos/HospisaludWeb/GEMINI.md)

---

## 1. Problema
Los usuarios deben ser asociados a la farmacia más cercana a su ubicación física para optimizar los tiempos de entrega y verificar el stock real disponible. Si el usuario rechaza dar su ubicación, la plataforma no debe bloquearse sino asignar una sede por defecto (Sede Hospital). Además, para la zona de Guanipa, si la sucursal local no dispone del stock completo, el sistema debe aplicar una regla de enrutamiento especial hacia las sucursales de El Tigre, cobrando una tarifa fija de $5 USD tras aprobación del operador.

---

## 2. Historias de Usuario
- **US-03.1:** Como visitante, al ingresar a la web, quiero que el sistema me solicite permiso de GPS para sugerirme automáticamente la sede más cercana.
- **US-03.2:** Como visitante, si rechazo el permiso de GPS, quiero continuar navegando sin interrupciones con la Sede Hospital asignada por defecto.
- **US-03.3:** Como visitante, quiero poder cambiar mi sede manualmente en cualquier momento mediante un selector en la barra superior.
- **US-03.4:** Como cliente de Guanipa, si la sede Guanipa no tiene todo mi pedido, quiero que el sistema me permita enrutar el despacho desde El Tigre bajo tarifa especial de $5.00 USD.

---

## 3. Criterios de Aceptación (Notación EARS)

### Ubicuo
- **AC-03.1:** El sistema deberá persistir la sede activa del usuario en una cookie segura (`hospisalud_branch_id`) para mantener el contexto entre páginas y sesiones.
- **AC-03.2:** SI el usuario deniega o bloquea los permisos de geolocalización, ENTONCES el sistema deberá asignar por defecto la **Sede Hospital** (`code = 'HOSPITAL'`).

### Dirigido por Evento
- **CUANDO** el usuario concede permisos de ubicación en el navegador, el sistema deberá calcular las distancias euclidianas/Haversine hacia las 4 sedes y seleccionar automáticamente la más cercana.
- **CUANDO** el usuario cambia manualmente la sede en el dropdown de la barra superior, el sistema deberá actualizar el contexto global, refrescar los stocks visibles en el catálogo y notificar si algún producto del carrito cambia de disponibilidad.
- **CUANDO** se procesa un pedido con destino a Guanipa y la Sede Guanipa no cuenta con el 100% del stock solicitado, el sistema deberá asignar el pedido a una sede de El Tigre con stock completo, marcar su estado inicial en `EN_ESPERA_GUANIPA` y fijar la tarifa de delivery en $5.00 USD.

### Comportamiento no deseado
- **SI** ninguna sede en El Tigre ni en Guanipa cuenta con el stock completo de los productos solicitados, **ENTONCES** el sistema deberá notificar al usuario antes del pago indicando qué artículos no tienen disponibilidad total.

---

## 4. Modelo de Datos Relevante
- `branches`: `id`, `code`, `name`, `latitude`, `longitude`, `is_active`.
- `orders`: `branch_id`, `delivery_type`, `delivery_zone`, `delivery_fee_usd`, `order_status` (`EN_ESPERA_GUANIPA`).

### Coordenadas Base de Sedes:
- **Hospital:** Lat 8.88720, Lng -64.24560
- **Centro:** Lat 8.89500, Lng -64.25000
- **Petrucci:** Lat 8.87800, Lng -64.26000
- **Guanipa:** Lat 8.88200, Lng -64.16500

---

## 5. Contratos de Server Actions y Funciones
- `calculateClosestBranch(lat: number, lng: number)`: Devuelve la sede con menor distancia en km.
- `setBranchCookie(branchId: string)`: Guarda la cookie de sede en el servidor.
- `evaluateOrderRouting(items: CartItem[], destinationZone: 'URBANA' | 'INTERURBANA' | 'GUANIPA')`: Devuelve `{ assignedBranchId: string, isSpecialGuanipaRoute: boolean, suggestedFee: number, status: OrderStatus }`.

---

## 6. Casos Límite
- Dispositivos sin soporte de Geolocation API (browsers antiguos): Fallback silencioso a Sede Hospital.
- Coordenadas GPS fuera del estado Anzoátegui: Asignar Sede Hospital e informar en checkout sobre zonas de cobertura.

---

## 7. Fuera de Alcance
- Integración con dispositivos GPS en vivo de motos de repartidores (MVP usa enrutamiento estático de sedes).
