# 📘 PARTE 1: CIMENTACIÓN ARQUITECTÓNICA, ENTORNO Y ESTRATEGIA DE INTEGRACIÓN (Semanas 1-3)

Esta primera parte establece los cimientos técnicos, de seguridad y de arquitectura necesarios para una aplicación FinTech de nivel institucional. Se aborda la configuración del proyecto Flutter, la estructura de capas, la estrategia de conexión segura entre Firebase y tu base de datos PostgreSQL, y los protocolos de cumplimiento normativo.

---

## 🔹 1.1. Inicialización del Proyecto y Estructura de Código

### 1.1.1. Configuración Base
```bash
flutter create investech_app \
  --org com.investech.mobile \
  --platforms android \
  --empty \
  --project-name investech
cd investech_app
flutter pub add flutter_bloc equatable dio get_it injectable go_router freezed json_annotation json_serializable hive flutter_secure_storage firebase_core firebase_auth firebase_storage firebase_crashlytics firebase_analytics flutter_dotenv envied intl cached_network_image shimmer lottie
flutter pub add --dev build_runner freezed_annotation injectable_generator json_serializable mocktail very_good_analysis bloc_test flutter_test
```

### 1.1.2. Estructura de Directorios (Feature-First + Clean Architecture)
| Nivel | Ruta | Propósito |
|-------|------|-----------|
| `lib/` | `app/` | Configuración global, routing, inyección de dependencias, temas |
| `lib/` | `core/` | Utilidades transversales: errores, red, constantes, seguridad, validadores |
| `lib/` | `features/` | Módulos funcionales alineados al dominio SQL (ver tabla 1.2) |
| `lib/` | `domain/` | Entidades, repositorios abstractos, casos de uso (regla de negocio pura) |
| `lib/` | `data/` | Implementaciones de repositorios, DTOs, mappers, servicios de red/local |
| `lib/` | `presentation/` | Screens, widgets reutilizables, BLoC/Cubit, state managers UI |

**📁 Estructura por Feature (Ejemplo: `features/auth/`)**
```
features/auth/
├── data/
│   ├── datasources/
│   │   ├── auth_remote_datasource.dart
│   │   └── auth_local_datasource.dart
│   ├── models/
│   │   └── user_model.dart (DTO + fromJson/toJson)
│   └── repositories/
│       └── auth_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── user_entity.dart
│   ├── repositories/
│   │   └── auth_repository.dart
│   └── usecases/
│       ├── login_usecase.dart
│       ├── register_usecase.dart
│       └── verify_kyc_usecase.dart
└── presentation/
    ├── bloc/
    │   ├── auth_bloc.dart
    │   ├── auth_event.dart
    │   └── auth_state.dart
    ├── screens/
    │   ├── login_screen.dart
    │   ├── register_screen.dart
    │   └── onboarding_screen.dart
    └── widgets/
        └── custom_auth_form.dart
```

---

## 🔹 1.2. Mapeo de Dominio SQL → Módulos Flutter

Tu esquema PostgreSQL define 5 dominios claros. Se estructuran como features independientes en Flutter:

| Dominio SQL | Tablas Asociadas | Módulo Flutter | Responsabilidad Principal |
|-------------|------------------|----------------|---------------------------|
| **Identidad & Acceso** | `USUARIO`, `SESION`, `DOCUMENTO`, `PERFIL_RIESGO` | `features/auth/`, `features/kyc/`, `features/profile/` | Registro, login, biometría, carga de INE/pasaporte, evaluación de riesgo |
| **Cuentas & Dinero** | `CUENTA`, `TRANSACCION` | `features/accounts/`, `features/wallet/` | Visualización de saldos, bloqueos, historial de movimientos, depósitos/retiros |
| **Mercados & Instrumentos** | `MERCADO`, `INSTRUMENTO`, `PRECIO_HISTORICO` | `features/markets/`, `features/instruments/` | Listado de activos, cotizaciones, filtros, gráficos OHLCV, búsqueda por ticker/ISIN |
| **Operativa de Trading** | `ORDEN`, `EJECUCION` | `features/trading/`, `features/orders/` | Creación de órdenes (mercado/limite/stop), estado en tiempo real, historial de fills |
| **Portafolios & Posiciones** | `PORTAFOLIO`, `POSICION` | `features/portfolio/`, `features/positions/` | Vista de holdings, P&L, rendimiento %, rebalanceo, métricas de diversificación |

---

## 🔹 1.3. Configuración de Entornos, Sabores y CI/CD

### 1.3.1. Flavores (Dev / Staging / Prod)
```yaml
# pubspec.yaml
flutter:
  flavors:
    dev:
      applicationId: com.investech.mobile.dev
      appIcon: "assets/icons/dev.png"
      appName: "Investech Dev"
    staging:
      applicationId: com.investech.mobile.stg
      appIcon: "assets/icons/stg.png"
      appName: "Investech Staging"
    prod:
      applicationId: com.investech.mobile
      appIcon: "assets/icons/prod.png"
      appName: "Investech"
```
Se utiliza `flutter_flavorizr` o `main_dev.dart`, `main_stg.dart`, `main_prod.dart` con `Envied` para cargar variables sensibles (API URLs, Firebase keys, SSL pins).

### 1.3.2. Pipeline CI/CD (GitHub Actions)
| Etapa | Acciones | Herramientas |
|-------|----------|--------------|
| **Lint & Format** | `dart analyze`, `dart format`, `custom_lint` | GitHub Actions `flutter-actions/setup-flutter` |
| **Test Unitario** | Ejecutar `flutter test --coverage` con >80% cobertura | `lcov`, `coveralls` |
| **Build APK/AAB** | Compilación optimizada (`--release --flavor prod`) | `gradlew`, `flutter build appbundle` |
| **Distribución** | Subida a Firebase App Distribution / Play Internal | `firebase appdistribution:distribute` |
| **Estático** | Generar reportes de seguridad, dependencias vulnerables | `dart pub outdated`, `snyk` |

---

## 🔹 1.4. Estrategia de Seguridad y Cumplimiento Normativo (FinTech)

Una aplicación de inversiones requiere protocolos institucionales desde el día 1:

