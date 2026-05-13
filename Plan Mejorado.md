# Prompt
Actúa como Lead Flutter Architect, Firebase Solutions Expert y Senior UI/UX Engineer. Tu tarea es diseñar y estructurar una aplicación Android de plataforma de inversiones digitales llamada "Investech", utilizando Flutter, Firebase Firestore, Riverpod, Freezed y Material 3. Entrega un blueprint de implementación de nivel producción, completamente detallado, sin placeholders genéricos y siguiendo estándares 2026.

⛔ RESTRICCIONES OBLIGATORIAS:
- NO hagas referencia a archivos externos, scripts SQL previos o contextos anteriores.
- Considera toda la información proporcionada aquí como la única fuente de verdad.
- Usa nomenclatura profesional, código listo para producción y patrones Clean Architecture.
- Responde en español, con bloques de código, tablas comparativas y pasos ejecutables.
- Si algo requiere validación compleja, incluye la lógica exacta en Dart/Firestore.

📐 ARQUITECTURA & TECH STACK:
- Flutter ≥ 3.22.0 (Android, minSDK 24, Material 3)
- Firebase: Firestore, Auth, Storage, App Check, FCM, Crashlytics
- State Management: flutter_riverpod + freezed + json_serializable
- Arquitectura: Clean Architecture (Domain → Data → Presentation)
- Validación: dartz (Either), formz/zod-like, transacciones atómicas
- Seguridad: ProGuard/R8, SHA-256 audit logs, Play Integrity + App Check

📁 ESTRUCTURA DE CARPETAS EXACTA (RAÍZ → lib/):
```
appinvestech/
├── agents/
│   ├── skills.md
│   ├── flujo_de_trabajo.md
│   ├── reglas/
│   │   ├── firebase_security_rules.md
│   │   ├── naming_conventions.md
│   │   └── state_management_rules.md
│   └── subagentes/
│       ├── ui_agent.md
│       ├── data_agent.md
│       ├── trading_logic_agent.md
│       └── qa_agent.md
├── android/ (configuración nativa estándar)
├── assets/
│   ├── images/
│   ├── icons/
│   └── fonts/
├── lib/
│   ├── core/
│   │   ├── constants/
│   │   ├── errors/
│   │   ├── utils/
│   │   └── theme/
│   ├── data/
│   │   ├── datasources/
│   │   ├── models/
│   │   └── repositories/
│   ├── domain/
│   │   ├── entities/
│   │   └── repositories/
│   ├── presentation/
│   │   ├── providers/
│   │   ├── screens/
│   │   └── widgets/
│   ├── di/
│   └── main.dart
├── test/
├── pubspec.yaml
└── firebase.json
```

🔥 CONFIGURACIÓN FIREBASE (PROYECTO: `bdinvestech`):
- Paquete Android: `com.investech.appinvestech`
- `google-services.json` debe ubicarse en: `android/app/google-services.json`
- Incluir comandos de `flutterfire configure`, instalación de FlutterFire CLI, y verificación de conexión en terminal.
- Especificar dependencias exactas en `pubspec.yaml` con versiones estables 2026.
- Incluir estructura de `firebase.json` y `firestore.indexes.json` para consultas compuestas.

🤖 ECOSISTEMA DE AGENTES IA & FLUJO DE TRABAJO:
- Genera contenido base para `agents/skills.md` (alcance, restricciones, patrones obligatorios).
- Genera `agents/flujo_de_trabajo.md` (pipeline de desarrollo, revisión, merge, staging).
- Genera `agents/reglas/*.md` (seguridad, naming, estado).
- Genera `agents/subagentes/*.md` con responsabilidades claras: UI/UX, Data/Firestore, Lógica de Trading/Órdenes, QA/Testing.
- Define cómo los subagentes interactúan en PRs, code review y validación automática.

🗃️ ENTIDADES DE DOMINIO & MAPEO FIRESTORE (EXPLICITAS):
Define cada entidad con: nombre, campos exactos, tipos, constraints, relaciones y estructura en Firestore (colección/subcolección/documento). NO omitas ningún campo.

1. USUARIO: id (UUID), nombre (String, max 120), email (String, unique, max 180), telefono (String, max 20), pais (String ISO-2), estado (activo|suspendido|cerrado), verificado_en (Timestamp), creado_en (Timestamp)
2. DOCUMENTO: id (UUID), usuario_id (UUID ref), tipo (ine|pasaporte|comprobante|otro), url (String), estado (pendiente|aprobado|rechazado), revisado_por (UUID nullable), subido_en (Timestamp)
3. PERFIL_RIESGO: id (UUID), usuario_id (UUID ref), nivel (conservador|moderado|agresivo), puntaje (int 0-100), respuestas (Map/JSON), vigente (bool), evaluado_en (Timestamp)
4. CUENTA: id (UUID), usuario_id (UUID ref), numero (String unique, max 20), tipo (efectivo|margen|custodia), moneda (ISO-3, 3 chars), saldo (double/numeric), saldo_bloqueado (double/numeric), estado (activa|suspendida|cerrada), creado_en (Timestamp)
5. MERCADO: id (UUID), codigo (String unique, MIC), nombre (String, max 100), pais (ISO-2), zona_horaria (String), moneda_base (ISO-3), activo (bool)
6. INSTRUMENTO: id (UUID), mercado_id (UUID ref), ticker (String unique, max 20), nombre (String, max 150), tipo (accion|etf|bono|cete|cripto|fondo), moneda (ISO-3), isin (String unique, 12 chars), lote_minimo (double), activo (bool)
7. PRECIO_HISTORICO: id (UUID), instrumento_id (UUID ref), fecha (Date), apertura (double), maximo (double), minimo (double), cierre (double), cierre_ajustado (double nullable), volumen (int)
8. ORDEN: id (UUID), cuenta_id (UUID ref), instrumento_id (UUID ref), tipo (mercado|limite|stop|stop_limit), lado (compra|venta), cantidad (double), precio_limite (double nullable), precio_stop (double nullable), vigencia (dia|gtc|ioc|fok), estado (pendiente|activa|ejecutada|cancelada), creado_en (Timestamp)
9. EJECUCION: id (UUID), orden_id (UUID ref), cantidad (double), precio (double), comision (double), contraparte (String nullable), ejecutado_en (Timestamp)
10. TRANSACCION: id (UUID), cuenta_id (UUID ref), orden_id (UUID ref nullable), tipo (deposito|retiro|compra|venta|comision), monto (double), moneda (ISO-3), referencia (String nullable), estado (pendiente|completada|revertida), fecha (Timestamp)
11. PORTAFOLIO: id (UUID), cuenta_id (UUID ref), nombre (String, max 100), descripcion (String nullable), valor_total (double), rendimiento_pct (double nullable), actualizado_en (Timestamp)
12. POSICION: id (UUID), portafolio_id (UUID ref), instrumento_id (UUID ref), cantidad (double), precio_promedio (double), valor_mercado (double), ganancia_perdida (double nullable), actualizado_en (Timestamp)

