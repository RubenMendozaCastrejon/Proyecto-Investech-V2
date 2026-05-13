Este es el inicio de un **Plan de Implementación Maestro** para la plataforma Investech. Aunque el diccionario de datos original está estructurado para SQL, adaptaremos la arquitectura para un entorno NoSQL en **Firebase (Cloud Firestore)**, manteniendo la integridad de las relaciones mediante referencias y duplicación estratégica (denormalización).

---

## Parte 1: Cimentación, Arquitectura y Core de Identidad

Esta fase se centra en establecer el entorno de desarrollo, la estructura de la base de datos en Firebase y el sistema de autenticación, que es la puerta de entrada para la **Identidad y Acceso**.

### 1.1 Configuración del Ecosistema de Desarrollo

Antes de escribir código, debemos asegurar que el entorno soporte una aplicación de grado financiero.

| Paso | Actividad | Descripción Técnica |
| --- | --- | --- |
| **1.1.1** | **Provisionamiento de Firebase** | Crear proyecto en Console, habilitar Google Auth, Email Auth y configurar `google-services.json`. |
| **1.1.2** | **Inicialización de Flutter** | Crear proyecto con `flutter create --org com.investech.app --platforms android`. |
| **1.1.3** | **Setup de Flavors** | Configurar ambientes `development`, `staging` y `production` para no tocar datos reales en pruebas. |
| **1.1.4** | **Estructura de Carpetas (Clean Architecture)** | `lib/features/`, `lib/core/`, `lib/data/`, `lib/domain/`. |

---

### 1.2 Modelado de Datos: De SQL a Cloud Firestore

Transformaremos las entidades del diccionario de datos en una estructura de documentos y colecciones.

| Entidad Original | Tipo en Firebase | Relación / Estrategia |
| --- | --- | --- |
| **1 USUARIO** | Colección Raíz `users` | Documento por `uid` de Firebase Auth. |
| **3 DOCUMENTO** | Sub-colección `users/{uid}/documents` | Almacenamiento de metadatos de archivos KYC. |
| **4 PERFIL_RIESGO** | Documento `users/{uid}/risk_profile` | Mapeo de niveles (conservador, moderado, agresivo). |
| **2 SESION** | Firebase Auth State | Manejado automáticamente por el SDK de Firebase. |

> **Nota Pro:** Para cumplir con la entidad **USUARIO**, el campo `pais` debe validarse bajo la norma ISO 3166-1 alpha-2 como se especifica en el diccionario.

---

### 1.3 Implementación del Core de Autenticación (Identity & Access)

Basado en los requerimientos de **Estado** y **Verificación** del usuario.

1. **Capa de Dominio (Domain):**
* Definir `UserEntity` con atributos: `id`, `nombre`, `email`, `estado` (activo/suspendido), y `verificado_en`.
* Crear `AuthRepository` (Interface).


2. **Capa de Datos (Data):**
* Implementar `FirebaseAuthService`.
* Crear `UserModel` con métodos `fromFirestore` y `toFirestore` para manejar los tipos `TIMESTAMPTZ` de la DB original como `Timestamp` de Firebase.


3. **Capa de Presentación (Bloc/UI):**
* **AuthBloc:** Maneja estados `Uninitialized`, `Authenticated`, `Unauthenticated`.
* **Registro KYC:** Flujo para capturar `nombre`, `email`, `telefono` y `pais`.



---

### 1.4 Diseño e Implementación del Módulo KYC (Documentos)

La entidad **DOCUMENTO** es crítica para una app de inversiones.

* **Integración con Firebase Storage:**
* Los archivos físicos (INE, Pasaporte) se suben a `gs://investech-app/kyc/{uid}/`.
* La URL resultante se guarda en el documento de la sub-colección `documents` con el `estado` inicial como "pendiente".


* **Validación de tipos:** Implementar un selector que restrinja los tipos a: `ine`, `pasaporte`, `comprobante`, `otro`.

---

### 1.5 Evaluación de Perfil de Riesgo

Implementación de la entidad **PERFIL_RIESGO**.

* **Cuestionario Dinámico:** Widget que recorre un JSON de preguntas y calcula el `puntaje` (0–100).
* **Lógica de Clasificación:**
* Puntaje < 40: **Conservador**
* Puntaje 40-70: **Moderado**
* Puntaje > 70: **Agresivo**