| Capa | Implementación | Justificación |
|------|----------------|---------------|
| **Autenticación** | Firebase Auth + MFA (SMS/Email) + Biometría local (`local_auth`) | Cumple con PSD2/Strong Customer Authentication |
| **Almacenamiento Seguro** | `flutter_secure_storage` para tokens, claves de sesión | Android Keystore System protegido por hardware |
| **Red** | Certificate Pinning (`http_certificate_pinning` o `dio` plugin) | Previene MITM en redes públicas |
| **Logs** | `log` con máscaras automáticas (PII, saldos, cuentas) | Cumple GDPR/LGPD, evita fugas en Crashlytics |
| **KYC/AML** | Flujo de verificación de identidad con `DOCUMENTO` y `PERFIL_RIESGO` | Obligatoriedad regulatoria para plataformas de inversión |
| **Sesiones** | Rotación de tokens, timeout inactivo, revocación remota vía `SESION.revocada` | Mitiga riesgo de cuentas comprometidas |

**🔐 Flujo de Seguridad de Sesión (Alineado a tu esquema)**
1. Usuario inicia sesión → Firebase Auth devuelve ID Token.
2. App envía token a Cloud Function `/v1/auth/validate`.
3. CF verifica firma, consulta tabla `SESION` en PostgreSQL.
4. Si válida → devuelve `access_token` + `refresh_token` + metadata de cuenta.
5. Cada petición incluye `Authorization: Bearer <access_token>`.
6. En logout o revocación, CF actualiza `SESION.revocada = TRUE`.

---

## 🔹 1.5. Puente Firebase ↔ PostgreSQL: Diseño del BFF (Backend for Frontend)

⚠️ **Regla Crítica:** *Nunca conectar Flutter directamente a PostgreSQL desde el cliente móvil.* Expones credenciales, bypass de validación de negocio, riesgo de SQL injection, y falta de pool de conexiones.

### 🌉 Arquitectura de Integración
```
[Flutter App] 
   ↓ (HTTPS + JWT)
[Firebase Auth / Cloud Messaging / Remote Config]
   ↓ (Cloud Functions / Run)
[BFF API Gateway (Node.js/TypeScript o Python/FastAPI)]
   ↓ (Connection Pooling: PgBouncer)
[PostgreSQL (bdinvestech)]
```

### 📦 Stack del BFF (Cloud Functions / Cloud Run)
| Componente | Tecnología | Rol |
|------------|------------|-----|
| **Framework** | Express.js / Fastify / FastAPI | Routing, middleware, validación Zod/Pydantic |
| **ORM/Query** | Prisma / Kysely / SQLAlchemy | Mapeo seguro a tu esquema SQL, migraciones automáticas |
| **Pool Conexiones** | PgBouncer / Prisma Connection Pool | Escalabilidad a +10k conexiones concurrentes |
| **Validación** | JSON Schema + TypeBox | Garantiza que los DTOs coincidan con tus ENUMs y CHECKs |
| **WebSockets/SSE** | Socket.io / Server-Sent Events | Precios en tiempo real (`PRECIO_HISTORICO`), estado de órdenes (`ORDEN.estado`) |
| **Auditoría** | Middleware de logging inmutable | Registra IP, dispositivo, hora, acción (cumplimiento financiero) |

### 🔄 Mapeo DTO → Entidad SQL (Ejemplo: `CUENTA`)
```json
// DTO entrante (Flutter → BFF)
{
  "tipo_cuenta": "margen",
  "moneda": "MXN"
}
// BFF valida, aplica reglas de negocio, inserta en PostgreSQL
// Respuesta:
{
  "id": "uuid",
  "usuario_id": "uuid",
  "numero": "INV-MX-00048291",
  "tipo": "margen",
  "saldo": "0.000000",
  "saldo_bloqueado": "0.000000",
  "estado": "activa"
}
```

---

## 🔹 1.6. Sistema de UI/UX, Design Tokens y Navegación Inicial

### 🎨 Design Tokens (Archivo `lib/core/theme/tokens.dart`)
```dart
abstract class InvestechTokens {
  // Colores FinTech (confianza, claridad, accesibilidad)
  static const primary = Color(0xFF0A2540); // Azul institucional
  static const accent = Color(0xFF00D084);  // Verde rendimiento positivo
  static const error = Color(0xFFE53935);
  static const background = Color(0xFFF8F9FA);
  static const surface = Color(0xFFFFFFFF);
  
  // Tipografía
  static const fontBase = 'Inter';
  static const sizeH1 = 28.0;
  static const sizeBody = 15.0;
  
  // Espaciado
  static const spacingS = 8.0;
  static const spacingM = 16.0;
  static const spacingL = 24.0;
}
```

### 🧭 Estructura de Navegación (`go_router`)
| Ruta | Tipo | Acceso | Widget Principal |
|------|------|--------|------------------|
| `/` | Shell | Público | `OnboardingScreen` → `LoginScreen` |
| `/home` | Shell | Autenticado | `DashboardScreen` (BottomNav: Portfolio, Markets, Trade, Profile) |
| `/kyc/verify` | Route | Autenticado (pendiente) | `DocumentUploadScreen` + `RiskAssessmentWizard` |
| `/orders/:id` | Route | Autenticado | `OrderDetailScreen` + WebSocket status stream |
| `/markets/:ticker` | Route | Público/Autenticado | `InstrumentChartScreen` + Historical Data Table |
| `/settings/security` | Route | Autenticado | `BiometricsScreen`, `PinSetup`, `SessionManager` |

### 📱 Componentes Base Obligatorios
- `InvestechButton` (Primary, Secondary, Danger, Loading state)
- `InvestechInput` (Validación en tiempo real, máscaras numéricas)
- `InvestechCard` (Saldo, Posición, Transacción)
- `InvestechSkeleton` (Placeholder para carga asíncrona)
- `InvestechEmptyState` (Cero transacciones, sin portafolio, sin instrumentos)
- `InvestechErrorBoundary` (Manejo global de excepciones, retry logic)

---

## 📊 TABLA RESUMEN DE ENTREGABLES - PARTE 1