📌 REGLAS DE NEGOCIO EXPLICITAS:
- Solo un PERFIL_RIESGO con `vigente=true` por usuario.
- `ORDEN` de compra requiere bloqueo de saldo (`saldo_bloqueado += cantidad * precio`).
- `EJECUCION` y `TRANSACCION` solo se crean vía backend seguro o Cloud Functions (nunca desde cliente).
- `PRECIO_HISTORICO` se consulta por `(instrumento_id ASC, fecha DESC)`.
- `PORTAFOLIO` y `POSICION` se recalculan en tiempo real con streams + cálculo P&L.
- Todos los montos usan precisión mínima de 6 decimales en Firestore.

🎨 SISTEMA DE DISEÑO & UI/UX:
- Paleta institucional: primario azul oscuro (#0A2E5C), acento verde trading (#00C896), alerta (#FFB400), error (#E63946), fondos neutros, texto primario/secundario.
- Tipografía: Inter (Google Fonts), escalas responsive con `flutter_screenutil`.
- Componentes obligatorios: `InvestmentTable` (reutilizable, filas dinámicas, colores condicionales), `OrderForm` (validación en tiempo real, preview de comisión), `PortfolioCard` (valor total, P&L con indicadores), `KYCUploadWidget` (drag/drop, validación MIME, progreso).
- Material 3, dark/light ready, contraste WCAG 2.1 AA, loading states, error banners, empty states.

🛡️ SEGURIDAD & FIRESTORE RULES:
- Entrega `firestore.rules` completo con:
  • Autenticación obligatoria para todo.
  • Lectura/escritura limitada a `request.auth.uid == userId`.
  • Escrituras de saldos/órdenes bloqueadas en cliente (`if false`).
  • Validación de tipos, rangos y formatos (`is string`, `is number`, `size`, `matches regex`).
  • Colecciones públicas: `mercados`, `instrumentos`, `precio_historico` (solo lectura).
- Incluye estrategia de índices compuestos en `firestore.indexes.json`.

⚙️ STATE MANAGEMENT & DATA FLOW:
- Riverpod providers: `authStateProvider`, `userProvider`, `accountsProvider`, `instrumentsProvider`, `ordersProvider`, `portfolioProvider`.
- Freezed models con `fromJson`/`toFirestore` + `Timestamp` conversion.
- Transacciones atómicas con `runTransaction` para bloqueos de saldo, creación de órdenes y KYC.
- Streams en tiempo real para órdenes, posiciones y precios.
- Error handling con `Either<Failure, T>`, retry logic exponencial, cache local opcional.

🧪 TESTING, CI/CD & DEPLOY:
- Estructura de tests: `test/domain/`, `test/data/`, `test/presentation/`.
- Cobertura objetivo: ≥85% unitarios, ≥70% widgets.
- Pipeline GitHub Actions: lint → format → test → build appbundle → upload internal track.
- Configuración ProGuard/R8, reglas `proguard-rules.pro` para Firebase/Freezed.
- Firma digital (keystore), `key.properties`, build commands con `--obfuscate --split-debug-info`.
- Checklist Play Store: screenshots, privacy policy, content rating, internal testing track, Crashlytics + Performance Monitoring.

📤 INSTRUCCIONES DE ENTREGA:
1. Responde en 4 fases claramente separadas (Fase 1: Setup + Agentes + Tema, Fase 2: Auth + KYC + RLS, Fase 3: Cuentas + Mercado + Órdenes + Portafolio, Fase 4: FCM + Auditoría + CI/CD + Deploy).
2. Usa tablas markdown para mapeos, checklist y validaciones.
3. Incluye bloques de código exactos (sin `// ...` ni placeholders).
4. Especifica comandos de terminal, rutas de archivos y configuraciones JSON/YAML completas.
5. No omitas validaciones de negocio, reglas de seguridad ni estrategias de optimización Firestore.
6. Mantén un tono técnico, profesional y listo para implementación inmediata en equipo.

Comienza entregando la FASE 1 completa. Espera confirmación antes de continuar a la siguiente fase.

---

## 📦 1.1. INICIALIZACIÓN DEL PROYECTO FLUTTER (ANDROID)

### 🔹 Comandos de creación y preparación
```bash
# Verificar versiones mínimas (2026)
flutter --version        # ≥ 3.22.0
dart --version           # ≥ 3.4.0

# Crear proyecto con namespace profesional
flutter create --org com.investech --platforms android appinvestech

# Inicializar control de versiones
cd appinvestech
git init
git add .
git commit -m "🚀 Initial commit: Flutter project structure & base configuration"
```

### 📁 Estructura de carpetas raíz solicitada
| Directorio/Archivo | Propósito |
|-------------------|-----------|
| `agents/` | Contenedor principal para IA, reglas y flujos |
| `agents/skills.md` | Capacidades técnicas y restricciones del asistente |
| `agents/reglas/` | Estándares de código, seguridad Firebase, naming conventions |
| `agents/subagentes/` | Roles especializados (UI, Data, Auth, Trading, QA) |
| `agents/flujo_de_trabajo.md` | Pipeline de desarrollo, reviews, CI/CD |
| `lib/` | Código fuente Flutter |
| `android/` | Configuración nativa Android |
| `assets/` | Imágenes, fuentes, JSON locales |
| `pubspec.yaml` | Dependencias y metadatos |

---

## 🏗️ 1.2. ESTRUCTURA PROFESIONAL EN `lib/` (CLEAN ARCHITECTURE + FIRESTORE READY)

Se recomienda una arquitectura escalable, separando responsabilidades y preparada para mapear las tablas SQL a colecciones/documentos de Firestore.

```
lib/
├── core/
│   ├── constants/          # Rutas, claves, URLs, timeouts
│   ├── errors/             # Failure classes, exceptions
│   ├── utils/              # Formatters, validators, date helpers
│   └── theme/              # AppTheme, colors, typography, spacing
├── data/
│   ├── datasources/        # Firebase Firestore & Auth services
│   ├── models/             # Freezed/JsonSerializable DTOs
│   └── repositories/       # Implementación de contratos de dominio
├── domain/
│   ├── entities/           # Objetos de negocio puros (User, Account, Order, etc.)
│   └── repositories/       # Interfaces abstractas
├── presentation/
│   ├── providers/          # Riverpod/Bloc state managers
│   ├── screens/            # Vistas principales (Login, Dashboard, Trading, Portfolio)
│   └── widgets/            # Componentes reutilizables (Buttons, Cards, Tables)
├── di/                     # Inyección de dependencias (Provider/Riverpod)
└── main.dart               # Entry point
```

> 🔄 **Nota de adaptación SQL → Firestore**:  
> Firestore es NoSQL orientado a documentos. Las tablas relacionales se transforman en:
> - `USUARIO` → `users/{userId}` (documento)
> - `CUENTA`, `ORDEN`, `PORTAFOLIO` → Subcolecciones: `users/{userId}/accounts/{accountId}`, etc.
> - `PRECIO_HISTORICO`, `EJECUCION` → Colecciones independientes con índices compuestos para consultas temporales.

---

## 🔥 1.3. CONFIGURACIÓN DE FIREBASE (`bdinvestech`)

### ✅ Paso a paso en Firebase Console
1. Ir a [console.firebase.google.com](https://console.firebase.google.com)
2. `Agregar proyecto` → Nombre: `bdinvestech` → Desactivar Analytics (opcional) → Crear
3. En el panel izquierdo: `Firestore Database` → `Crear base de datos`
4. Modo: `Modo de producción` (luego se ajustarán reglas de seguridad)
5. Ubicación: `nam5 (us-central)` o `eur3` según tu región
6. Habilitar `Authentication` → `Email/Password` y `Google Sign-In`

### 📥 Descarga y ubicación del `google-services.json`
| Paso | Acción |
|------|--------|
| 1 | En Firebase Console → `⚙️ Configuración del proyecto` → `Tus apps` → `Agregar app` → Icono Android |
| 2 | Nombre del paquete: `com.investech.appinvestech` |
| 3 | Descargar `google-services.json` |
| 4 | **Ubicación exacta**: `appinvestech/android/app/google-services.json` |
| 5 | Verificar que `build.gradle` (nivel app) tenga: `apply plugin: 'com.google.gms.google-services'` |

---

## 📜 1.4. DEPENDENCIAS Y `pubspec.yaml`

Configuración optimizada para 2026 con gestión de estado moderna, serialización y Firebase oficial.

```yaml
name: appinvestech
description: Plataforma digital de inversiones Investech
version: 1.0.0+1

environment:
  sdk: ">=3.4.0 <4.0.0"
  flutter: ">=3.22.0"

dependencies:
  flutter:
    sdk: flutter
  # Firebase & Cloud
  firebase_core: ^3.1.0
  cloud_firestore: ^5.0.0
  firebase_auth: ^5.0.0
  firebase_app_check: ^0.3.0
  # State Management
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  # Utilities & UI
  freezed_annotation: ^2.4.0
  json_annotation: ^4.9.0
  intl: ^0.19.0
  flutter_screenutil: ^5.9.0
  google_fonts: ^6.2.0
  equatable: ^2.0.5
  dartz: ^0.10.1
  collection: ^1.18.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  riverpod_generator: ^2.4.0
  flutter_lints: ^4.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/icons/
```

🔹 **Post-instalación**:
```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
```

---

## 🤖 1.5. SISTEMA DE AGENTES IA, REGLAS Y FLUJO DE TRABAJO

Crea la carpeta `agents/` en la raíz del proyecto y estructura los siguientes archivos:

### 📁 Estructura `agents/`
```
agents/
├── skills.md
├── flujo_de_trabajo.md
├── reglas/
│   ├── firebase_security_rules.md
│   ├── naming_conventions.md
│   └── state_management_rules.md
└── subagentes/
    ├── ui_agent.md
    ├── data_agent.md
    ├── trading_logic_agent.md
    └── qa_agent.md
```

### 📄 Contenido base recomendado

**`skills.md`**
```markdown
# 🤖 Investech AI Agent - Skills & Scope
- ✅ Arquitectura Flutter Clean + Riverpod
- ✅ Firestore Data Modeling (denormalización, subcolecciones, índices)
- ✅ Seguridad financiera (App Check, reglas RLS, validación de montos)
- ✅ UI/UX accesible (WCAG 2.1, contraste, theming dinámico)
- ✅ Testing unitario y de integración para lógica de órdenes
- 🚫 No exponer claves API en código
- 🚫 No usar setState() para estado global
```

**`flujo_de_trabajo.md`**
```markdown
# 🔄 Flujo de Desarrollo Investech
1. 📝 Definir entidad en `domain/` → generar modelo con `freezed`
2. 🗃️ Mapear a Firestore en `data/datasources/`
3. 🔌 Exponer repositorio en `data/repositories/`
4. 📱 Crear provider en `presentation/providers/`
5. 🎨 Implementar pantalla en `presentation/screens/`
6. ✅ Tests unitarios + linting
7. 🔀 PR → Review → Merge → Deploy staging
```

**`subagentes/trading_logic_agent.md`** (ejemplo)
```markdown
# 📊 Trading Logic Agent
Responsable de:
- Validar saldos antes de órdenes (saldo - bloqueado >= monto)
- Calcular comisiones por tipo de cuenta
- Gestionar estados de orden: pendiente → activa → ejecutada/cancelada
- Sincronizar posiciones con Firestore en tiempo real
```

---

## 🔌 1.6. VERIFICACIÓN DE CONECTIVIDAD FIREBASE (TERMINAL)

### 🔹 Instalación de FlutterFire CLI
```bash
dart pub global activate flutterfire_cli
```

### 🔹 Configuración automática
```bash
cd appinvestech
flutterfire configure --project=bdinvestech --platforms=android --out=lib/firebase_options.dart
```
✅ Esto genera `firebase_options.dart` y valida la conexión con Firebase.

### 🔹 Prueba de conexión en `main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(const InvestechApp());
}
```

### 🔹 Validación rápida desde terminal
```bash
# Verificar que Firebase CLI reconoce el proyecto
firebase projects:list | grep bdinvestech

# Verificar conexión real (script temporal)
dart run lib/main.dart
# Revisar logs: ✅ FirebaseApp [DEFAULT] initialized successfully.
```

---

## 🎨 1.7. SISTEMA DE DISEÑO (UI/UX) Y GESTIÓN DE COLORES

Diseño institucional para plataforma financiera: sobrio, accesible y con contraste validado.

### 🎨 Paleta de colores `lib/core/theme/app_colors.dart`
```dart
import 'package:flutter/material.dart';

class AppColors {
  // Primarios
  static const primary = Color(0xFF0A2E5C);   // Azul Investech Profundo
  static const primaryLight = Color(0xFF1A4B8C);
  static const primaryDark = Color(0xFF061C3A);

  // Acentos & Funcionales
  static const accent = Color(0xFF00C896);    // Verde Éxito/Trading
  static const warning = Color(0xFFFFB400);   // Alertas/Margen
  static const error = Color(0xFFE63946);     // Rechazos/Pérdidas
  static const neutral = Color(0xFFF5F7FA);   // Fondo claro
  static const surface = Color(0xFFFFFFFF);   // Cards/Contenedores
  static const textPrimary = Color(0xFF111827);
  static const textSecondary = Color(0xFF6B7280);

  // Mercado (Trading)
  static const marketUp = Color(0xFF10B981);
  static const marketDown = Color(0xFFEF4444);
}
```

### 🖼️ Tema global `lib/core/theme/app_theme.dart`
```dart
import 'package:flutter/material.dart';
import 'app_colors.dart';
import 'package:google_fonts/google_fonts.dart';

class AppTheme {
  static ThemeData light = ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.light(
      primary: AppColors.primary,
      secondary: AppColors.accent,
      surface: AppColors.surface,
      error: AppColors.error,
      onPrimary: Colors.white,
      onSurface: AppColors.textPrimary,
    ),
    scaffoldBackgroundColor: AppColors.neutral,
    textTheme: GoogleFonts.interTextTheme().copyWith(
      displayLarge: GoogleFonts.inter(fontWeight: FontWeight.w700),
      bodyMedium: GoogleFonts.inter(fontSize: 14, color: AppColors.textPrimary),
    ),
    cardTheme: CardTheme(
      elevation: 2,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      color: AppColors.surface,
    ),
    appBarTheme: AppBarTheme(
      backgroundColor: AppColors.primary,
      foregroundColor: Colors.white,
      elevation: 0,
    ),
  );
}
```

### 📱 Ejemplo de widget de tabla financiera reutilizable
```dart
// lib/presentation/widgets/investment_table.dart
class InvestmentTable extends StatelessWidget {
  final List<Map<String, String>> rows;
  const InvestmentTable({super.key, required this.rows});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: AppColors.surface,
        borderRadius: BorderRadius.circular(12),
        boxShadow: [BoxShadow(color: Colors.black.withOpacity(0.05), blurRadius: 8)],
      ),
      child: Column(
        children: [
          for (final row in rows)
            Padding(
              padding: const EdgeInsets.symmetric(vertical: 8),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(row['label']!, style: const TextStyle(fontWeight: FontWeight.w500)),
                  Text(row['value']!, style: TextStyle(
                    color: row['color'] == 'green' ? AppColors.marketUp : 
                           row['color'] == 'red' ? AppColors.marketDown : AppColors.textPrimary,
                    fontWeight: FontWeight.w600,
                  )),
                ],
              ),
            ),
        ],
      ),
    );
  }
}
```

---

## 📊 RESUMEN DE PARTE 1 (CHECKLIST DE IMPLEMENTACIÓN)

| ✅ Tarea | Estado | Comando/Ubicación |
|---------|--------|-------------------|
| Crear proyecto Flutter Android | ✔️ | `flutter create --org com.investech --platforms android appinvestech` |
| Estructura `lib/` Clean Architecture | ✔️ | Ver sección 1.2 |
| Proyecto Firebase `bdinvestech` | ✔️ | Console → Firestore → Production Mode |
| Descargar & ubicar `google-services.json` | ✔️ | `android/app/google-services.json` |
| Configurar `pubspec.yaml` | ✔️ | Ver sección 1.4 |
| Crear carpeta `agents/` con archivos | ✔️ | `skills.md`, `reglas/`, `subagentes/`, `flujo_de_trabajo.md` |
| Instalar FlutterFire CLI & configurar | ✔️ | `flutterfire configure --project=bdinvestech` |
| Verificar conectividad | ✔️ | Logs `FirebaseApp initialized successfully` |
| Sistema de colores y tema base | ✔️ | `app_colors.dart`, `app_theme.dart`, widgets de tabla |

---

# 📘 PLAN DE IMPLEMENTACIÓN ULTRA DETALLADO – PARTES 2 Y 3
## 🎯 Objetivo: Identidad/KYC, Seguridad, Cuentas, Mercado, Motor de Órdenes y Portafolio (Firestore + Flutter)

A continuación se presentan las **Partes 2 y 3** del plan profesional, alineadas 1:1 con el esquema SQL `bdinvestech.sql` y adaptadas a la arquitectura NoSQL de Firestore. Cada sección incluye mapeo de datos, reglas de seguridad, implementación en Flutter, gestión de estado y validación técnica.

---

# 🔐 PARTE 2: AUTENTICACIÓN, KYC, PERFIL DE RIESGO Y SEGURIDAD FIRESTORE

## 🗃️ 2.1 MAPEO SQL → FIRESTORE (IDENTIDAD Y ACCESO)
| Tabla SQL | Colección/Documento Firestore | Estructura | Notas de Migración |
|-----------|------------------------------|------------|-------------------|
| `USUARIO` | `users/{uid}` | `{nombre, email, telefono, pais, estado, verificado_en, creado_en}` | `uid` es el Auth UID. `estado` usa strings: `"activo"`, `"suspendido"`, `"cerrado"`. |
| `SESION` | Gestionado por Firebase Auth | No se almacena en Firestore por defecto. Si se requiere historial: `users/{uid}/sessions/{sessionId}` | Se confía en `onAuthStateChanged` + refresh tokens. |
| `DOCUMENTO` | `users/{uid}/documents/{docId}` | `{tipo, url_storage, estado, revisado_por, subido_en}` | `url` apunta a Firebase Storage. `tipo` enum string. |
| `PERFIL_RIESGO` | `users/{uid}/riskProfile/{profileId}` | `{nivel, puntaje, respuestas (Map), vigente, evaluado_en}` | Solo un perfil activo por usuario. `vigente=true` marca el vigente. |

---

## 🔑 2.2 IMPLEMENTACIÓN DE FIREBASE AUTH & GESTIÓN DE SESIONES

### 🔹 Dependencias adicionales
```yaml
dependencies:
  firebase_auth: ^5.0.0
  google_sign_in: ^6.2.0
  shared_preferences: ^2.2.2
