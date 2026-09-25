> Registro del uso de IA en PlyOff. Los prompts de este documento están **reconstruidos y parafraseados** a partir del historial de trabajo, ya que la mayor parte del diseño se hizo de forma conversacional. Lo que se documenta es el flujo de trabajo y las decisiones, no cada mensaje literal.

## Flujo de trabajo con IA

### Herramientas y para qué se usan

| Fase | Herramienta | Uso |
|---|---|---|
| Definición de producto | **OpenSpec** (comandos `/opsx:*` y skills en `.claude/`) con un asistente de código | Especificaciones de producto en `openspec/`: visión, capacidades, sala, sesión, identidad, contenido de juegos y motor de juego |
| Diseño técnico y documentación | **Claude (Cowork)** | Análisis de los requisitos de la entrega, decisión del stack, arquitectura, modelo de datos, API, historias, tickets y redacción del README |
| Implementación (Entrega 2 en adelante) | **Cursor** con modelos **Claude Sonnet** | Generación de código a partir de los tickets, siempre con el contexto de las specs |
| Gestión de tareas | **Linear**, con su MCP integrado en Cursor | Los tickets viven en Linear y el asistente los lee directamente; el README es el registro visible |

**Modelos.** En Cursor se trabaja con Claude Sonnet, que es suficiente para la mayoría de las tareas de generación de código.

### Método: desarrollo guiado por especificaciones (Spec-Driven Development)

El flujo completo, de la idea al código, es:

1. **Explorar** (`/opsx:explore`): pensar el problema con la IA antes de comprometer nada.
2. **Proponer** (`/opsx:propose`): genera un cambio con `proposal.md` (qué y por qué), `specs/` (qué debe hacer el sistema, como delta sobre las specs maestras), `design.md` (cómo) y `tasks.md` (pasos de implementación). Este comando **solo planifica**: no toca código.
3. **Convertir las tareas en tickets** de Linear, uno por tarea relevante, con criterios de aceptación.
4. **Implementar** (`/opsx:apply`, o Cursor con Sonnet sobre el ticket): la IA trabaja con la spec y el ticket como contexto, y no con una petición vaga.
5. **Sincronizar y archivar** (`/opsx:sync`, `/opsx:archive`): la spec maestra se actualiza y el cambio queda archivado como historial.

Esto garantiza que la IA nunca implementa a partir de una conversación suelta, sino de artefactos revisados por una persona.

### Contexto que se le da a la IA

- **Specs maestras** en `openspec/specs/` (fuente de verdad del producto): `product-vision`, `product-capability-map`, `core-user-journey`, `local-session-identity`, `local-room`, `local-game-session` y `game-content-framework`.
- **Comandos y skills de OpenSpec** en `.claude/commands/opsx/` y `.claude/skills/`, que fuerzan el límite entre planificar e implementar.
- **Reglas de arquitectura** que se fijarán antes de programar (arquitectura hexagonal, puerto de transporte, ledger de saldo), recogidas en la sección 2 del README.

### Especificaciones creadas con este flujo (evidencia en `openspec/changes/archive/`)

| Fecha | Cambio | Resultado |
|---|---|---|
| 2026-09-02 | `define-product-vision-and-scope` | `product-vision` |
| 2026-09-02 | `define-product-capabilities` | `product-capability-map` |
| 2026-09-02 | `define-core-user-journey` | `core-user-journey` |
| 2026-09-02 | `define-local-session-identity` | `local-session-identity` |
| 2026-09-02 | `define-local-room` | `local-room` |
| 2026-09-02 | `define-local-game-session` | `local-game-session` |
| 2026-09-03 | `define-game-content-framework` | `game-content-framework` |
| En curso | `define-game-runtime` | Contrato del motor de juego (10 preguntas de arquitectura abiertas, resueltas en la sección 2) |

### Ajustes humanos sobre el resultado de la IA

La IA propone y una persona decide. Ejemplos reales de este proyecto:

- **Registro de usuarios.** La IA partía de las specs, que priorizaban jugar sin cuenta. El autor corrigió el rumbo: quiere fidelizar con login social, zona privada e historial. Se resolvió con un muro suave tras la primera partida, que respeta la spec de identidad local.
- **Argumento del modo avión.** La IA defendió que el registro obligatorio rompía el uso en un avión; el autor señaló que la app y el contenido se descargan de todos modos antes de volar. La IA rectificó y el argumento quedó reducido a los casos reales (primera apertura sin conexión y viralidad).
- **Modelo de monetización.** El autor definió las reglas de negocio (todos los jugadores gastan partidas, juegos VIP de compra única con prueba gratuita, promociones desde el CMS) y la IA las tradujo en un ledger y en cambios de spec pendientes.
- **Historias y tickets.** La primera propuesta de la IA era demasiado genérica y no seguía el orden de desarrollo. El autor lo cuestionó y se rehízo como una rebanada vertical concreta, con la base de datos poblada con un script.
- **Verificación del `.gitignore`.** El autor pidió comprobar qué se subía al repo y se detectó una carpeta de borradores que se habría colado.
- **Revisión automática del PR (CodeRabbit).** El pull request recibió 6 hallazgos válidos, que se corrigieron: pasos de instalación en terminales separadas para los procesos de larga duración; un ejemplo de partida de El Infiltrado con menos jugadores que el mínimo; identidad de los participantes desalineada entre Firestore y la API (se introdujo un `participantId` generado por el host); descarga de paquetes VIP sin control de acceso (Storage sin lectura directa y URL firmada solo para `owned`); caché del catálogo sin política por usuario (`Cache-Control: private, no-store`); y ciclo de vida de la identidad de invitado offline sin definir.
- **Límites de la IA.** En el entorno de trabajo no se pudieron renderizar los diagramas Mermaid ni subir al repositorio, por lo que ambas cosas las verifica y realiza el autor.

---

## Índice