| Ítem | Estado | Herramienta/Estándar | Responsable |
|------|--------|----------------------|-------------|
| Repositorio inicial + estructura Clean Architecture | ✅ | GitFlow, `pubspec.yaml` | Lead Mobile |
| Configuración de sabores (Dev/Stg/Prod) | ✅ | `flutter_flavorizr`, `envied` | DevOps/Mobile |
| Pipeline CI/CD básico (Lint → Test → Build → Distribute) | ✅ | GitHub Actions, Firebase App Distribution | DevOps |
| BFF (Cloud Functions) scaffold + conexión a PostgreSQL | ✅ | Node.js/TS, Prisma, PgBouncer | Backend Lead |
| Estrategia de seguridad (SSL Pin, SecureStorage, MFA) | ✅ | `local_auth`, `flutter_secure_storage` | Security Engineer |
| Design System v1 + Navegación declarativa | ✅ | `go_router`, `Material 3` + custom tokens | UX/UI + Frontend |
| Mapeo completo SQL → Features Flutter documentado | ✅ | Tabla 1.2 | Arquitecto |

---

# 📘 PARTE 2: NÚCLEO FUNCIONAL, CAPA DE DATOS, ESTADO Y ESTRATEGIA OFFLINE-FIRST (Semanas 4-7)

Esta fase materializa la arquitectura definida en la Parte 1. Se implementan los flujos críticos de identidad, cuentas, mercados y se construye la capa de datos robusta, tipada y segura, alineada estrictamente con los `ENUM`, `NUMERIC(18,6)` y `JSONB` de tu esquema PostgreSQL.

---

## 🔹 2.1. Mapeo de Tipos PostgreSQL → Dart & Gestión de Precisión Financiera

⚠️ **Regla Crítica FinTech:** *Nunca usar `double` o `float` para saldos, precios o montos.* La imprecisión binaria genera errores de redondeo inaceptables en trading.

| Tipo PostgreSQL | Tipo Dart Recomendado | Librería/Manejo | Justificación |
|-----------------|----------------------|-----------------|---------------|
| `UUID` | `String` | `uuid` package (solo para generación local) | Consistencia con BFF y DB |
| `NUMERIC(18,6)` | `String` → `Decimal` | `decimal` package (`Decimal.parse()`) | Precisión exacta 18 dígitos, 6 decimales |
| `TIMESTAMPTZ` | `DateTime` (UTC) | `intl` + `timezone` | Normalización global, sin desfases horarios |
| `JSONB` (`respuestas`) | `Map<String, dynamic>` | `json_serializable` | Serialización segura, validación esquemática |
| `ENUM` (`estado_usuario`, etc.) | `enum Dart` | `@JsonValue()` con `freezed` | Tipado fuerte, prevención de valores inválidos |
| `INET` | `String` | Validación regex en BFF | Solo se consume como metadata de auditoría |

**📦 Ejemplo de DTO con Precisión Decimal (`freezed` + `json_serializable`)**
```dart
@freezed
class CuentaDto with _$CuentaDto {
  const factory CuentaDto({
    required String id,
    required String usuarioId,
    required String numero,
    @JsonValue('tipo') required TipoCuenta tipo,
    required String moneda,
    @JsonKey(name: 'saldo') required String saldoRaw,
    @JsonKey(name: 'saldo_bloqueado') required String saldoBloqueadoRaw,
    required EstadoCuenta estado,
    @JsonKey(name: 'creado_en') required DateTime creadoEn,
  }) = _CuentaDto;

  factory CuentaDto.fromJson(Map<String, dynamic> json) => _$CuentaDtoFromJson(json);
}

// Mapper a Entidad de Dominio
extension DtoToEntity on CuentaDto {
  CuentaEntity toEntity() => CuentaEntity(
    id: id,
    numero: numero,
    tipo: tipo,
    moneda: moneda,
    saldo: Decimal.parse(saldoRaw),
    saldoBloqueado: Decimal.parse(saldoBloqueadoRaw),
    estado: estado,
    creadoEn: creadoEn,
  );
}
```

---

## 🔹 2.2. Flujo de Autenticación, Sesiones y KYC (Tablas `USUARIO`, `SESION`, `DOCUMENTO`, `PERFIL_RIESGO`)

### 2.2.1. Arquitectura de Login Seguro
1. **Firebase Auth:** Registro/Login con Email/Password o Google/Apple Sign-In.
2. **Token Exchange:** App envía `Firebase ID Token` → BFF `/v1/auth/exchange`.
3. **BFF:** Verifica firma, crea registro en `SESION` (`ip`, `dispositivo`, `expiracion`), devuelve `access_token` (JWT) + `refresh_token`.
4. **Flutter:** Almacena tokens en `flutter_secure_storage`. Configura interceptor `Dio` para inyectar `Authorization: Bearer <access_token>`.
5. **Refresco Automático:** Si `401 Unauthorized`, interceptor usa `refresh_token` → `/v1/auth/refresh`. Si falla, redirige a `/login` y revoca sesión en `SESION.revocada = TRUE`.

### 2.2.2. KYC & Perfil de Riesgo
| Paso | Tabla SQL | Acción Flutter | Validación |
|------|-----------|----------------|------------|
| 1. Identidad | `USUARIO` | Registro básico + país (`CHAR(2)`) | Regex email, teléfono, ISO 3166 país |
| 2. Documentos | `DOCUMENTO` | Captura foto INE/Pasaporte → `Firebase Storage` → URL → POST a BFF | `tipo_documento`, `estado_documento = 'pendiente'` |
| 3. Evaluación | `PERFIL_RIESGO` | Wizard de 15 preguntas → `respuestas JSONB` → cálculo puntaje 0-100 | `CHECK (puntaje BETWEEN 0 AND 100)`, `nivel_riesgo` auto-asignado |
| 4. Aprobación | `DOCUMENTO` + `USUARIO` | Polling `/v1/kyc/status` cada 30s | Si `estado_documento = 'aprobado'` → `USUARIO.verificado_en = NOW()` → acceso full |

**🧩 BLoC State Machine para KYC**
```dart
abstract class KycState extends Equatable {}
class KycInitial extends KycState {}
class KycDocumentUploading extends KycState { final double progress; ... }
class KycDocumentUploaded extends KycState { final String docId; ... }
class KycRiskAssessmentComplete extends KycState { final NivelRiesgo nivel; ... }
class KycPendingReview extends KycState {}
class KycApproved extends KycState { final DateTime verificadoEn; ... }
class KycRejected extends KycState { final String motivo; ... }
```