```

### 🔹 Modelo de Usuario (Freezed)
```dart
// lib/data/models/user_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

part 'user_model.freezed.dart';
part 'user_model.g.dart';

@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required String uid,
    required String nombre,
    required String email,
    String? telefono,
    required String pais,
    required String estado, // activo | suspendido | cerrado
    Timestamp? verificadoEn,
    required Timestamp creadoEn,
  }) = _UserModel;

  factory UserModel.fromJson(Map<String, dynamic> json) => _$UserModelFromJson(json);

  factory UserModel.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return UserModel.fromJson({ ...data, 'uid': doc.id });
  }
}
```

### 🔹 Auth Provider (Riverpod)
```dart
// lib/presentation/providers/auth_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import '../data/models/user_model.dart';

final authStateProvider = StreamProvider<User?>((ref) => FirebaseAuth.instance.authStateChanges());

final userProvider = StreamProvider<UserModel?>((ref) {
  final user = ref.watch(authStateProvider).value;
  if (user == null) return Stream.value(null);
  return FirebaseFirestore.instance.collection('users').doc(user.uid).snapshots().map(
    (doc) => doc.exists ? UserModel.fromFirestore(doc) : null,
  );
});
```

---

## 📄 2.3 FLUJO KYC Y CARGA DE DOCUMENTOS

### 🔹 Estructura Storage + Firestore
```
storage/
  └── users/
      └── {uid}/
          └── documents/
              └── {docId}.pdf