* **Persistencia:** Guardar en Firestore con el campo `vigente: true` y la fecha de evaluación `evaluado_en`.

Aquí tienes la **Parte 2** del Plan de Implementación Maestro para **Investech**. Esta fase es el núcleo financiero del sistema, donde transformamos la estructura de **Cuentas y Dinero** del diccionario de datos en una arquitectura funcional de billetera digital y registro transaccional.

---

## Parte 2: Arquitectura de Cuentas, Monederos y Transacciones

Esta fase se enfoca en la gestión de fondos, la conversión de tipos de moneda (ISO 4217) y el motor de movimientos financieros, asegurando que el saldo disponible y el saldo bloqueado se sincronicen correctamente.

### 2.1 Modelado de la Entidad CUENTA en Firestore

A diferencia de SQL, en Firebase utilizaremos una estructura de sub-colecciones para mantener la seguridad y el aislamiento de datos por usuario.

| Campo (Atributo) | Tipo Firestore | Lógica de Negocio |
| --- | --- | --- |
| **id** | String (DocID) | Generado automáticamente por Firestore. |
| **usuario_id** | DocumentReference | Referencia al documento en la colección `users`. |
| **numero** | String | Generado con un algoritmo de máscara (ej. INV-XXXX). |
| **tipo** | String (Enum) | `efectivo`, `margen` o `custodia`. |
| **moneda** | String | Código ISO 4217 (MXN, USD). |
| **saldo** | Number (Double) | Fondos líquidos disponibles para operar. |
| **saldo_bloqueado** | Number (Double) | Fondos retenidos por órdenes pendientes. |
| **estado** | String (Enum) | `activa`, `suspendida` o `cerrada`. |

---

### 2.2 Implementación del Motor de Transacciones (TRANSACCION)

Para garantizar la integridad financiera (evitar que un usuario gaste dinero que no tiene), utilizaremos **Cloud Functions** y **Transacciones Atómicas** de Firestore.

1. **Registro de Movimientos:**
* Cada vez que ocurre un `deposito`, `retiro`, `compra` o `venta`, se crea un documento en la colección `accounts/{accountId}/transactions`.
* Se debe capturar la `referencia` bancaria para depósitos externos.


2. **Lógica de Atomicidad (Crucial):**
* No se debe actualizar el saldo desde la app móvil directamente.
* Se crea un "Trigger" en Firebase que, al detectar una nueva transacción con estado `completada`, actualiza el campo `saldo` en el documento de la `CUENTA` correspondiente.


3. **Manejo de estados:**
* Implementar lógica para transacciones `revertidas` (ej. si un retiro es rechazado por el banco).



---

### 2.3 UI/UX: El Dashboard Financiero (Wallet)

Desarrollo de la interfaz de usuario basada en Flutter para la gestión de fondos.

| Widget Flutter | Componente Relacionado | Función |
| --- | --- | --- |
| **BalanceCard** | `CUENTA.saldo` | Muestra el balance total y el saldo bloqueado de forma visual. |
| **CurrencySelector** | `CUENTA.moneda` | Permite al usuario filtrar sus cuentas por divisa. |
| **TransactionList** | `TRANSACCION` | Lista infinita (Scroll) con iconos diferenciados por tipo (flecha arriba para depósitos, abajo para retiros). |
| **ActionButtons** | Operativa | Botones rápidos para "Fondear" (Depósito) y "Retirar". |

---

### 2.4 Seguridad y Reglas de Firestore

Es vital proteger la entidad **TRANSACCION** para evitar manipulaciones malintencionadas.

* **Regla de Solo Lectura:** El usuario puede leer sus transacciones pero **nunca** editarlas o borrarlas.
* **Validación de Saldo:** La regla de escritura en la sub-colección de transacciones debe validar que, si el tipo es `retiro` o `compra`, el `monto` no exceda el `saldo` actual de la cuenta.

---

### 2.5 Gestión de Saldo Bloqueado

Implementación de la lógica para la entidad **ORDEN** (que se detallará más en la Parte 3, pero afecta el dinero aquí).

* Cuando un usuario coloca una orden de compra `limite`, el sistema debe:
1. Calcular el monto necesario: `cantidad * precio_limite`.
2. Mover ese monto del campo `saldo` al campo `saldo_bloqueado`.
3. Esto asegura que el usuario no pueda retirar dinero que ya está comprometido en el mercado.