---

## 🔹 2.3. Cuentas, Saldos y Transacciones (Tablas `CUENTA`, `TRANSACCION`)

### 2.3.1. Visualización de Saldos
- `CUENTA.saldo` y `CUENTA.saldo_bloqueado` se muestran con formato de moneda local (`intl`).
- **Bloqueo en Tiempo Real:** Al crear una orden, BFF actualiza `saldo_bloqueado` y emite evento WS. UI refleja disponibilidad inmediatamente.
- **Estado:** Badges dinámicos según `estado_cuenta` (`activa` = verde, `suspendida` = naranja, `cerrada` = rojo + bloqueo de trading).

### 2.3.2. Historial de Transacciones
- Endpoint paginado: `GET /v1/accounts/{id}/transactions?limit=20&offset=0&tipo=deposito,retiro`
- Mapeo `tipo_transaccion` → Icono + Color + Descripción
- **Filtros Avanzados:** Rango de fechas (`fecha TIMESTAMPTZ`), tipo, estado (`pendiente`, `completada`, `revertida`).
- **UX:** Infinite scroll con `ListView.builder`, skeleton loaders, pull-to-refresh.

---

## 🔹 2.4. Mercados, Instrumentos y Datos Históricos (Tablas `MERCADO`, `INSTRUMENTO`, `PRECIO_HISTORICO`)

### 2.4.1. Catálogo de Activos
- `MERCADO` → Tabs superiores (NYSE, BMV, NASDAQ, CRYPTO, CETES)
- `INSTRUMENTO` → Lista con búsqueda por `ticker` o `nombre`. Filtro por `tipo_instrumento`, `activo = TRUE`.
- **Detalle:** `isin`, `moneda`, `lote_minimo`, zona horaria del mercado, horario de operación.

### 2.4.2. Gráficos OHLCV (`PRECIO_HISTORICO`)
- **Librería:** `fl_chart` o `syncfusion_flutter_charts`
- **Endpoints:** 
  - `GET /v1/instruments/{id}/prices/history?range=1D|1W|1M|3M|1Y`
  - Respuesta optimizada: Array de `[timestamp, open, high, low, close, volume]` para reducir payload JSON.
- **Indicadores Técnicos (Opcional V1):** SMA/EMA calculados en BFF o en `compute()` isolates en Flutter para no bloquear UI.

---

## 🔹 2.5. Gestión de Estado y Patrón Offline-First

### 2.5.1. Arquitectura BLoC + Selector
```dart
// Evita rebuilds innecesarios
BlocBuilder<PortfolioBloc, PortfolioState>(
  buildWhen: (prev, curr) => prev.positions != curr.positions,
  builder: (context, state) => _buildPositionsList(state.positions),
)
```

### 2.5.2. Estrategia de Cache Local (`Hive` / `Isar`)
| Entidad | Estrategia Cache | TTL | Sincronización |
|---------|------------------|-----|----------------|
| `INSTRUMENTO` | `Hive` (Key: ticker) | 1 hora | Background fetch al abrir app |
| `PRECIO_HISTORICO` | `Isar` (DB local) | 24h por rango | Refresh on pull, delta sync |
| `TRANSACCION` | `Hive` (Pagina actual) | Indefinido (hasta refresh) | Append on new, invalidate on pull |
| `CUENTA`/`POSICION` | Memory + `flutter_secure_storage` | 5 min | Polling/WS push |
| `ORDEN` | Memory | Indefinido hasta cierre | Optimistic UI + WS reconciliation |

**🔄 Sync Manager**
- `ConnectivityPlus` detecta red.
- Al reconectar: `SyncService.flushPendingRequests()` → ejecuta cola de operaciones fallidas (ej. subida de documentos, creación de órdenes pendientes).
- Resolución de conflictos: Server wins para saldos/sesiones. Client wins para UI drafts no enviados.

---

## 🔹 2.6. Integración BFF: Endpoints REST y Estándares

| Recurso | Método | Ruta | Payload/Query | Respuesta | Mapeo SQL |
|---------|--------|------|---------------|-----------|-----------|
| Auth | POST | `/v1/auth/exchange` | `{ id_token: "..." }` | `{ access, refresh, user }` | `SESION`, `USUARIO` |
| KYC | POST | `/v1/kyc/document` | `multipart/form-data` | `{ doc_id, estado }` | `DOCUMENTO` |
| KYC | POST | `/v1/kyc/risk` | `{ respuestas: JSONB }` | `{ nivel, puntaje }` | `PERFIL_RIESGO` |
| Cuentas | GET | `/v1/accounts` | - | `[CuentaDto]` | `CUENTA` |
| Transacciones | GET | `/v1/transactions` | `?account_id=&tipo=&limit=&offset=` | `{ data, total, has_next }` | `TRANSACCION` |
| Mercados | GET | `/v1/markets` | - | `[MercadoDto]` | `MERCADO` |
| Instrumentos | GET | `/v1/instruments` | `?ticker=&tipo=&activo=true` | `[InstrumentoDto]` | `INSTRUMENTO` |
| Precios | GET | `/v1/instruments/{id}/prices` | `?range=1D&interval=15m` | `{ candles: [...] }` | `PRECIO_HISTORICO` |

**🛡️ Manejo de Errores Estandarizado**
```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Saldo disponible insuficiente para ejecutar orden",
    "details": { "required": "150.50", "available": "42.10" },
    "trace_id": "req_8f3a9b2c"
  }
}
```
- Flutter `Dio` interceptor mapea a `AppException` con `RetryStrategy` según código.

---

## 📊 TABLA RESUMEN DE ENTREGABLES - PARTE 2