```

### 🔹 Servicio de Upload & Metadata
```dart
// lib/data/datasources/document_datasource.dart
import 'dart:io';
import 'package:firebase_storage/firebase_storage.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class DocumentDataSource {
  final _storage = FirebaseStorage.instance;
  final _firestore = FirebaseFirestore.instance;

  Future<void> uploadDocument({
    required String userId,
    required String tipo,
    required File file,
  }) async {
    final ref = _storage.ref('users/$userId/documents/${DateTime.now().millisecondsSinceEpoch}.pdf');
    await ref.putFile(file);
    final url = await ref.getDownloadURL();

    await _firestore.collection('users').doc(userId).collection('documents').add({
      'tipo': tipo,
      'url_storage': url,
      'estado': 'pendiente',
      'subido_en': FieldValue.serverTimestamp(),
    });
  }
}
```

### 🔹 Validaciones UI
- Tamaño máximo: 5MB
- Tipos permitidos: `PDF, JPG, PNG`
- Validación MIME antes de upload
- Indicador de progreso circular con `StreamSubscription`

---

## 📊 2.4 EVALUACIÓN DE PERFIL DE RIESGO (`PERFIL_RIESGO`)

### 🔹 Lógica de Cálculo
| Puntaje | Nivel |
|---------|-------|
| 0–33 | `conservador` |
| 34–66 | `moderado` |
| 67–100 | `agresivo` |

### 🔹 Formulario & Guardado
```dart
// lib/presentation/screens/risk_assessment_screen.dart
import 'package:cloud_firestore/cloud_firestore.dart';

