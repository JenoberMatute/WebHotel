# Documento de Contexto para web especializada hacia el Hotel Circulo Militar Fuerzas Armadas

> **Propósito:** Definir de forma clara y estructurada cada una de las funciones a desarrollar dentro de la web del hotel. Este documento sirve como esquema de trabajo para llevar a cabo el desarrollo por partes, de forma óptima y ordenada.
>
> **Alcance:** Cubre roles, modelo de datos, reglas de negocio, permisos, requisitos no funcionales, estados del sistema y flujos por rol.

---

## Índice

1. [Visión general](#1-visión-general)
2. [Roles del sistema](#2-roles-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Reglas de negocio](#4-reglas-de-negocio)
5. [Tabla de permisos por rol](#5-tabla-de-permisos-por-rol)
6. [Requisitos no funcionales](#6-requisitos-no-funcionales)
7. [Estados del sistema](#7-estados-del-sistema)
8. [Flujos por rol](#8-flujos-por-rol)

---

## 1. Visión general

La web del hotel permite a visitantes conocer las instalaciones, a clientes reservar y dejar reseñas, a recepcionistas gestionar la operación diaria, y a administradores supervisar y controlar todo el sistema.

El sistema se organiza en cuatro roles con permisos jerárquicos:

**Visitante → Cliente → Recepcionista → Administrador**

Cada rol hereda las capacidades de los roles inferiores y agrega las propias.

---

## 2. Roles del sistema

| Rol | Descripción |
|---|---|
| **Visitante** | Cualquier persona que entra a la web sin autenticarse. Ve la página principal, fotos de habitaciones y reseñas aprobadas. Puede registrarse o iniciar sesión. |
| **Cliente** | Usuario registrado. Puede reservar, ver sus reservaciones y dejar feedback. |
| **Recepcionista** | Operador básico. Gestiona check-in, check-out, pagos, habitaciones y reportes de guardia. |
| **Administrador** | Cargo superior. Supervisa, controla, audita y gestiona a los recepcionistas, habitaciones y reservaciones. |

---

## 3. Modelo de datos

### 3.1 Entidades principales

#### `Usuario`
Representa a cualquier persona que inicia sesión (Cliente, Recepcionista, Administrador).

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | Identificador único |
| nombre_apellidos | VARCHAR(120) | Nombre completo |
| telefono | VARCHAR(20) | Número de contacto |
| cedula | VARCHAR(20) | Única, identidad |
| sexo | ENUM(M, F, Otro) | Sexo |
| ciudad | VARCHAR(80) | Ciudad de residencia |
| correo | VARCHAR(120) | Único, usado para login |
| contrasena_hash | VARCHAR(255) | Nunca en texto plano |
| rol | ENUM(cliente, recepcionista, administrador) | Rol del usuario |
| activo | BOOLEAN | Para deshabilitar cuentas |
| creado_en | TIMESTAMP | Fecha de registro |
| actualizado_en | TIMESTAMP | Última modificación |

**Relaciones:**
- Un `Usuario` (cliente) → **N** `Reservacion`
- Un `Usuario` (cliente) → **N** `Feedback`
- Un `Usuario` (recepcionista) → **N** `ReporteGuardia`
- Un `Usuario` (cualquiera) → **N** `Notificacion`

#### `Habitacion`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | Identificador único |
| numero | VARCHAR(10) | Número/código de habitación |
| tipo | ENUM(simple, doble, suite, familiar…) | Categoría |
| descripcion | TEXT | Detalle de lo que incluye |
| precio_noche | DECIMAL(10,2) | Precio por noche |
| capacidad | INT | Personas máximas |
| estado | ENUM(vacante, ocupado, por_limpiar, reparacion, inoperativa) | Estado operativo |
| creado_en | TIMESTAMP | |
| actualizado_en | TIMESTAMP | |

**Relaciones:**
- Una `Habitacion` → **N** `HabitacionFoto`
- Una `Habitacion` → **N** `Reservacion`
- Una `Habitacion` → **N** `HistorialEstado`

#### `HabitacionFoto`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| habitacion_id | FK → Habitacion | |
| url | VARCHAR(255) | Ruta de la imagen |
| orden | INT | Para mostrar en carrusel |
| creado_en | TIMESTAMP | |

#### `Reservacion`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| cliente_id | FK → Usuario (nullable si la hizo recepcionista) | |
| recepcionista_id | FK → Usuario (nullable si la hizo el cliente) | |
| habitacion_id | FK → Habitacion | |
| fecha_checkin | DATE | |
| fecha_checkout | DATE | |
| noches | INT | Calculado |
| precio_total | DECIMAL(10,2) | |
| metodo_pago | ENUM(distancia, ingreso, estadia, egreso) | |
| estado | ENUM(pendiente, confirmada, en_curso, finalizada, cancelada) | |
| motivo_cancelacion | TEXT | Nullable |
| cancelada_por | FK → Usuario | Nullable |
| creado_en | TIMESTAMP | |
| actualizado_en | TIMESTAMP | |

**Relaciones:**
- Una `Reservacion` → **1** `Habitacion`
- Una `Reservacion` → **1** `Cliente`
- Una `Reservacion` → **N** `Pago`
- Una `Reservacion` → **1** `Factura`
- Una `Reservacion` → **1** `Feedback` (opcional)
- Una `Reservacion` → **N** `HistorialModificacion`

#### `Pago`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| reservacion_id | FK → Reservacion | |
| monto | DECIMAL(10,2) | |
| metodo | ENUM(efectivo, tarjeta, transferencia, otro) | |
| estado | ENUM(pendiente, parcial, pagado, reembolsado) | |
| fecha_pago | TIMESTAMP | |
| registrado_por | FK → Usuario | Quién verificó el pago |

#### `Factura`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| reservacion_id | FK → Reservacion | |
| numero_factura | VARCHAR(30) | Único |
| subtotal | DECIMAL(10,2) | |
| impuestos | DECIMAL(10,2) | |
| total | DECIMAL(10,2) | |
| emitida_por | FK → Usuario | Recepcionista |
| fecha_emision | TIMESTAMP | |

#### `Feedback`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| cliente_id | FK → Usuario | |
| reservacion_id | FK → Reservacion | Valida que sí se hospedó |
| comentario | TEXT | |
| calificacion | INT (1–5) | Opcional |
| aprobado | BOOLEAN | Moderación del admin |
| creado_en | TIMESTAMP | |

**Relaciones:**
- Un `Feedback` → **N** `FeedbackFoto`

#### `FeedbackFoto`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| feedback_id | FK → Feedback | |
| url | VARCHAR(255) | |

#### `ReporteGuardia`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| recepcionista_id | FK → Usuario | |
| turno | ENUM(manana, tarde, noche) | |
| contenido | TEXT | Novedades de la guardia |
| creado_en | TIMESTAMP | |

#### `Notificacion`

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| autor_id | FK → Usuario | Recepcionista o admin |
| tipo | ENUM(reporte_guardia, aviso_global) | |
| titulo | VARCHAR(120) | |
| contenido | TEXT | |
| alcance | ENUM(todos, recepcionistas, administradores) | |
| leida | BOOLEAN | |
| creado_en | TIMESTAMP | |

#### `HistorialEstado` (auditoría de habitaciones)

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| habitacion_id | FK → Habitacion | |
| estado_anterior | ENUM | |
| estado_nuevo | ENUM | |
| cambiado_por | FK → Usuario | |
| creado_en | TIMESTAMP | |

#### `HistorialModificacion` (auditoría de reservaciones)

| Campo | Tipo | Descripción |
|---|---|---|
| id | UUID / INT | |
| reservacion_id | FK → Reservacion | |
| campo_modificado | VARCHAR(50) | |
| valor_anterior | TEXT | |
| valor_nuevo | TEXT | |
| modificado_por | FK → Usuario | |
| creado_en | TIMESTAMP | |

### 3.2 Diagrama textual de relaciones

```
Usuario (cliente) 1───N Reservacion N───1 Habitacion 1───N HabitacionFoto
Reservacion 1───N Pago
Reservacion 1───1 Factura
Reservacion 1───1 Feedback 1───N FeedbackFoto
Usuario (recepcionista) 1───N ReporteGuardia
Usuario 1───N Notificacion
Habitacion 1───N HistorialEstado
Reservacion 1───N HistorialModificacion
```

---

## 4. Reglas de negocio

Redactadas en formato **"Dado X, cuando Y, entonces Z"** para que sean verificables y programables.

### 4.1 Registro y autenticación

- **RN-01**: Dado un visitante, cuando se registra, entonces se validan: correo único, cédula única, contraseña mínima de 8 caracteres con al menos 1 mayúscula y 1 número.
- **RN-02**: Dado un usuario, cuando falla el login, entonces se bloquea el intento por 10 segundos. Tras 5 fallos consecutivos, se bloquea la cuenta por 15 minutos.
- **RN-03**: Dado un usuario que olvidó su contraseña, cuando solicita recuperación, entonces recibe un código de 6 dígitos válido por 10 minutos, con máximo 3 intentos.

### 4.2 Reservaciones

- **RN-04**: Dado un cliente, cuando crea una reservación, entonces la habitación debe estar en estado `vacante` en las fechas seleccionadas.
- **RN-05**: Dado un cliente, cuando modifica sus datos personales en la reservación, entonces solo se actualizan los datos de esa reservación, no los de su cuenta.
- **RN-06**: Dado un cliente, cuando selecciona método de pago `distancia`, entonces debe pagar al menos el 50 % antes del check-in.
- **RN-07**: Dado un cliente, cuando selecciona método `ingreso`, `estadia` o `egreso`, entonces el pago se registra pero la reservación queda en `confirmada` hasta que el recepcionista verifique el pago.

### 4.3 Cancelaciones

- **RN-08**: Dado un administrador, cuando quiere cancelar una reservación, entonces solo puede hacerlo si faltan **más de 24 horas** para el check-in y si el cliente está de acuerdo (registrado en el sistema).
- **RN-09**: Dado un administrador que cancela, entonces se registra `motivo_cancelacion`, `cancelada_por` y se libera la habitación.

### 4.4 Check-in / Check-out

- **RN-10**: Dado un recepcionista, cuando registra un huésped, entonces debe verificar cédula y asignar habitación `vacante`.
- **RN-11**: Dado un recepcionista, cuando egresa a un huésped, entonces **no puede** completar el egreso si el pago no está en estado `pagado`.
- **RN-12**: Dado un egreso exitoso, entonces se genera la `Factura` automáticamente y se libera la habitación (estado `por_limpiar`).

### 4.5 Reasignación y ampliación

- **RN-13**: Dado un recepcionista, cuando reasigna una habitación, entonces la nueva debe estar `vacante` y, si su precio es mayor, se genera un `Pago` adicional por la diferencia.
- **RN-14**: Dado un recepcionista, cuando amplía noches, entonces se recalcula `precio_total` y se genera un `Pago` por la diferencia si aplica.
- **RN-15**: Toda reasignación o ampliación debe registrarse en `HistorialModificacion`.

### 4.6 Habitaciones

- **RN-16**: Solo el administrador puede **crear, eliminar o cambiar precios** de habitaciones.
- **RN-17**: El recepcionista solo puede cambiar el **estado** de la habitación, y cada cambio queda en `HistorialEstado`.
- **RN-18**: Una habitación en estado `ocupado` no puede reservarse ni reasignarse.

### 4.7 Feedback

- **RN-19**: Dado un cliente, cuando envía feedback, entonces debe tener al menos una reservación `finalizada`.
- **RN-20**: Dado un administrador, cuando recibe feedback, entonces puede aprobarlo u ocultarlo. Solo los aprobados se muestran en la página principal.
- **RN-21**: Un cliente solo puede dejar **un feedback por reservación**.

### 4.8 Bandeja de actualizaciones

- **RN-22**: Dado un recepcionista, cuando cierra su guardia, entonces **debe** registrar un `ReporteGuardia` para poder cerrar sesión.
- **RN-23**: Dado un administrador, cuando publica una notificación global, entonces todos los usuarios con rol recepcionista y administrador la reciben.
- **RN-24**: Toda notificación se marca como leída individualmente por usuario.

---

## 5. Tabla de permisos por rol

| Acción | Visitante | Cliente | Recepcionista | Administrador |
|---|---|---|---|---|
| Ver página principal y feedbacks aprobados | ✅ | ✅ | ✅ | ✅ |
| Ver fotos/descripción de habitaciones (público) | ✅ | ✅ | ✅ | ✅ |
| Registrarse | ✅ | — | — | — |
| Iniciar sesión | ✅ | ✅ | ✅ | ✅ |
| Recuperar contraseña | ✅ | ✅ | ✅ | ✅ |
| Hacer reservación | ❌ | ✅ | ✅ (a nombre de huésped) | ✅ |
| Ver sus propias reservaciones | ❌ | ✅ | — | — |
| Enviar feedback | ❌ | ✅ | ❌ | ❌ |
| Moderar feedback | ❌ | ❌ | ❌ | ✅ |
| Registrar huésped (check-in) | ❌ | ❌ | ✅ | ✅ |
| Egresar huésped (check-out) | ❌ | ❌ | ✅ | ✅ |
| Verificar pagos | ❌ | ❌ | ✅ | ✅ |
| Emitir factura | ❌ | ❌ | ✅ | ✅ |
| Modificar datos de huésped hospedado | ❌ | ❌ | ✅ | ✅ |
| Cambiar estado de habitación | ❌ | ❌ | ✅ | ✅ |
| Editar descripción/fotos/precio de habitación | ❌ | ❌ | ❌ | ✅ |
| Crear/eliminar habitaciones | ❌ | ❌ | ❌ | ✅ |
| Cancelar reservaciones | ❌ | ❌ | ❌ | ✅ |
| Ver reportes de ingresos (diario/semanal/mensual) | ❌ | ❌ | ❌ | ✅ |
| Ver habitaciones más usadas | ❌ | ❌ | ❌ | ✅ |
| Publicar reporte de guardia | ❌ | ❌ | ✅ | ✅ |
| Publicar aviso global | ❌ | ❌ | ❌ | ✅ |
| Leer bandeja de actualizaciones | ❌ | ❌ | ✅ | ✅ |
| Gestionar cuentas de recepcionistas | ❌ | ❌ | ❌ | ✅ |
| Auditar historial de cambios | ❌ | ❌ | ❌ | ✅ |

---

## 6. Requisitos no funcionales

### 6.1 Técnicos

- **RNF-01 (Stack sugerido)**: Frontend en React/Next.js o Vue; backend en Node.js (Express/Nest) o Laravel; base de datos PostgreSQL o MySQL.
- **RNF-02 (Responsive)**: La web debe funcionar correctamente en móvil, tablet y escritorio (mobile-first).
- **RNF-03 (Idioma)**: Interfaz completamente en español.
- **RNF-04 (Moneda)**: Definir moneda base (ej. USD o VES) y mostrar formato local.
- **RNF-05 (Zona horaria)**: Toda reserva se registra en la zona horaria del hotel (ej. America/Caracas).
- **RNF-06 (Seguridad)**: Contraseñas con hash bcrypt/argon2; comunicación HTTPS; tokens JWT con expiración; protección contra CSRF y XSS.
- **RNF-07 (Roles y permisos)**: Middleware de autorización por rol en cada endpoint.
- **RNF-08 (Auditoría)**: Registrar en `HistorialEstado` e `HistorialModificacion` todos los cambios críticos.
- **RNF-09 (Copias de seguridad)**: Backup diario de la base de datos.
- **RNF-10 (Rendimiento)**: Carga de la página principal en menos de 2 s en conexión 4G.

### 6.2 Funcionales / experiencia

- **RNF-11 (Notificaciones)**: Enviar correo al cliente al confirmar, modificar o cancelar una reservación. Recordatorio 24 h antes del check-in.
- **RNF-12 (Recuperación de contraseña)**: Código de 6 dígitos por correo, válido 10 minutos.
- **RNF-13 (Bloqueo de login)**: 10 s de espera por intento fallido; 15 min tras 5 intentos.
- **RNF-14 (Accesibilidad)**: Contraste adecuado, navegación por teclado, textos alternativos en imágenes.
- **RNF-15 (Manejo de errores)**: Mensajes claros al usuario; logs internos para el equipo.

### 6.3 Operativos

- **RNF-16 (Despliegue)**: Contenedores Docker; variables de entorno para credenciales.
- **RNF-17 (Escalabilidad)**: Diseño que permita agregar múltiples hoteles en el futuro (multi-tenant opcional).
- **RNF-18 (Mantenibilidad)**: Código modular, con documentación de endpoints (Swagger/OpenAPI).

---

## 7. Estados del sistema

### 7.1 Estados de una Reservación

```
pendiente → confirmada → en_curso → finalizada
     ↓            ↓           ↓
  cancelada   cancelada   cancelada (solo admin, con reglas)
```

| Estado | Significado |
|---|---|
| `pendiente` | Creada pero sin pago verificado |
| `confirmada` | Pago verificado o método acordado, habitación reservada |
| `en_curso` | Cliente ya hizo check-in |
| `finalizada` | Cliente hizo check-out y factura emitida |
| `cancelada` | Cancelada por admin con reglas cumplidas |

### 7.2 Estados de un Pago

| Estado | Significado |
|---|---|
| `pendiente` | Aún no se ha recibido nada |
| `parcial` | Se recibió parte del monto |
| `pagado` | Monto completo recibido |
| `reembolsado` | Devuelto al cliente tras cancelación |

### 7.3 Estados de una Habitación

| Estado | Significado | ¿Reservable? |
|---|---|---|
| `vacante` | Disponible | ✅ |
| `ocupado` | Con huésped | ❌ |
| `por_limpiar` | Requiere limpieza | ❌ |
| `reparacion` | En mantenimiento | ❌ |
| `inoperativa` | Fuera de servicio indefinido | ❌ |

### 7.4 Transiciones válidas de habitación

```
vacante ⇄ ocupado
vacante → por_limpiar → vacante
vacante → reparacion → vacante
cualquiera → inoperativa → (solo admin la reactiva)
```

### 7.5 Estados de un Usuario

| Estado | Significado |
|---|---|
| `activo` | Puede iniciar sesión |
| `bloqueado` | Bloqueo temporal por intentos fallidos |
| `inactivo` | Deshabilitado por admin |

### 7.6 Estados de una Notificación

| Estado | Significado |
|---|---|
| `no_leida` | El usuario aún no la abre |
| `leida` | El usuario la abrió |

---

## 8. Flujos por rol

### 8.1 Visitante

Todo aquel que ingrese a la página principal de la web tendrá acceso a:

- Capturas (imágenes) de las habitaciones.
- Reseñas de huéspedes anteriores (solo feedbacks aprobados).
- Acceso para registrarse e ingresar a todas las funciones como Cliente.

---

### 8.2 Inicio de sesión

Los Visitantes, para ingresar a la página como Clientes, necesitarán ingresar los siguientes datos:

1. Nombre y Apellidos
2. Número de Teléfono
3. Cédula de Identidad
4. Sexo
5. Ciudad
6. Correo
7. Contraseña

> **Nota:** Este bloque corresponde al **registro**. El inicio de sesión posterior solo requiere **correo** y **contraseña**.

#### Recuperación de contraseña

El inicio de sesión tiene la opción de recuperar la contraseña:

1. El usuario ingresa su correo de recuperación.
2. El sistema envía un código de 6 dígitos.
3. El usuario ingresa el código en la página.
4. Tras verificarlo, accede a un menú para ingresar una nueva contraseña.

#### Seguridad

En cada intento erróneo de contraseña, se deberá esperar **10 segundos** antes de ingresar de nuevo la contraseña. Tras 5 intentos fallidos consecutivos, la cuenta se bloquea por **15 minutos**.

---

### 8.3 Cliente

Luego de ingresar, los Clientes tendrán acceso a las siguientes funciones:

#### Hacer Reservaciones

El cliente tiene acceso a poder hacer reservaciones para el hotel. Para hacer esa solicitud se deberán llenar los siguientes campos:

> Se necesitarán los primeros 5 campos del inicio de sesión, se llenarán los datos automáticamente con los datos principales de la cuenta. El cliente tendrá la posibilidad de modificarlos si lo desea.

1. **Habitación:** Se le mostrarán las habitaciones disponibles en el hotel, con fotos y una descripción de lo que cuenta la habitación.
2. **Tiempo de hospedaje:** El cliente seleccionará el tiempo que se quedará hospedado dentro del hotel.
3. **Método de pago:** Puede seleccionar si pagar a distancia, al momento de ingresar, en plena estadía o al momento de egresar del hotel. (El recepcionista deberá verificar el pago antes de que el huésped salga de las instalaciones del hotel).

#### Feedback

El cliente tiene acceso a dar recomendaciones (feedback), comentando qué tal su experiencia en su tiempo de estadía en el hotel, con la posibilidad de agregar fotos si lo desea.

---

### 8.4 Recepcionistas

Los recepcionistas, como operadores básicos de la web, tienen acceso a las siguientes funciones:

#### Ingresar Nuevos Huéspedes

Se emplearía una función similar a la de hacer reservaciones, solo que esta vez el recepcionista deberá ingresar todos los datos del huésped, que serían:

1. Nombre y Apellidos
2. Número de Teléfono
3. Cédula de Identidad
4. Sexo
5. Ciudad
6. **Habitación:** Se le mostrarán las habitaciones disponibles en el hotel, con fotos y una descripción de lo que cuenta la habitación.
7. **Tiempo de hospedaje:** El recepcionista seleccionará el tiempo que se quedará hospedado el huésped dentro del hotel.
8. **Método de pago:** Puede seleccionar si el huésped pagará al momento de ingresar, en plena estadía o al momento de egresar del hotel.

#### Egresar Huéspedes

Se encarga de verificar que el pago esté cancelado y el tiempo de estadía asignado. Posteriormente se le haría entrega de factura por estadía en el hotel.

#### Modificar

**1. Huéspedes:** El recepcionista tiene acceso a modificar los datos del huésped ya estando dentro de la habitación, como:

1. Nombre y Apellidos
2. Número de Teléfono
3. Cédula de Identidad
4. Sexo
5. Ciudad
6. **Habitación:** Por si se hace una reasignación de habitación (si hay aumento de precio por habitación, deberá cancelar la diferencia).
7. **Tiempo de hospedaje:** Por si desea agregar noches de estadía (si hay aumento de precio por habitación, deberá cancelar la diferencia).

**2. Habitaciones:** El recepcionista puede modificar el **estado** de la habitación:

1. Vacante
2. Ocupado
3. Por Limpiar
4. Reparación
5. Inoperativa

> **Nota:** La edición de descripción, fotos y precios queda reservada al Administrador (ver RN-16 y RN-17).

#### Bandeja de Actualizaciones

Los recepcionistas, al final de guardia, deberán hacer un reporte de novedades durante su guardia. Este reporte aparecerá en la bandeja de actualizaciones de todos los trabajadores (otros recepcionistas y administradores).

---

### 8.5 Administradores

> "El cargo superior en la jerarquía del manejo de la web del hotel."

El Administrador contará con gestión, control e información de:

#### 1. Recepcionistas

- Supervisará que se hagan los pagos de las habitaciones.
- Que la asignación de las habitaciones sea correcta.
- Que los datos de los huéspedes concuerden (en habitaciones, facturas, etc.).
- Supervisará el estado actual de las habitaciones.
- Podrá ingresar y modificar cualquier dato del recepcionista.

#### 2. Bandeja de Actualizaciones

Los administradores tendrán acceso a leer las novedades de los recepcionistas, pero ellos tendrán la opción de notificar novedades globales (cambios en algún campo del hotel, llamado de atención general por faltas, agradecimiento por trabajos bien hechos en la semana, etc.), todo con la función de aplicar una comunicación eficiente y general en el grupo de trabajo.

#### 3. Cancelar Reservaciones

Los administradores tienen acceso a eliminar cualquier reservación con la condición de que:

- El huésped esté de acuerdo.
- El tiempo de ingreso del huésped sea después de un día (es decir, que falten **más de 24 horas** para el check-in). Allí la reservación tiene permiso a ser eliminada.

#### 4. Visualizaciones

El Administrador tiene permiso para visualizar:

- Datos completos de las habitaciones.
- Habitaciones más usadas.
- Ingreso diario, semanal, mensual.

#### 5. Habitaciones

Los administradores tienen acceso a modificar y tener control de las habitaciones, cambiando descripciones, capturas, precios, etc.

---

## Anexo A: Glosario

| Término | Definición |
|---|---|
| **Visitante** | Persona no autenticada que navega la web. |
| **Cliente** | Usuario registrado que puede reservar y dejar feedback. |
| **Huésped** | Persona que se hospeda en el hotel (puede o no ser Cliente registrado). |
| **Check-in** | Registro de entrada del huésped. |
| **Check-out** | Registro de salida del huésped y emisión de factura. |
| **Feedback** | Reseña o comentario del cliente sobre su estadía. |
| **Guardia** | Turno de trabajo de un recepcionista (mañana, tarde, noche). |
| **Bandeja de actualizaciones** | Espacio donde se publican reportes de guardia y avisos globales. |

---

## Anexo B: Convenciones del documento

- **RN-XX**: Regla de negocio numerada.
- **RNF-XX**: Requisito no funcional numerado.
- Los estados del sistema se escriben en `snake_case` y en minúsculas.
- Los nombres de tablas y campos en el modelo de datos se escriben en `snake_case`.
- Los roles se escriben en minúsculas: `cliente`, `recepcionista`, `administrador`.