| Ítem | Estado | Herramienta/Estándar | Métrica de Éxito |
|------|--------|----------------------|------------------|
| Capa de Datos (DTOs, Mappers, Repos) con precisión decimal | ✅ | `freezed`, `decimal`, `dio` | 0 errores de redondeo en tests unitarios |
| Auth + Sesiones + Refresh Token automático | ✅ | Firebase Auth + JWT interceptor | Sesión persiste 30 días, revocación remota funcional |
| KYC completo (Docs + Riesgo) | ✅ | `image_picker`, `firebase_storage`, `Hive` | Flujo < 3 min, validación en tiempo real |
| Cuentas & Transacciones con paginación | ✅ | `SliverList`, `Equatable`, `GoRouter` | Scroll fluido 60fps, carga < 800ms |
| Mercados & Gráficos OHLCV | ✅ | `fl_chart`, `Isar` cache | Renderizado 1Y < 1.2s, zoom/pan sin lag |
| Estrategia Offline-First | ✅ | `connectivity_plus`, `hive` | App usable sin red, sync automático al recuperar |
| BFF Endpoints V1 validados contra ENUM SQL | ✅ | `Zod`/`Pydantic`, `Prisma`/`Kysely` | 100% validación de tipos, 0 `null` inesperados |

---
---

# 📘 PARTE 3: OPERATIVA DE TRADING, PORTAFOLIOS, TIEMPO REAL Y CALIDAD (Semanas 8-11)

En esta fase se implementa el motor de negociación, la gestión de portafolios, la infraestructura de streaming financiero, las pruebas automatizadas y la optimización crítica para producción FinTech.

---

## 🔹 3.1. Motor de Órdenes y Ejecuciones (Tablas `ORDEN`, `EJECUCION`)

### 3.1.1. Creación de Órdenes (Formulario Avanzado)
| Campo UI | Mapeo SQL | Validación Frontend | Validación BFF |
|----------|-----------|---------------------|----------------|
| Tipo | `tipo_orden` | Mercado/Límite/Stop/Stop Limit | Coherencia con precios actuales |
| Lado | `lado_orden` | Compra/Venta | Verifica titularidad o margen |
| Cantidad | `cantidad NUMERIC(18,8)` | Múltiplo de `lote_minimo` | `CHECK` en DB, validación estricta |
| Precio Límite | `precio_limite NUMERIC(18,6)` | Obligatorio si `tipo != mercado` | Dentro de bands de volatilidad |
| Precio Stop | `precio_stop NUMERIC(18,6)` | Obligatorio si `tipo == stop` | Relación lógica con precio mercado |
| Vigencia | `vigencia_orden` | Día/GTC/IOC/FOK | Lógica de expiración en motor |
| Cuenta | `cuenta_id` | Selección de cuenta activa | `estado_cuenta = 'activa'` |

### 3.1.2. Máquina de Estados de `ORDEN`
```
[PENDIENTE] → (validación fondos/margen) → [ACTIVA]
   ↓                              ↓
[RECHAZADA] (saldo insuficiente)  ↓
                                 ↓ (match en mercado)
                              [EJECUTADA] → genera N registros en `EJECUCION`
                                 ↓
                          [CANCELADA] (usuario o expiración)
```
- **UI:** Timeline visual, badges de estado, botón de cancelación rápida para `pendiente`/`activa`.
- **Parcialidad:** Si `EJECUCION.cantidad < ORDEN.cantidad`, la orden queda `activa` con `cantidad_restante`. UI muestra `% llenado`.

---

## 🔹 3.2. Portafolios y Posiciones (Tablas `PORTAFOLIO`, `POSICION`)

### 3.2.1. Agregación de Posiciones
- `POSICION.cantidad`, `precio_promedio`, `valor_mercado`, `ganancia_perdida`
- **Cálculo P&L:** `ganancia_perdida = valor_mercado - (cantidad * precio_promedio)`
- **Actualización:** BFF recalcula `valor_mercado` cada vez que llega un `PRECIO_HISTORICO.cierre` o tick en tiempo real.
- **Visualización:** 
  - Pie chart por `tipo_instrumento` o `MERCADO`
  - Tabla ordenable por `ganancia_perdida` (abs y %)
  - Filtros: solo posiciones abiertas, por cuenta, por tipo

### 3.2.2. Portafolios Múltiples
- `PORTAFOLIO.nombre`, `descripcion`, `valor_total`, `rendimiento_pct`
- UI permite crear portafolios temáticos (ej. "Dividendos MX", "Tech US", "Cripto Largo Plazo")
- Rebalanceo sugerido basado en `PERFIL_RIESGO.nivel` y desviación > 5% del target.

---

## 🔹 3.3. Streaming y Actualización en Vivo (WebSockets / SSE)

### 3.3.1. Arquitectura de Tiempo Real
```
[PostgreSQL + Redis] → [Market Data Feed] → [BFF Publisher] → [WebSocket/SSE] → [Flutter Client]
```
- **Por qué no directo a DB:** PostgreSQL no está diseñado para 10k+ conexiones concurrentes de streaming. Se usa Redis Pub/Sub o NATS en BFF.

### 3.3.2. Canales de Suscripción
| Canal | Formato | Payload | Frecuencia | Acción Flutter |
|-------|---------|---------|------------|----------------|
| `prices/{instrument_id}` | JSON | `{ ts, bid, ask, last, vol }` | ~100-500ms | Actualiza gráfico, P&L en vivo, UI de compra/venta |
| `orders/{account_id}` | JSON | `{ order_id, estado, llenado, precio }` | Evento-driven | Actualiza lista de órdenes, notifica fills |
| `positions/{portfolio_id}` | JSON | `{ pos_id, valor_mercado, pnl }` | Cada tick relevante | Recalcula totales de portafolio |
| `account/{account_id}` | JSON | `{ saldo, bloqueado }` | Post-transacción | Actualiza header de saldo |

### 3.3.3. Manejo de Desconexiones y Duplicados
- `web_socket_channel` con `reconnect: true`, `maxRetries: 5`
- **Idempotencia:** Cada mensaje incluye `seq_id`. Flutter descarta `seq_id <= last_processed`.
- **Fallback:** Si WS cae > 30s, activa polling HTTP cada 5s hasta reconexión.
- **Optimistic UI:** Al enviar orden, se muestra `pendiente` inmediatamente. WS confirma `activa` o `rechazada`. Si diverge, UI se reconcilia con estado server.

---

## 🔹 3.4. Estrategia de Testing (Unit, Widget, Integration, E2E)

