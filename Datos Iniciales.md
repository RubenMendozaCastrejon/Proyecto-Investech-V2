Actúa como un administrador de base de datos proyecto Investech (que es una plataforma digital de inversiones), que entidades se necesitan para su gestión.

Como administrador de base de datos de **Investech**, las entidades principales del sistema se pueden agrupar en varios dominios. Aquí tienes el modelo entidad-relación completo:Las 13 entidades se organizan en cuatro dominios funcionales:
<img width="1440" height="2368" alt="image" src="https://github.com/user-attachments/assets/905dc32b-2150-4672-8f6a-0d41e043cc43" />

**Identidad y acceso** — `USUARIO`, `SESION`, `DOCUMENTO`, `PERFIL_RIESGO`. Gestiona el ciclo de vida del cliente: registro KYC, autenticación, y evaluación de tolerancia al riesgo regulatoria.

**Cuentas y dinero** — `CUENTA`, `TRANSACCION`. Cada usuario puede tener múltiples cuentas (efectivo, margen, custodia). Las transacciones registran depósitos, retiros y movimientos internos con trazabilidad contable.

**Operativa de mercado** — `ORDEN`, `EJECUCION`. Una orden puede fraccionarse en múltiples ejecuciones (fills parciales). Se distingue el tipo (mercado/límite/stop), el lado (compra/venta) y el estado del ciclo de vida (pendiente → activa → ejecutada → cancelada).

**Instrumentos y mercados** — `INSTRUMENTO`, `MERCADO`, `PRECIO_HISTORICO`, `PORTAFOLIO`, `POSICION`. El catálogo de instrumentos (acciones, ETFs, bonos, cripto) se liga a mercados con zona horaria propia. Las posiciones en cada portafolio se valúan a precio de mercado en tiempo real.

Algunas consideraciones de diseño relevantes para esta base:
- `PRECIO_HISTORICO` crecerá muy rápido; conviene particionarla por `fecha` desde el inicio.
- `EJECUCION` y `TRANSACCION` son candidatas a almacenamiento append-only (sin updates) para mantener el audit trail.
- `PERFIL_RIESGO` debe versionar sus registros, no sobreescribirlos, por requisitos regulatorios.

Las entidades con sus atributos y tipo en forma de tabla para cada una de las entidades

<img width="671" height="738" alt="image" src="https://github.com/user-attachments/assets/2d9725f6-4553-443c-855a-a1a3bf30dece" />
<img width="667" height="694" alt="image" src="https://github.com/user-attachments/assets/83f12fb0-caab-49fe-8cfa-f98cc9f22f8d" />
<img width="594" height="402" alt="image" src="https://github.com/user-attachments/assets/d7146266-cdab-4952-9ea6-bd15b5d0d776" />
<img width="593" height="396" alt="image" src="https://github.com/user-attachments/assets/0b7e34ac-d63a-4d80-b272-e8b12e079f38" />
<img width="594" height="741" alt="image" src="https://github.com/user-attachments/assets/f6afaf7c-0839-42c1-861b-6982292eb618" />
<img width="600" height="759" alt="image" src="https://github.com/user-attachments/assets/11c38603-52f7-4d31-a4d3-4f593fc8a734" />
<img width="594" height="635" alt="image" src="https://github.com/user-attachments/assets/c2c956e4-6007-4e72-bed5-2078440414b0" />
<img width="593" height="334" alt="image" src="https://github.com/user-attachments/assets/f351c83b-c5ce-417a-902d-fb2cbddc8e60" />
