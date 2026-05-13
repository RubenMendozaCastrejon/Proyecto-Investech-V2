INVESTECH
Plataforma Digital de Inversiones

PLAN DE IMPLEMENTACIÓN

Aplicación Móvil Android · Flutter · Firebase

1. Visión General del Proyecto

Investech es una plataforma digital de inversiones diseñada para democratizar el acceso a los
mercados financieros en México y Latinoamérica. La aplicación móvil permitirá a los usuarios gestionar
portafolios de inversión, ejecutar órdenes de compra/venta de instrumentos financieros, monitorear
rendimientos en tiempo real y acceder a herramientas de análisis de mercado desde cualquier
dispositivo Android.
1.1 Objetivos Estratégicos
Desarrollar una aplicación Android nativa con Flutter con una experiencia de usuario de clase
mundial.
Implementar una arquitectura de base de datos robusta sobre Firebase para soportar 13
entidades.
Garantizar la seguridad financiera mediante MFA y cifrado, cumpliendo con regulaciones CNBV.
Lograr tiempos de respuesta menores a 200ms en operaciones críticas.
Soportar hasta 500,000 usuarios concurrentes.
1.2 Alcance del Proyecto
El alcance cubre el diseño, desarrollo, prueba y despliegue de los siguientes módulos:
Identidad: Registro, KYC/AML y perfil de riesgo.
Cuentas: Gestión de inversiones, depósitos y retiros.
Mercados: Datos en tiempo real y análisis técnico.
Trading: Creación y gestión de órdenes (Market, Limit, etc.).
Portafolio: Valuación Mark-to-Market y análisis de diversificación.
•
•
•
•
•

•
•
•
•
•

Página 1

1.3 Stack Tecnológico
Capa Tecnología Propósito
Frontend Móvil Flutter 3.22+ Framework UI multiplataforma
Autenticación Firebase Auth Gestión de identidad
Base de Datos Cloud Firestore NoSQL en tiempo real
Funciones Backend Cloud Functions Lógica de negocio serverless
Estado Riverpod 2.5+ Gestión de estado reactiva
Inyección Dep. Get_it + Injectable Inversión de control

2. Arquitectura del Sistema

2.1 Clean Architecture + MVVM
La aplicación adopta Clean Architecture con el patrón MVVM (Model-View-ViewModel) adaptado para
Flutter con Riverpod para asegurar la testabilidad y mantenibilidad.

Capa de Dominio (Domain Layer)
Contiene entidades puras, casos de uso (use cases) y contratos de repositorios sin dependencias
de frameworks externos.

Capa de Datos (Data Layer)
Implementación de repositorios, fuentes de datos (Firebase/Local) y mapeo de DTOs a entidades
de dominio.

2.2 Estructura de Directorios

investech_app/
├── lib/
│ ├── core/ # Utilidades transversales
│ ├── features/ # Módulos: auth, market, trading, etc.
│ │ ├── domain/ # Entidades y Casos de Uso
│ │ ├── data/ # Modelos y Repositorios
│ │ └── presentation/# Screens y Providers
│ └── shared/ # Widgets compartidos
├── test/ # Pruebas unitarias

Página 2

└── functions/ # Backend (Node.js)

3. Configuración de Firebase
Se establece una infraestructura multi-servicio que combina Firestore, Storage y Cloud Functions en
tres entornos separados: dev, staging y prod.
Seguridad en Firestore
Se implementan reglas estrictas de acceso. Por ejemplo, la colección de usuarios solo permite lectura y
escritura si el auth.uid coincide con el ID del documento.

match /users/{userId} {
allow read: if isOwner(userId) || isAdmin();
allow create: if isAuthenticated() && request.auth.uid == userId;
}

4. Implementación de Datos

4.1 Entidades con Freezed
Se utilizan modelos inmutables para garantizar la integridad de los datos financieros. La entidad de
Usuario incluye campos como nombre, email, país, estado y fecha de creación.
4.2 Gestión de Órdenes
El sistema soporta tipos de órdenes complejos (Mercado, Límite, Stop) y gestiona estados de
transacción (Pendiente, Activa, Ejecutada, Cancelada).
SEGURIDAD: Nunca subir claves de firma (keystores) al repositorio. Utilizar secretos de CI/CD o bóvedas de
seguridad.

Página 3