| Nivel | Herramienta | Cobertura Objetivo | Ejemplo Crítico |
|-------|-------------|-------------------|-----------------|
| **Unit** | `flutter_test`, `mocktail` | >85% capa `domain/` | `CalculatePnLUseCase` con `Decimal`, validación `lote_minimo` |
| **BLoC** | `bloc_test` | 100% estados | `OrderBloc` transición `pendiente → activa → ejecutada` |
| **Widget** | `golden_toolkit`, `mocktail` | UI componentes | `InvestechCard` renderiza saldos, estados, skeletons |
| **Integration** | `integration_test` | Flujos completos | Login → KYC → Depósito → Crear Orden → Ver Posición |
| **E2E** | `maestro` o `patrol` | Críticos en producción | Onboarding real, push notifications, deep links, recovery |

**🧪 Ejemplo Test BLoC con Enum SQL**
```dart
blocTest<OrderBloc, OrderState>(
  'emits [OrderLoading, OrderSuccess] when valid market order',
  build: () {
    when(() => repo.createOrder(any())).thenAnswer((_) async => Right(orderEntity));
    return OrderBloc(repo);
  },
  act: (bloc) => bloc.add(CreateOrderEvent(type: TipoOrden.mercado, lado: LadoOrden.compra, ...)),
  expect: () => [OrderLoading(), OrderSuccess(order)],
);
```

---

## 🔹 3.5. Optimización de Rendimiento y UX Crítica

| Área | Técnica | Impacto |
|------|---------|---------|
| **Rebuilds** | `BlocSelector`, `const` widgets, `ValueListenableBuilder` | -60% CPU en listas largas |
| **Listas** | `ListView.builder`, `SliverAppBar`, `cacheExtent` | Scroll 60fps constante con 10k items |
| **Cálculos Pesados** | `compute()` isolates para P&L, indicadores técnicos | 0 frame drops en UI thread |
| **Imágenes/Assets** | `cached_network_image`, `Image.asset` con `isAntiAlias` | -40% memory leak en gráficos históricos |
| **Network** | `Dio` con `Interceptor` de retry exponencial, compresión `gzip` | TTFB < 300ms en 4G, tolerancia a fallos |
| **Memory** | `devtools` profiling, `leak_tracker` | 0 fugas en navegación repetida |

---

## 🔹 3.6. Analytics, Crashlytics y Observabilidad

### 3.6.1. Eventos de Negocio (`firebase_analytics`)
| Evento | Trigger | Parámetros | Uso |
|--------|---------|------------|-----|
| `onboarding_complete` | Fin wizard | `risk_level`, `country` | Tasa conversión registro |
| `kyc_submitted` | Upload docs | `doc_type`, `time_taken` | Friction points |
| `deposit_initiated` | Click depósito | `amount_range`, `method` | Funnel funding |
| `order_created` | Submit orden | `type`, `side`, `instrument` | Engagement trading |
| `order_filled` | WS match | `fill_pct`, `slippage` | Quality execution |
| `portfolio_view` | Tab portfolio | `positions_count`, `pnl_sign` | Retención |

### 3.6.2. Crashlytics & Logs Seguros
- `FirebaseCrashlytics.instance.recordError(e, stack)` en catch global.
- **PII Masking:** Middleware de logging reemplaza emails, cuentas, saldos con `***` antes de enviar.
- **Custom Keys:** `setUserId()`, `setCustomKey('risk_profile', nivel)`, `setCustomKey('app_flavor', env)`
- **Alertas:** Reglas en Firebase Console para `Crash-free users < 99.5%` o `ANR rate > 0.5%`.

---

## 📊 TABLA RESUMEN DE ENTREGABLES - PARTE 3

| Ítem | Estado | Herramienta/Estándar | Métrica de Éxito |
|------|--------|----------------------|------------------|
| Motor de Órdenes completo (4 tipos + vigencias) | ✅ | Validación estricta `Decimal`, máquina estados | 0 órdenes inválidas aceptadas, reconciliación 100% |
| Portafolios & Posiciones con P&L en tiempo real | ✅ | `Decimal`, `Isar` cache, agregación BFF | Cálculo exacto, actualización < 500ms post-tick |
| Streaming WebSocket/SSE con fallback | ✅ | `web_socket_channel`, `seq_id`, polling fallback | Latencia < 200ms, reconexión automática, 0 duplicados |
| Suite de Testing (Unit, Widget, Integration) | ✅ | `bloc_test`, `mocktail`, `maestro` | Cobertura >80%, CI gate en PRs |
| Optimización Rendimiento 60fps | ✅ | `compute`, `BlocSelector`, `devtools` | Memory < 150MB, CPU < 30% en uso intensivo |
| Analytics & Crashlytics con PII safe | ✅ | `firebase_analytics`, logging masks | Funnel tracking activo, crash-free >99.5% |

---
---

# 📘 PARTE 4: DESPLIEGUE, CUMPLIMIENTO, ESCALABILIDAD Y OPERACIONES POST-LANZAMIENTO (Semanas 12-14+)

Esta fase final materializa la transición de desarrollo a producción, garantiza el cumplimiento estricto de políticas financieras de Google Play, establece protocolos de lanzamiento controlado, define la arquitectura de mantenimiento continuo y cierra con una auditoría de seguridad regulatoria y hoja de ruta evolutiva.

---

## 🔹 4.1. Preparación para Google Play Store & Cumplimiento Normativo FinTech

### 4.1.1. Compilación, Firmado y Optimización de Release
```bash
# Generación de App Bundle optimizado para arquitectura ARM64 + x86_64
flutter build appbundle \
  --release \
  --flavor prod \
  --dart-define=APP_FLAVOR=prod \
  --dart-define=API_URL=https://api.investech.com \
  --target-platform android-arm64,android-x64

# Verificación de tamaño y dependencias
./gradlew :app:dependencies --configuration releaseRuntimeClasspath
./gradlew :app:lintProdRelease
```

| Configuración | Archivo | Propósito | Valor Crítico |
|---------------|---------|-----------|---------------|
| **ProGuard/R8** | `android/app/proguard-rules.pro` | Ofuscación + eliminación de código muerto | `-keep class com.investech.** { *; }` <br> `-keep class io.flutter.** { *; }` <br> `-keep class com.google.firebase.** { *; }` |
| **Play Integrity** | `android/app/build.gradle` | Reemplazo de SafetyNet, detección de root/emuladores | `implementation 'com.google.android.play:integrity:1.3.0'` |
| **Minify & Shrink** | `buildTypes.release` | Reducción de APK size + seguridad | `minifyEnabled true`, `shrinkResources true` |
| **Multi-dex** | `defaultConfig` | Soporte >64K métodos (Firebase + Flutter) | `multiDexEnabled true`, `coreLibraryDesugaringEnabled true` |
| **Target SDK** | `compileSdk 34`, `targetSdk 34` | Cumplimiento políticas 2024+ de Google | Requerido para Play App Signing y notificaciones |