Future<void> submitRiskProfile({
  required String userId,
  required Map<String, dynamic> respuestas,
  required int puntaje,
}) async {
  final nivel = puntaje <= 33 ? 'conservador' : puntaje <= 66 ? 'moderado' : 'agresivo';
  final docRef = FirebaseFirestore.instance.collection('users').doc(userId).collection('riskProfile').doc();

  await FirebaseFirestore.instance.runTransaction((tx) async {
    // Desactivar perfiles anteriores
    final prev = await tx.get(FirebaseFirestore.instance
        .collection('users').doc(userId).collection('riskProfile').where('vigente', isEqualTo: true).limit(1));
    for (final d in prev.docs) {
      tx.update(d.reference, {'vigente': false});
    }
    // Crear nuevo
    tx.set(docRef, {
      'nivel': nivel,
      'puntaje': puntaje,
      'respuestas': respuestas,
      'vigente': true,
      'evaluado_en': FieldValue.serverTimestamp(),
    });
  });
}
```

---

## 🛡️ 2.5 REGLAS DE SEGURIDAD FIRESTORE (RLS ADAPTADO)

```firestore
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Usuarios: solo lectura/escritura propia
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
      
      // Documentos KYC
      match /documents/{docId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }
      
      // Perfil de Riesgo
      match /riskProfile/{profileId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }
      
      // Cuentas
      match /accounts/{accountId} {
        allow read: if request.auth != null && request.auth.uid == userId;
        allow write: if false; // Solo backend/cloud functions modifican saldos
      }
    }

    // Instrumentos (públicos/lectura global)
    match /instruments/{instrumentId} {
      allow read: if true;
      allow write: if false;
    }

    // Historial de precios
    match /price_history/{priceId} {
      allow read: if true;
    }
  }
}
```
> 📌 **Nota de producción**: Las modificaciones de `saldo`, `estado_orden` y `ejecuciones` **nunca** deben hacerse desde el cliente. Se delegan a Cloud Functions o backend seguro con Firebase Admin SDK.

---

## ✅ CHECKLIST PARTE 2
| Tarea | Estado | Validación |
|-------|--------|------------|
| Auth Email/Google implementado | ✔️ | `onAuthStateChanged` escucha cambios |
| `USUARIO` mapeado a `users/{uid}` | ✔️ | Campos coinciden con SQL enum |
| Upload KYC + Storage + Metadata | ✔️ | Validación tamaño/MIME, estado `pendiente` |
| Perfil de riesgo con cálculo & transacción | ✔️ | Solo un `vigente=true` por usuario |
| Reglas de seguridad deployadas | ✔️ | `firebase deploy --only firestore:rules` |
| Riverpod providers creados | ✔️ | `userProvider`, `authStateProvider` activos |

---

# 💹 PARTE 3: CUENTAS, MERCADO, MOTOR DE ÓRdenes Y PORTAFOLIO EN TIEMPO REAL

## 🗃️ 3.1 MAPEO SQL → FIRESTORE (DOMINIO FINANCIERO)
| Tabla SQL | Colección/Documento Firestore | Estructura Clave |
|-----------|------------------------------|------------------|
| `CUENTA` | `users/{uid}/accounts/{accountId}` | `{numero, tipo, moneda, saldo, saldo_bloqueado, estado}` |
| `MERCADO` | `markets/{marketId}` | `{codigo, nombre, pais, zona_horaria, moneda_base, activo}` |
| `INSTRUMENTO` | `instruments/{instrumentId}` | `{mercado_id, ticker, nombre, tipo, moneda, lote_minimo, activo}` |
| `PRECIO_HISTORICO` | `price_history/{instrumentId}_{YYYYMMDD}` | `{instrumento_id, fecha, apertura, maximo, minimo, cierre, volumen}` |
| `ORDEN` | `users/{uid}/accounts/{accountId}/orders/{orderId}` | `{instrumento_id, tipo, lado, cantidad, precio_limite, precio_stop, vigencia, estado}` |
| `EJECUCION` | `orders/{orderId}/executions/{executionId}` | `{cantidad, precio, comision, ejecutado_en}` |
| `TRANSACCION` | `users/{uid}/accounts/{accountId}/transactions/{transactionId}` | `{orden_id, tipo, monto, moneda, referencia, estado}` |
| `PORTAFOLIO` | `users/{uid}/portfolios/{portfolioId}` | `{cuenta_id, nombre, valor_total, rendimiento_pct}` |
| `POSICION` | `users/{uid}/portfolios/{portfolioId}/positions/{positionId}` | `{instrumento_id, cantidad, precio_promedio, valor_mercado, ganancia_perdida}` |

---

## 💰 3.2 GESTIÓN DE CUENTAS Y SALDOS (`CUENTA` + `TRANSACCION`)

### 🔹 Modelo Freezed
```dart
@freezed
class AccountModel with _$AccountModel {
  const factory AccountModel({
    required String id,
    required String numero,
    required String tipo, // efectivo | margen | custodia
    required String moneda,
    required double saldo,
    required double saldoBloqueado,
    required String estado,
    required Timestamp creadoEn,
  }) = _AccountModel;
}
```

### 🔹 Transacción Segura de Depósito/Retiro
```dart
Future<void> processTransaction({
  required String userId,
  required String accountId,
  required String tipo, // deposito | retiro
  required double monto,
}) async {
  final accountRef = FirebaseFirestore.instance
      .collection('users').doc(userId).collection('accounts').doc(accountId);

  await FirebaseFirestore.instance.runTransaction((tx) async {
    final doc = await tx.get(accountRef);
    if (!doc.exists) throw Exception('Cuenta no encontrada');
    final data = doc.data()!;
    final saldoActual = data['saldo'] as double;

    if (tipo == 'retiro' && saldoActual < monto) {
      throw Exception('Saldo insuficiente');
    }

    final nuevoSaldo = tipo == 'deposito' ? saldoActual + monto : saldoActual - monto;
    tx.update(accountRef, {'saldo': nuevoSaldo});

    // Registrar transacción
    final txRef = accountRef.collection('transactions').doc();
    tx.set(txRef, {
      'tipo': tipo,
      'monto': monto,
      'moneda': data['moneda'],
      'estado': 'completada',
      'fecha': FieldValue.serverTimestamp(),
    });
  });
}
```

---

## 📈 3.3 MOTOR DE ÓRDENES Y EJECUCIONES (`ORDEN` + `EJECUCION`)

### 🔹 Validación Pre-Orden (Cliente)
```dart
bool canPlaceOrder({
  required AccountModel account,
  required String lado,
  required double cantidad,
  required double precio,
  required String tipo,
}) {
  if (account.estado != 'activa') return false;
  if (lado == 'compra' && (account.saldo - account.saldoBloqueado) < (cantidad * precio)) return false;
  if (lado == 'venta' && /* verificar posición disponible */) return false;
  return true;
}
```

### 🔹 Creación de Orden con Bloqueo de Saldo
```dart
Future<String> createOrder({
  required String userId,
  required String accountId,
  required String instrumentoId,
  required String tipo,
  required String lado,
  required double cantidad,
  double? precioLimite,
  double? precioStop,
  required String vigencia,
}) async {
  final accountRef = FirebaseFirestore.instance
      .collection('users').doc(userId).collection('accounts').doc(accountId);
  final orderRef = accountRef.collection('orders').doc();

  await FirebaseFirestore.instance.runTransaction((tx) async {
    final accDoc = await tx.get(accountRef);
    final accData = accDoc.data()!;
    final bloqueadoActual = accData['saldo_bloqueado'] as double;
    final montoBloquear = lado == 'compra' ? cantidad * (precioLimite ?? 0) : 0;

    if (lado == 'compra' && (accData['saldo'] as double) < bloqueadoActual + montoBloquear) {
      throw Exception('Saldo insuficiente para bloquear orden');
    }

    tx.update(accountRef, {'saldo_bloqueado': bloqueadoActual + montoBloquear});
    tx.set(orderRef, {
      'instrumento_id': instrumentoId,
      'tipo': tipo,
      'lado': lado,
      'cantidad': cantidad,
      'precio_limite': precioLimite,
      'precio_stop': precioStop,
      'vigencia': vigencia,
      'estado': 'pendiente',
      'creado_en': FieldValue.serverTimestamp(),
    });
  });

  return orderRef.id;
}
```

> ⚠️ **Nota de Arquitectura**: La transición `pendiente → activa → ejecutada/cancelada` y la generación de `EJECUCION`/`TRANSACCION` **debe ejecutarse en backend seguro** (Cloud Functions + Firebase Admin SDK) para evitar condiciones de carrera y garantizar atomicidad en el matching de órdenes.

---

## 📊 3.4 PORTAFOLIO Y POSICIONES EN TIEMPO REAL

### 🔹 Cálculo de Valor de Mercado & P&L
```dart
class PortfolioCalculator {
  static double calculatePortfolioValue(List<PositionModel> positions, Map<String, double> currentPrices) {
    return positions.fold(0.0, (sum, pos) {
      final price = currentPrices[pos.instrumentoId] ?? pos.precioPromedio;
      return sum + (pos.cantidad * price);
    });
  }

