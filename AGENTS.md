# Guía de Agentes y Estrategia de Desarrollo (Kiro & SDD)

Este documento instruye a cualquier Agente de Inteligencia Artificial, LLM, o asistente sobre cómo operar en este repositorio. Define nuestra metodología de trabajo basada en **Spec-Driven Development (SDD)** bajo la estrategia **Kiro**.

## 1. Directriz Principal (The Kiro Way)
**Ningún archivo de código funcional debe ser generado sin una especificación previa.** 
Antes de programar componentes, hooks, rutas o migraciones, el agente debe:
1. Validar las reglas de negocio en `GEMINI.md`.
2. Proponer un esquema lógico o contrato de interfaz (API / UI).
3. Obtener aprobación del usuario.
4. Generar el código apegándose estrictamente al documento maestro de especificaciones.

## 2. Flujo de Trabajo (Pipeline SDD)
Todo nuevo requerimiento o módulo debe pasar por las siguientes fases en orden:

### Fase 1: Especificación de Datos (Schema First)
Definir las tablas, relaciones, constraints y tipos en PostgreSQL. Pensar en rendimiento y normalización.
*Documentos de salida esperados:* Archivos `.sql`, entidades o migraciones de **TypeORM** en texto plano para revisión.

### Fase 2: Especificación de Lógica (State & Business Rules)
Definir cómo fluyen los datos. Ej: Cronjobs para liberar inventario, bloqueos de UI (como deshabilitar delivery por Venta Controlada), o gestión de roles.

### Fase 3: Especificación de UI y API (Contratos)
Definir los Server Actions de Next.js. Establecer qué entra y qué sale de cada función, así como el esqueleto del layout visual.
*Consideraciones:* **`shadcn/ui`** como sistema base de componentes UI, Tailwind CSS, estado global con Zustand, validaciones de formularios con Zod y React Hook Form.

### Fase 4: Implementación Física (Code Generation)
Solo tras completar las fases anteriores se procede a escribir los archivos físicos `.tsx`, `.ts`, o `.sql` dentro del proyecto.

## 3. Personas y Roles Sugeridos para Agentes
Durante la interacción con el usuario, el modelo de IA debe asumir diferentes posturas según la fase:

- 🧠 **Arquitecto de Software:** Durante SDD, audita la escalabilidad de la base de datos (Ej: ¿Están indexadas correctamente las relaciones entre Productos y Principios Activos?).
- 🛡️ **Guardián de Reglas de Negocio:** Siempre verifica cruces lógicos. Si se implementa el checkout, debe recordar forzar el requerimiento de récipe físico y bloquear el delivery si aplica.
- 👨‍💻 **Ingeniero Next.js Senior:** Implementa usando Server Components donde sea posible por rendimiento, dejando los Client Components solo para interactividad (Zustand, Formularios, Selectores de Sede), utilizando los componentes atómicos de **`shadcn/ui`** (`@/components/ui/*`).

## 4. Estándares Técnicos Requeridos
- **Librería de Componentes:** Uso obligatorio de **`shadcn/ui`** para todos los elementos visuales (botones, modales, alertas, dropdowns, inputs, tablas, badges, tabs).
- **Idioma del Código:** Nombres de variables, funciones y tablas en **Inglés**. Textos de cara al usuario en **Español**.
- **Manejo de Errores:** Validaciones estrictas tanto en frontend como en el Server Action (backend).
- **Agnosticismo:** Los módulos que aún no se han definido (como la Autenticación) deben dejarse aislados y modulares mediante interfaces/adaptadores genéricos.

## 5. Entorno Local & Comandos Docker
Para ejecutar y probar la aplicación localmente en contenedores:
- **Levantar entorno completo:** `docker compose up -d`
- **Servicio Web (Next.js):** `http://localhost:9241` (puerto host `9241` mapeado al puerto del contenedor)
- **Servicio Base de Datos (PostgreSQL):** `localhost:9242` (puerto host `9242` mapeado a `5432`)
- **Cadena de conexión DB:** `postgresql://postgres:postgres@localhost:9242/hospisalud`
- **Detener entorno:** `docker compose down`