### 4.1.2. Cumplimiento de Políticas FinTech en Google Play
| Requisito Play Console | Implementación Investech | Evidencia/Entregable |
|------------------------|--------------------------|----------------------|
| **Financial Services Policy** | Disclaimer de riesgo obligatorio en onboarding + KYC | Pantalla `RiskDisclosureScreen` con checkbox `acepto_terminos_inversion = TRUE` |
| **Data Safety Form** | Declaración de recolección: email, teléfono, docs KYC, IPs, ubicación | Documentación cifrada en tránsito (TLS 1.3) y reposo (AES-256) |
| **Age Rating** | Clasificación `13+` o `16+` según jurisdicción | Validación `USUARIO.pais` + fecha nacimiento en registro |
| **Privacy Policy** | URL pública en Play Store + in-app link | `https://investech.com/privacy` (GDPR/LGPD compliant) |
| **Terms of Service** | Aceptación digital vinculante con hash de versión | `USUARIO.terms_accepted_at TIMESTAMPTZ` en DB |
| **Investment Risk Warning** | Mensaje estático en pantallas de trading y portafolio | `PERFIL_RIESGO.nivel` determina nivel de advertencia visual |

---

## 🔹 4.2. Estrategia de Lanzamiento Gradual (Phased Rollout)

El lanzamiento de una app FinTech no es un evento, es un proceso controlado con gates de métricas.

### 4.2.1. Matriz de Rollout por Fases
| Fase | Audiencia | Duración | Gates de Aprobación | Métricas Clave a Monitorear |
|------|-----------|----------|---------------------|-----------------------------|
| **Internal Testing** | 20-50 empleados/QA | 7 días | 0 crashes críticos, flujo KYC 100% funcional | `Crash-free users ≥ 99.8%`, `ANR rate ≤ 0.3%` |
| **Closed Testing** | 300-500 usuarios pre-seleccionados | 14 días | Conversión KYC > 65%, órdenes ejecutadas sin fallos | `Order success rate ≥ 94%`, `WS reconnect ≤ 2%` |
| **Open Testing** | Público (Beta pública) | 21 días | Retención D7 ≥ 28%, P&L sync < 1s latency | `D1/D7 Retention`, `Avg session duration ≥ 6 min` |
| **Production (1% → 5%)** | 1% de tráfico orgánico | 72h | 0 incidencias de saldo, 0 fugas PII | `Revenue/funding funnel`, `Support tickets < 0.5%` |
| **Production (20% → 100%)** | Escalado progresivo | 14 días | Infraestructura estable bajo carga pico | `CPU/RAM BFF ≤ 70%`, `DB connection pool ≤ 80%` |

### 4.2.2. Protocolo de Rollback Inmediato
1. **Trigger:** `Crash-free < 99%` o `Order failure rate > 3%` por 30 min.
2. **Acción Play Console:** Detener rollout → Revertir a versión anterior estable.
3. **Comunicación:** Push notification a usuarios afectados + email de transparencia.
4. **Hotfix:** Branch `hotfix/v1.0.1` → CI/CD → Test interno acelerado → Re-subida.
5. **Post-Mortem:** Root cause analysis (RCA) en < 48h, actualización de `SESION` y `ORDEN` logs.

---

## 🔹 4.3. Mantenimiento Post-Lanzamiento, Hotfixes y Escalabilidad

### 4.3.1. Gestión de Versiones y Hotfixes
| Proceso | Herramienta | Frecuencia | SLA |
|---------|-------------|------------|-----|
| **Patch Semanal** | `flutter upgrade`, `pub get`, dependabot | Cada lunes | 4h |
| **Hotfix Crítico** | Gitflow `hotfix/` → Play Console Fast Track | Bajo demanda | < 2h desde detección |
| **Minor Release** | Feature branches → Staging → Prod | Cada 2-3 semanas | 24h |
| **Major Release** | Quarterlies (V1.1, V1.2, V2.0) | Trimestral | 1-2 semanas |

### 4.3.2. Escalabilidad de Infraestructura (BFF + PostgreSQL + Firebase)
| Componente | Estrategia de Escala | Umbral de Trigger | Acción Automática |
|------------|---------------------|-------------------|-------------------|
| **Cloud Run (BFF)** | Auto-scaling horizontal (1 → 50 instancias) | CPU > 65% o Latencia p95 > 400ms | `gcloud run services update --cpu-threshold=65` |
| **PostgreSQL** | Read replicas + Connection Pooling (PgBouncer) | Active connections > 80% pool size | Redirect `GET` a replica, mantener `POST/PUT` en primary |
| **Redis (Market Data)** | Cluster mode + eviction policy `allkeys-lru` | Memory > 75% o Hit rate < 85% | Auto-scale nodes, purge expired `PRECIO_HISTORICO` cache |
| **Firebase Auth** | Multi-factor + Project quota monitoring | Sign-in rate > 10k/min o Error 429 > 1% | Enable reCAPTCHA Enterprise, throttle IPs |
| **Storage (KYC Docs)** | CDN + Lifecycle policies (move to Coldline > 90d) | Egress > 5TB/mo o Latencia > 800ms | Compress images server-side, enable signed URLs with 15m expiry |