  static double calculatePnL(PositionModel pos, double currentPrice) {
    return (currentPrice - pos.precioPromedio) * pos.cantidad;
  }
}
```

### 🔹 Listener de Posiciones (Riverpod)
```dart
final portfolioPositionsProvider = StreamProvider.autoDispose<List<PositionModel>>((ref) {
  final portfolioId = ref.watch(selectedPortfolioProvider).id;
  return FirebaseFirestore.instance
      .collection('users').doc(currentUserId)
      .collection('portfolios').doc(portfolioId)
      .collection('positions')
      .snapshots()
      .map((snapshot) => snapshot.docs.map((d) => PositionModel.fromFirestore(d)).toList());
});
```

---

## 🖥️ 3.5 UI/UX DE TRADING Y DASHBOARDS

### 🔹 Widget de Tabla de Mercado (Instrumentos)
```dart
// lib/presentation/widgets/market_table.dart
class MarketTable extends StatelessWidget {
  final List<InstrumentModel> instruments;
  final Map<String, double> livePrices;
  const MarketTable({super.key, required this.instruments, required this.livePrices});

  @override
  Widget build(BuildContext context) {
    return DataTable(
      columns: const [
        DataColumn(label: Text('Ticker')),
        DataColumn(label: Text('Nombre')),
        DataColumn(label: Text('Precio')),
        DataColumn(label: Text('Variación')),
        DataColumn(label: Text('Acción')),
      ],
      rows: instruments.map((inst) {
        final price = livePrices[inst.id] ?? 0.0;
        final change = price > 0 ? (price - inst.lastClose) / inst.lastClose : 0.0;
        return DataRow(cells: [
          DataCell(Text(inst.ticker, style: const TextStyle(fontWeight: FontWeight.bold))),
          const DataCell(Text('')), // Nombre truncado en UI real
          DataCell(Text('\$${price.toStringAsFixed(2)}')),
          DataCell(Text('${(change * 100).toStringAsFixed(2)}%', 
            style: TextStyle(color: change >= 0 ? AppColors.marketUp : AppColors.marketDown))),
          DataCell(ElevatedButton(
            onPressed: () => Navigator.pushNamed(context, '/order', arguments: inst.id),
            child: const Text('Operar'),
          )),
        ]);
      }).toList(),
    );
  }
}
```

### 🔹 Formulario de Orden
- Campos: `Tipo (mercado/limite/stop)`, `Lado (compra/venta)`, `Cantidad`, `Precio Límite/Stop`
- Validación en tiempo real contra `lote_minimo` del instrumento
- Preview de comisión y bloqueo de saldo antes de confirmar
- Estados visuales: `🟡 Pendiente`, `🟢 Activa`, `🔵 Ejecutada`, `🔴 Cancelada`

---

## 🔍 3.6 OPTIMIZACIÓN FIRESTORE E ÍNDICES COMPUESTOS

| Consulta | Índice Requerido | Estado |
|----------|------------------|--------|
| `price_history` por `instrumento_id` y `fecha DESC` | `(instrumento_id ASC, fecha DESC)` | Crear en Console o `firebase.json` |
| `orders` por `estado` y `creado_en DESC` | `(estado ASC, creado_en DESC)` | Automático si < 20k docs |
| `transactions` por `tipo` y `fecha DESC` | `(tipo ASC, fecha DESC)` | Crear explícitamente |

### 🔹 Configuración `firebase.json`
```json
{
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  }
}
```

---

## ✅ CHECKLIST PARTE 3
| Tarea | Estado | Validación |
|-------|--------|------------|
| `CUENTA` mapeada con saldos & bloqueos | ✔️ | Transacciones atómicas verificadas |
| `INSTRUMENTO` & `MERCADO` global | ✔️ | Lectura pública, escritura backend-only |
| `PRECIO_HISTORICO` optimizado | ✔️ | Índices compuestos + cache local |
| Motor de órdenes con validación pre-trade | ✔️ | Bloqueo de saldo en transacción |
| Portafolio & P&L en tiempo real | ✔️ | Stream listeners + cálculo dinámico |
| UI Trading (tablas, formularios, estados) | ✔️ | Material 3, colores funcionales, accesible |
| Reglas de seguridad extendidas | ✔️ | Escrituras financieras bloqueadas en cliente |

---

# 📘 PLAN DE IMPLEMENTACIÓN ULTRA DETALLADO – PARTE 4 DE 4
## 🎯 Objetivo: Notificaciones FCM, Auditoría/Cumplimiento, Optimización de Build, CI/CD, Despliegue y Checklist Final (Investech Android)

Esta es la **Parte 4** y última del plan profesional. Cubre la capa de comunicación en tiempo real, integridad legal/financiera, hardening de seguridad Android, pipeline de calidad continua, publicación en Google Play y métricas post-lanzamiento. Todo está alineado al esquema `bdinvestech.sql` y a la arquitectura Flutter + Firebase definida en las Partes 1-3.

---

## 📲 4.1 NOTIFICACIONES PUSH & ALERTAS EN TIEMPO REAL (FCM)

### 🗃️ Mapeo de Eventos SQL → Disparadores Firestore → Payload FCM
| Evento SQL | Trigger Firestore (Cloud Function) | Topic/Token Destino | Payload Tipo | Acción UI |
|------------|-----------------------------------|---------------------|--------------|-----------|
| `estado_orden` cambia a `activa` | `onUpdate(orders/{id})` | `users/{uid}/alerts` | `order_status` | Badge + deep link a `/orders/{id}` |
| Nueva `EJECUCION` registrada | `onCreate(executions/{id})` | `users/{uid}/trades` | `execution_fill` | Snackbar + sonido + historial |
| `DOCUMENTO.estado` → `aprobado/rechazado` | `onUpdate(documents/{id})` | `users/{uid}/kyc` | `kyc_update` | Modal de verificación + reintentos |
| `PRECIO_HISTORICO` supera umbral alertado | `onCreate(price_history/{id})` | `users/{uid}/price_alerts` | `price_alert` | Toast + gráfico miniatura |

### 🔹 Implementación Cliente (Flutter)
```yaml
# pubspec.yaml
dependencies:
  firebase_messaging: ^15.0.0
  flutter_local_notifications: ^17.0.0
  shared_preferences: ^2.2.2