---

Esta es la **Parte 3** del Plan de Implementación Maestro para **Investech**. Entramos en el motor central del negocio: la **Operativa de Mercado**. Aquí es donde conectamos los activos financieros (instrumentos) con las intenciones de los usuarios (órdenes) y la realidad del mercado (ejecuciones y precios).

---

## Parte 3: Operativa de Mercado e Instrumentos (Trading Engine)

En esta fase, configuramos el catálogo de activos y el sistema de mensajería para las órdenes de compra y venta.

### 3.1 Catálogo Global de Activos (MERCADO e INSTRUMENTO)

A diferencia de las cuentas, estas colecciones son **públicas (solo lectura)** para todos los usuarios.

| Colección Firestore | Estrategia de Datos | Atributos Clave |
| --- | --- | --- |
| **markets** | Documentos por Código MIC. | `codigo` (BMV, NYSE), `zona_horaria`, `moneda_base`. |
| **instruments** | Sub-colección de `markets` o colección raíz con `market_id`. | `ticker` (AMZN, TSLA), `tipo` (accion, etf, cripto), `lote_minimo`, `isin`. |

* **Sincronización de Precios:** Implementar la entidad **PRECIO_HISTORICO** mediante una integración con una API externa (como Alpha Vantage o Bloomberg) que alimente una sub-colección `prices` con datos de apertura, máximo, mínimo y cierre.

---

### 3.2 Flujo de Ciclo de Vida de una ORDEN

La entidad **ORDEN** requiere un manejo de estados riguroso para evitar errores en la ejecución financiera.

1. **Creación (Draft/Pending):** El usuario selecciona el `instrumento_id`, define el `lado` (compra/venta) y el `tipo` (mercado/limite).
2. **Validación de Margen:** Una *Cloud Function* verifica que el usuario tenga saldo suficiente (si es compra) o títulos suficientes (si es venta).
3. **Estado Activa:** La orden se marca como `activa` y se envía al "Order Book" simulado o real.
4. **Ejecución (Fills):** Cuando el precio de mercado coincide con el `precio_limite`, se genera la entidad **EJECUCION**.

---

### 3.3 Implementación de la Entidad EJECUCION

La tabla **EJECUCION** es la prueba de que una orden se completó parcial o totalmente.

* **Cálculo de Comisión:** En cada ejecución, se debe calcular y registrar la `comision` cobrada por Investech.
* **Actualización de Orden:** Si la `cantidad` ejecutada es igual a la solicitada en la orden original, el estado de la **ORDEN** cambia a `ejecutada`.
* **Trigger de Liquidación:** Al crearse una ejecución, se debe disparar automáticamente una **TRANSACCION** de tipo `compra` o `venta` para reflejar el movimiento de dinero en la cuenta.

---

### 3.4 UI/UX: Terminal de Trading en Flutter

Desarrollo de la interfaz para la toma de decisiones y envío de instrucciones.