### 4.3.3. Mantenimiento de Base de Datos (Alineado a tu esquema)
```sql
-- Indexación crítica para carga de producción
CREATE INDEX idx_orden_estado_cuenta ON ORDEN(estado, cuenta_id) WHERE estado IN ('pendiente', 'activa');
CREATE INDEX idx_transaccion_fecha_tipo ON TRANSACCION(fecha DESC, tipo);
CREATE INDEX idx_posicion_pnl ON POSICION(ganancia_perdida DESC, portafolio_id);
CREATE INDEX idx_precio_fecha ON PRECIO_HISTORICO(instrumento_id, fecha DESC);

-- Vistas materializadas para dashboards (actualización cada 5 min)
CREATE MATERIALIZED VIEW mv_portfolio_summary AS
SELECT p.id, p.nombre, p.cuenta_id,
       COUNT(pos.id) as posiciones_abiertas,
       SUM(pos.valor_mercado) as valor_total,
       AVG(pos.ganancia_perdida) as pnl_promedio
FROM PORTAFOLIO p JOIN POSICION pos ON p.id = pos.portafolio_id
GROUP BY p.id;
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_portfolio_summary;
```

---

## 🔹 4.4. Checklist Final de Auditoría de Seguridad y Cumplimiento Regulatorio

Antes del rollout al 100%, se ejecuta una auditoría transversal validada por equipo de Compliance/Security.

| Categoría | Ítem de Auditoría | Estado | Herramienta/Evidencia | Responsable |
|-----------|-------------------|--------|------------------------|-------------|
| **Cifrado** | TLS 1.3 forzado en todas las conexiones BFF ↔ App | ✅ | `ssllabs.com` scan, `NetworkSecurityConfig` | DevOps |
| **Cifrado** | Datos sensibles en reposo (`flutter_secure_storage`, DB) | ✅ | Android Keystore, `pgcrypto` extension | Security Eng |
| **Sesiones** | Rotación de tokens + revocación remota (`SESION.revocada`) | ✅ | Test de invalidación en paralelo, logs `SESION` | Backend |
| **KYC/AML** | Validación de `PERFIL_RIESGO` antes de activar trading | ✅ | Gate en `CreateOrderUseCase`, validación `nivel_riesgo` | Compliance |
| **Auditoría** | Logs inmutables de `USUARIO`, `DOCUMENTO`, `ORDEN` | ✅ | ELK Stack / Cloud Logging, retención 5 años | Legal/Sec |
| **Integridad** | Play Integrity API + detección root/emuladores | ✅ | `firebase_app_check`, server-side validation | Mobile |
| **Privacidad** | PII masking en Crashlytics/Analytics | ✅ | Middleware `LogMasker`, `setAnalyticsCollectionEnabled(false)` hasta consent | Data Eng |
| **Resiliencia** | Circuit breaker en llamadas a BFF + fallback offline | ✅ | `Dio` interceptor + `Hive` sync queue | Mobile Lead |
| **Backups** | PostgreSQL PITR (Point-in-Time Recovery) + snapshots | ✅ | `pg_dump`, Cloud SQL automated backups, tested restore | DBA |
| **PenTest** | Escaneo OWASP Top 10 + pruebas de inyección SQL/API | ✅ | Reporte externo (firma certificada), 0 Critical/High | External Firm |

**📜 Declaración de Cumplimiento FinTech (Extracto)**
> *"Investech Mobile App v1.0 implementa controles de idoneidad (PERFIL_RIESGO), trazabilidad de órdenes (ORDEN → EJECUCION → TRANSACCION), cifrado de documentos KYC (DOCUMENTO) y auditoría de sesiones (SESION). Todos los saldos se calculan con precisión decimal NUMERIC(18,6) y se reconcilian diariamente contra el ledger contable. La plataforma opera bajo principios de transparencia, protección de datos y cumplimiento normativo aplicable a intermediarios de valores digitales."*

---

## 🔹 4.5. KPIs de Éxito, Roadmap de Evolución y Cierre del Proyecto

### 4.5.1. Dashboard de KPIs Post-Lanzamiento
| KPI | Meta V1 | Herramienta de Medición | Acción si < Meta |
|-----|---------|-------------------------|------------------|
| **Crash-Free Users** | ≥ 99.7% | Firebase Crashlytics | Hotfix inmediato, análisis de `main` thread |
| **KYC Conversion Rate** | ≥ 60% | Firebase Analytics + BFF | Optimizar UX wizard, reducir pasos, feedback en tiempo real |
| **Order Success Rate** | ≥ 95% | BFF logs + WS | Ajustar validación de margen, mejorar fallback de precios |
| **D7 Retention** | ≥ 28% | Analytics cohortes | Push notifications educativas, alertas de mercado personalizadas |
| **Avg Time to Fund** | ≤ 3 min | Transaction funnel | Integrar más pasarelas, pre-validar datos bancarios |
| **API p95 Latency** | ≤ 250ms | Cloud Monitoring | Optimizar queries, escalar PgBouncer, cachear respuestas estáticas |

### 4.5.2. Roadmap de Evolución (V1.5 → V3.0)
| Versión | Funcionalidades Clave | Dependencias DB/Backend | Impacto Negocio |
|---------|----------------------|-------------------------|-----------------|
| **V1.5** | Órdenes OCO/Trailing Stop, Alertas de precio, Exportación fiscal PDF | `ORDEN.tipo` extendido, `NOTIFICACION` table | +15% retención traders activos |
| **V2.0** | Web Dashboard (React), API Pública para desarrolladores, Multi-cuenta familiar | `API_KEY` table, `CUENTA.relations`, `JWT.scope` | Expansión B2B, ingresos por API |
| **V2.5** | Staking de cripto, Préstamos con colateral (margen avanzado), Social trading | `PRESTAMO` table, `STAKING_REWARDS`, `PORTAFOLIO.share` | Diversificación revenue streams |
| **V3.0** | IA para rebalanceo automático, Análisis de sentimiento en tiempo real, iOS nativo | `MODELO_PREDICCION`, `NEWS_SENTIMENT`, `CROSS_PLATFORM_SYNC` | Posicionamiento institucional |

### 4.5.3. Checklist de Cierre de Proyecto
- [ ] Código mergueado a `main` con tag `v1.0.0-prod`
- [ ] Documentación técnica (Swagger BFF, Diagramas de arquitectura, Runbooks)
- [ ] Manual de operaciones (SOPs para soporte, escalación, rollback)
- [ ] Transferencia de credenciales Firebase/Cloud Run/Play Console a equipo Ops
- [ ] Sesión de handover con Compliance y Legal (validación final de flujos KYC/Trading)
- [ ] Plan de contingencia documentado (DRP: Disaster Recovery Plan)
- [ ] Firma de aceptación de entregables por Product Owner / CTO

---