1. [Descripción general del producto](#1-descripción-general-del-producto)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Especificación de la API](#4-especificación-de-la-api)
5. [Historias de usuario](#5-historias-de-usuario)
6. [Tickets de trabajo](#6-tickets-de-trabajo)
7. [Pull requests](#7-pull-requests)

---

## 1. Descripción general del producto

**Prompt 1 (registro y fidelización):**
> Ojo, ¿sin cuentas ni registro? Yo sí intentaría tener a la gente fidelizada: login social para agilizar, pero que esté registrada, con su zona privada, sus partidas, rankings y, a futuro, contacto entre jugadores, algo como una red social. A largo plazo me gustaría crecer hacia una plataforma de juegos social, incluso con juegos remotos y streaming.

*Resultado:* la IA señaló el choque con `product-vision` y `local-session-identity` y propuso opciones; se eligió registro incentivado con muro suave tras la primera partida.

**Prompt 2 (juegos del MVP):**
> [Mockups del selector de juegos, información privada y fase social] En el mockup inicial pensamos en juegos sencillos, pero muy "instagrameables" y que puedan hacerse virales. Tiene que fomentar la interacción entre usuarios, ser gracioso pero no molesto. Elige tú, o piensa algo que se pueda convertir en viral rápida e intensamente.

*Resultado:* El Infiltrado como juego principal, La Mentira Perfecta como secundario, y mecánicas pensadas para compartir (momento de revelación, tarjeta de resultados, packs temáticos).

**Prompt 3 (monetización):**
> Juegos "VIP" que se compran una vez, se descargan con conexión y luego se juegan offline; y otro formato de partidas disponibles que se consiguen con packs o viendo anuncios. ¿Qué te parece? Las partidas las gastan todos, y la clave es un buen pack freemium para tener siempre partidas, con promos y regalos desde el CMS.

*Resultado:* modelo freemium con cupo diario, packs, anuncios con recompensa, VIP con prueba gratuita y un registro de movimientos (ledger).

---

## 2. Arquitectura del Sistema

### **2.1. Diagrama de arquitectura:**

**Prompt 1 (stack):**
> Podemos tantear el stack que tengo pensado: una app (había pensado en Flutter, aunque al ser tan específico del hardware igual la haría nativa), una landing en React o Next, un CMS en Angular con PrimeNG, y una API con Cloud Functions y Node 22, todo desplegado en Firebase (Firestore, Storage, notificaciones push, autenticación).

*Resultado:* se confirmó Flutter y toda la plataforma Firebase, y se propuso un monorepo.

**Prompt 2 (transporte):**
> Había pensado en Bluetooth. Piensa que uno de los objetivos es poder jugar en un avión o en sitios sin cobertura.

*Resultado:* decisión central de la arquitectura: BLE con el host como periférico GATT, tras una interfaz de transporte abstracta, con los riesgos documentados.

**Prompt 3 (multiplataforma):**
> Tiene que compartir sala: Android e iOS jugando juntos. El máximo de 8 jugadores por sala me cuadra.

*Resultado:* transporte BLE común a ambas plataformas y límite de 8 jugadores por sala.

### **2.2. Descripción de componentes principales:**

**Prompt 1:** (mismo prompt de stack de la sección 2.1; la descripción de cada componente se generó a partir de esa decisión y se revisó manualmente).

### **2.3. Descripción de alto nivel del proyecto y estructura de ficheros**

**Prompt 1:**
> Este proyecto van a ser 4 repos, ¿no?

*Resultado:* la IA recomendó un monorepo por ser un desarrollador único con una entrega evaluada; el autor lo aceptó.

### **2.4. Infraestructura y despliegue**

**Prompt 1:**
> El repo es `plyoff-project`, el proyecto de Firebase es `plyoff-project`, y los sitios de Hosting son `plyoff-project.web.app` y `cms-plyoff-project.web.app`. Todavía no hay nada desplegado.

*Resultado:* el despliegue se documentó sobre esos sitios, marcando como previsto lo que aún no existe.

### **2.5. Seguridad**

Sin prompt específico: la sección se derivó de las decisiones de 2.1 y de las specs (información privada por dispositivo, servidor como única vía de escritura del saldo) y la revisó el autor.

### **2.6. Tests**

Sin prompt específico: la estrategia se derivó de los escenarios `WHEN/THEN` de las specs de OpenSpec.

---

## 3. Modelo de Datos

**Prompt 1 (usuario registrado):**
> Nombre, apellidos, username, email, proveedor y consentimientos.

*Resultado:* entidad `users` con esos campos y modelo de unicidad del username.

**Prompt 2 (saldo):**
> Los juegos VIP los compran todos los jugadores, y a los que no lo tienen hay que incentivarlos, por ejemplo con un par de partidas gratis para probar.

*Resultado:* entidad `entitlements` con prueba gratuita por juego y usuario.

---

## 4. Especificación de la API

Sin prompt específico: los tres endpoints se derivaron del modelo de datos y de la primera rebanada vertical (catálogo, consumo de partida, historial), y se validó el YAML de OpenAPI.

---

## 5. Historias de Usuario

**Prompt 1:**
> ¿No las ves muy genéricas? ¿No las pondrías en el orden en el que vamos a empezar a desarrollar?

*Resultado:* historias más concretas, con criterios de aceptación medibles, ordenadas según el orden de desarrollo y trazadas a las specs.

---

## 6. Tickets de Trabajo

**Prompt 1:**
> Yo la base de datos la poblaría con un script, y sus reglas e índices.

*Resultado:* el ticket de base de datos incluye las reglas, los índices y un script `seed` idempotente, y los tres tickets forman una rebanada vertical.

---

## 7. Pull Requests

Pendiente: todavía no hay código ni pull requests (se documentarán a partir de la Entrega 2).