| Componente Widget | Datos Relacionados | Funcionalidad |
| --- | --- | --- |
| **CandlestickChart** | `PRECIO_HISTORICO` | Gráfica de velas japonesas usando `fl_chart` con datos OHLCV. |
| **OrderForm** | `ORDEN` | Formulario dinámico que habilita/deshabilita campos según el tipo (ej. oculta `precio_limite` si es a `mercado`). |
| **OrderBook** | Datos en Tiempo Real | Muestra las puntas de compra y venta (Bids/Asks). |
| **VigenciaSelector** | `ORDEN.vigencia` | Selector para `dia`, `gtc` (Good 'til Cancelled), `ioc`, etc. |

---

### 3.5 Real-time Tickers (StreamBuilder)

Para que Investech se sienta como una plataforma profesional, los precios no pueden ser estáticos.

* **Firestore Snapshots:** Utilizar `StreamBuilder` en Flutter para escuchar cambios en el documento del instrumento.
* **Optimización de Costos:** No actualizar cada segundo en Firestore; usar un WebSocket para el precio "en vivo" y solo persistir en la base de datos los cierres de velas o cambios significativos.

---

Esta es la **Parte 4** y final del Plan de Implementación Maestro para **Investech**. Aquí cerramos el círculo operativo enfocándonos en la **Gestión de Portafolios**, la visualización de la **Posición** del usuario y el cálculo de métricas de desempeño (ganancias y pérdidas), transformando datos crudos en información estratégica para el inversionista.

---

## Parte 4: Portafolios, Posiciones y Análisis de Rendimiento

Esta fase se centra en consolidar las ejecuciones de mercado en una vista de patrimonio neto y rendimiento histórico.

### 4.1 Consolidación de la Entidad PORTAFOLIO

El **PORTAFOLIO** actúa como el contenedor lógico de las inversiones vinculadas a una cuenta.

| Atributo | Implementación en Firebase | Lógica de Actualización |
| --- | --- | --- |
| **valor_total** | Campo calculado (Number) | Suma de `valor_mercado` de todas las posiciones + saldo en efectivo de la cuenta. |
| **rendimiento_pct** | Campo calculado (Percentage) | Comparación entre el costo total de adquisición vs. el valor de mercado actual. |
| **actualizado_en** | Timestamp | Se dispara cada vez que el precio de un activo en cartera cambia. |

* **Estrategia NoSQL:** Para optimizar la lectura, el `valor_total` se actualiza mediante una **Cloud Function** que se activa cada vez que una `POSICION` cambia de valor.

---

### 4.2 Gestión Dinámica de la Entidad POSICION

La **POSICION** representa la tenencia real de un activo tras las ejecuciones de órdenes.

* **Cálculo de Precio Promedio:** Al ocurrir una `EJECUCION` de compra, el sistema debe recalcular el `precio_promedio` (costo ponderado):

$$Precio\_Promedio = \frac{(Cantidad\_Actual \times Precio\_Antiguo) + (Cantidad\_Nueva \times Precio\_Ejecucion)}{Cantidad\_Total}$$


* **Ganancia/Pérdida (P&L):** Se calcula en tiempo real para la entidad `ganancia_perdida` restando el costo promedio del valor de mercado actual.
* **Cierre de Posición:** Si la `cantidad` llega a cero tras una venta, el documento de la posición puede archivarse para mantener el histórico de rendimiento.

---

### 4.3 UI/UX: Visualización Avanzada y Reportes

La interfaz final de Investech debe ser intuitiva y profesional, utilizando los datos de **PORTAFOLIO** y **POSICION**.

| Widget Flutter | Datos Fuente | Descripción |
| --- | --- | --- |
| **PortfolioDonutChart** | `POSICION.valor_mercado` | Gráfico de dona que muestra la diversificación por tipo de activo (Acciones vs. Cripto). |
| **PerformanceGraph** | Histórico de `valor_total` | Gráfica de línea que muestra el crecimiento del patrimonio en el tiempo. |
| **HoldingsTable** | Listado de `POSICION` | Tabla con `ticker`, `cantidad`, `precio_promedio` y el P&L coloreado (verde/rojo). |

---

### 4.4 Reporteo y Cumplimiento (Compliance)

Basado en los dominios de **Identidad** y **Operativa**:

* **Generación de Estados de Cuenta:** Función que agrupa `TRANSACCION`, `EJECUCION` y `PORTAFOLIO` para generar un PDF mensual.
* **Resumen Fiscal:** Cálculo de utilidades realizadas basándose en las ventas del periodo.
* **Alertas de Riesgo:** Notificaciones push si el `rendimiento_pct` cae por debajo de un umbral definido por el `PERFIL_RIESGO` del usuario.

---

### 4.5 Cierre del Proyecto: Despliegue y Mantenimiento

1. **Seguridad Final:** Revisión de las *Firebase Security Rules* para asegurar que un usuario solo pueda leer su propio `PORTAFOLIO` y `POSICION`.
2. **Pruebas de Carga:** Simular múltiples órdenes simultáneas para asegurar que las **Cloud Functions** procesen las ejecuciones y actualicen los saldos sin colisiones.
3. **Monitoreo:** Implementar *Firebase Crashlytics* para detectar errores en la lógica de cálculo financiero en dispositivos Android.

---

### Resumen Final del Objetivo

Con estas 4 partes, has pasado de un diccionario de datos estático a una arquitectura robusta de Flutter + Firebase. Investech ahora cuenta con:

1. Identidad segura y procesos KYC.
2. Gestión de cuentas multimoneda y transacciones atómicas.
3. Operativa de mercado con órdenes y ejecuciones en tiempo real.
4. Control de portafolios con métricas de rendimiento avanzadas.

**¡El ecosistema de Investech está listo para producción!**