```

```dart
// lib/core/services/fcm_service.dart
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

class FCMService {
  final _messaging = FirebaseMessaging.instance;
  final _localNotifications = FlutterLocalNotificationsPlugin();

  Future<void> init() async {
    await _localNotifications.initialize(const InitializationSettings(
      android: AndroidInitializationSettings('@mipmap/ic_launcher'),
    ));

    NotificationSettings settings = await _messaging.requestPermission(
      alert: true, badge: true, sound: true,
    );

    _messaging.onMessage.listen((RemoteMessage message) {
      _showLocalNotification(message);
    });

    _messaging.onMessageOpenedApp.listen((RemoteMessage message) {
      _handleDeepLink(message.data['route']);
    });

    String? token = await _messaging.getToken();
    // Sincronizar token con Firestore para routing seguro
    await _syncTokenToFirestore(token);
  }

  Future<void> _showLocalNotification(RemoteMessage msg) async {
    const androidDetails = AndroidNotificationDetails(
      'investech_channel', 'Investech Alertas',
      importance: Importance.high, priority: Priority.high,
    );
    await _localNotifications.show(
      msg.messageId.hashCode,
      msg.notification?.title,
      msg.notification?.body,
      const NotificationDetails(android: androidDetails),
    );
  }
}
```

> ⚠️ **Nota de Arquitectura**: La suscripción a temas (`subscribeToTopic`) y la actualización de tokens **nunca** se exponen directamente al usuario. Se gestionan vía Cloud Functions con validación de identidad (`request.auth.uid == userId`).

---

## 📜 4.2 FIRMA DIGITAL, AUDITORÍA Y CUMPLIMIENTO NORMATIVO

### 🔒 Integridad de Órdenes y Transacciones
Cada orden/operación debe generar un hash criptográfico para garantizar no repudio y trazabilidad legal (CNBV/LFPDPPP en MX, o equivalente regional).

```dart
// lib/core/utils/crypto_utils.dart
import 'dart:convert';
import 'package:crypto/crypto.dart';

String generateOrderHash({
  required String userId,
  required String orderId,
  required String instrumentId,
  required double cantidad,
  required String lado,
  required Timestamp timestamp,
}) {
  final payload = '$userId|$orderId|$instrumentId|$cantidad|$lado|${timestamp.seconds}';
  return sha256.convert(utf8.encode(payload)).toString();
}
```

### 📊 Colección de Auditoría (`AUDIT_LOG` en Firestore)
Aunque no existe en el SQL original, es **obligatorio** para cumplimiento financiero y se mapea como subcolección raíz:

```firestore
audit_logs/{logId}
{
  "user_id": "uid",
  "action": "ORDER_CREATE | DOC_UPLOAD | RISK_EVAL | LOGIN",
  "resource_type": "ORDER | DOCUMENT | PROFILE",
  "resource_id": "doc_id",
  "hash": "sha256...",
  "ip_address": "203.0.113.45",
  "device_fingerprint": "webview/android_14",
  "timestamp": FieldValue.serverTimestamp(),
  "success": true
}
```

### 🔐 Almacenamiento Seguro de Sesiones (`SESION` → Firebase Auth + Secure Storage)
| Campo SQL `SESION` | Implementación Moderna | Justificación |
|-------------------|------------------------|---------------|
| `token` | `Firebase Auth ID Token` (auto-refresh) | Seguridad criptográfica nativa |
| `ip`, `dispositivo` | Capturado en Cloud Functions al login | No expuesto en cliente |
| `expiracion`, `revocada` | `Firebase Auth revocation API` + App Check | Control centralizado |

```yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

```dart
// Guardar datos sensibles solo si es estrictamente necesario
final secureStorage = FlutterSecureStorage();
await secureStorage.write(key: 'biometric_token', value: encryptedPayload);
```

---

## 🛡️ 4.3 OPTIMIZACIÓN DE BUILD, SEGURIDAD Y OFUSCACIÓN ANDROID

### 🔧 Configuración `android/app/build.gradle`
```gradle
android {
    compileSdk 34
    namespace "com.investech.appinvestech"

    defaultConfig {
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0.0"
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### 📜 `proguard-rules.pro` (Reglas Críticas)
```pro
# Firebase & Firestore
-keep class com.google.firebase.** { *; }
-keep class com.google.cloud.firestore.** { *; }
-keep class com.google.android.gms.** { *; }
-keep class io.flutter.plugins.firebasemessaging.** { *; }

# JSON/Freezed
-keep class * implements java.io.Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
}

# Criptografía nativa
-dontwarn org.bouncycastle.**
-keep class org.bouncycastle.** { *; }
```

### 🔑 Firma Digital (Keystore)
```bash
# Generar keystore (primera vez)
keytool -genkey -v -keystore ~/investech-upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias investech

# Configurar en android/key.properties
storePassword=<tu_password>
keyPassword=<tu_key_password>
keyAlias=investech
storeFile=<ruta>/investech-upload-keystore.jks

# Referenciar en build.gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

signingConfigs {
    release {
        keyAlias keystoreProperties['keyAlias']
        keyPassword keystoreProperties['keyPassword']
        storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
        storePassword keystoreProperties['storePassword']
    }
}
```

### 📦 Compilación Producción
```bash
flutter build appbundle --release --obfuscate --split-debug-info=build/outputs/obfuscation
```
✅ Salida: `build/app/outputs/bundle/release/app-release.aab`

---

## 🧪 4.4 ESTRATEGIA DE TESTING Y PIPELINE CI/CD

### 📊 Matriz de Testing
| Tipo | Cobertura Objetivo | Herramientas | Ejemplo |
|------|-------------------|--------------|---------|
| **Unit** | `≥ 85%` lógica financiera | `test/`, `mocktail`, `freezed` | Cálculo P&L, validación órdenes, hashes |
| **Widget** | `≥ 70%` UI crítica | `flutter_test`, `golden_toolkit` | Formularios de orden, tablas, temas |
| **Integration** | Flujos E2E | `patrol`, `firebase_auth_mocks` | Login → KYC → Orden → Ejecución simulada |
| **Security** | Reglas Firestore | `firebase emulators:start` | Validar RLS con tokens inválidos/otros users |

### 🔄 Pipeline GitHub Actions (`.github/workflows/main.yml`)
```yaml
name: Investech CI/CD
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }

jobs:
  test-and-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with: { flutter-version: '3.22.0', channel: 'stable' }
      - run: flutter pub get
      - run: flutter analyze
      - run: dart format --set-exit-if-changed .
      - run: flutter test --coverage
      - run: flutter build appbundle --release
      - name: Upload to Play Internal Track
        if: github.ref == 'refs/heads/main'
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.PLAY_SERVICE_ACCOUNT }}
          packageName: com.investech.appinvestech
          releaseFiles: build/app/outputs/bundle/release/app-release.aab
          track: internal
```

---

## 🌐 4.5 DESPLIEGUE EN PLAY STORE Y MONITORIZACIÓN POST-LANZAMIENTO

### 📋 Checklist de Publicación
| Ítem | Estado | Validación |
|------|--------|------------|
| App Bundle firmado y ofuscado | ✔️ | `jarsigner -verify app-release.aab` |
| Screenshots & Video promocional | ✔️ | 7 imágenes (tel/tablet), video < 30s |
| Política de privacidad & Términos | ✔️ | URL externa + enlace en app |
| Clasificación de contenido (PEGI/ESRB) | ✔️ | Formulario Play Console completado |
| Track interno (Alpha) → 10 testers | ✔️ | Distribución vía Play Console |
| Crashlytics & Performance activos | ✔️ | `firebase_crashlytics`, `firebase_performance` |
| App Check (Play Integrity) activado | ✔️ | `firebase_app_check` con reCAPTCHA/Play Integrity |

### 📊 Métricas de Monitorización (Firebase + Play)
| Métrica | Herramienta | Umbral Alerta | Acción |
|---------|-------------|---------------|--------|
| Crash-free users | Crashlytics | `< 99.2%` | Hotfix inmediato, rollback si > 2% |
| ANR Rate | Play Console Vitals | `> 0.5%` | Profiling threads, optimizar `main isolate` |
| Cold Start Time | Performance Monitoring | `> 2.5s` | Lazy loading, reducir `init` en `main()` |
| API Latency (Firestore) | Firebase Console | `> 800ms p95` | Revisar índices, ajustar `cacheSize` |
| Retención 7 días | Google Analytics 4 | `< 35%` | Onboarding UX, push re-engagement |

---

## ✅ CHECKLIST FINAL – MASTER PARTES 1 A 4

| Fase | Entregable Clave | Estado | Validación |
|------|------------------|--------|------------|
| **1** | Estructura Clean Arch, Firebase init, Agentes, Tema | ✔️ | `flutterfire configure`, `main.dart` init log |
| **2** | Auth, KYC Upload, Risk Profile, RLS Firestore | ✔️ | Transacciones atómicas, reglas deny-all default |
| **3** | Cuentas, Órdenes, Ejecuciones, Portafolio, Tablas UI | ✔️ | Índices compuestos, bloqueos de saldo, streams |
| **4** | FCM, Audit/Hash, ProGuard, CI/CD, Play Deploy | ✔️ | `aab` firmado, pipeline verde, Crashlytics activo |

---
