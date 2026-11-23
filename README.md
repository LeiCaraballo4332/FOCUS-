# FOCUS

> **Estado del proyecto:** prototipo funcional en desarrollo. La persistencia actual es local; Supabase y el despliegue integrado en Vercel están previstos, pero todavía no están configurados.

## 1. Nombre y descripción

**FOCUS** es un organizador web de actividades académicas y personales que ayuda a planificar el día, priorizar pendientes y reservar tiempo para concentrarse y descansar.

## 2. Problema

Estudiantes y jóvenes suelen gestionar clases, entregas, proyectos y responsabilidades personales en herramientas separadas o de forma improvisada. Esto dificulta identificar qué es urgente, calcular cuánto tiempo requiere cada actividad y dejar espacio para pausas. FOCUS busca reducir esa fricción con una vista diaria sencilla y un plan que se adapta a la disponibilidad indicada.

## 3. Objetivo general

Desarrollar una aplicación accesible y clara para registrar responsabilidades, visualizar el progreso y generar —con ayuda de IA— horarios diarios realistas que consideren prioridades, fechas límite, duración estimada, disponibilidad y descansos.

## 4. Público objetivo

- Estudiantes de secundaria, bachillerato y educación superior.
- Jóvenes que combinan responsabilidades académicas, proyectos y actividades personales.
- Personas que buscan una forma visual y simple de organizar su tiempo y concentración.

## 5. Funcionalidades principales

- Dashboard con tareas pendientes, tiempo enfocado y actividades completadas.
- Crear, editar, buscar, filtrar, completar y eliminar tareas.
- Registro de descripción, categoría, fecha límite, hora sugerida, duración y prioridad.
- Disponibilidad diaria configurable.
- Planificador con IA que considera fechas límite, prioridades, duración, disponibilidad y pausas.
- Validación de horarios para impedir solapamientos, duraciones incorrectas y descansos insuficientes.
- Indicación de actividades que no caben en el tiempo disponible, sin forzar un horario poco realista.
- Acción para reorganizar tareas vencidas o pendientes cuando el plan original ya no funciona.
- Vista semanal y temporizador de concentración de 25 minutos con descansos de 5 minutos.
- Interfaz adaptable a escritorio, tablet y celular.

Las tareas y la disponibilidad se guardan actualmente en `localStorage` del navegador. No hay cuentas ni sincronización entre dispositivos.

## 6. Tecnologías utilizadas

- HTML semántico y CSS responsivo.
- JavaScript nativo en el navegador; no se usa un framework de frontend.
- Node.js 20 o superior y módulos nativos para el servidor HTTP.
- API compatible con OpenAI Chat Completions para la planificación con IA.
- `node:test` para pruebas unitarias; no hay dependencias de npm de terceros.

## 7. Arquitectura general

```text
Navegador
  index.html + styles.css + app.js
  ├── Interfaz, tareas, disponibilidad y temporizador
  ├── localStorage para persistencia local
  └── POST /api/plan
          |
Servidor Node.js (server.js)
  ├── Sirve los archivos estáticos
  ├── Mantiene la clave del proveedor en el entorno del servidor
  └── planner.js: valida entrada, construye el prompt y valida la respuesta
          |
Proveedor de IA compatible con Chat Completions
```

El servidor no almacena las solicitudes de planificación. Antes de enviar información de tareas al proveedor, la interfaz solicita confirmación. La clave de IA nunca se entrega al navegador.

## 8. Estructura del proyecto

```text
.
├── .env.example          # Nombres y valores por defecto de configuración
├── .gitignore            # Excluye secretos y dependencias locales
├── AGENTS.md             # Guía permanente para agentes de programación
├── README.md             # Documentación del proyecto
├── app.js                # Interacciones y estado de la interfaz
├── index.html            # Estructura de la aplicación
├── package.json          # Scripts y requisito de Node.js
├── planner.js            # Validación, prompt y validación del plan generado
├── server.js             # Servidor local y endpoint POST /api/plan
├── styles.css            # Estilos, estados y diseño responsivo
└── test/
    └── planner.test.js   # Pruebas de entrada, prompts y horarios
```

## 9. Instalación y ejecución local

Requisitos: Node.js 20 o superior y npm.

```bash
git clone <URL_DEL_REPOSITORIO>
cd FOCUS-
npm start
```

Abre <http://127.0.0.1:3000>. No se requiere `npm install`: el proyecto no declara dependencias externas.

Para ejecutar las pruebas:

```bash
npm test
```

## 10. Variables de entorno

`.env.example` es una referencia de configuración; Node.js no carga automáticamente archivos `.env`. Define las variables en el entorno de ejecución o mediante el gestor de secretos de tu plataforma.

| Variable | Obligatoria | Valor por defecto | Uso |
| --- | --- | --- | --- |
| `OPENAI_API_KEY` | Para generar planes | — | Credencial del proveedor, solo en el servidor |
| `OPENAI_MODEL` | No | `gpt-4o-mini` | Modelo solicitado |
| `OPENAI_API_URL` | No | `https://api.openai.com/v1/chat/completions` | Endpoint compatible con Chat Completions |
| `PORT` | No | `3000` | Puerto del servidor local |
| `HOST` | No | `127.0.0.1` | Interfaz de red del servidor |

Ejemplo en macOS/Linux:

```bash
export OPENAI_API_KEY="tu_clave"
npm start
```

Ejemplo en PowerShell:

```powershell
$env:OPENAI_API_KEY = "tu_clave"
npm start
```

No escribas claves reales en archivos versionados, en `app.js`, en el HTML ni en solicitudes del navegador. Si la clave no está configurada, las páginas siguen disponibles y el endpoint de planificación devuelve un error explícito.

## 11. Integración prevista con Supabase

Supabase **todavía no está integrado**. La aplicación usa almacenamiento local, no tiene autenticación, base de datos remota ni sincronización multi-dispositivo.

Una integración futura debería contemplar:

1. Supabase Auth para identidad y sesiones.
2. PostgreSQL para perfiles, tareas, disponibilidad y sesiones de concentración.
3. Una columna `user_id` y políticas RLS por usuario en todas las tablas con datos privados.
4. Migraciones versionadas y pruebas para el esquema y las políticas.
5. Migrar gradualmente desde `localStorage`, con una estrategia explícita de importación y manejo de errores.
6. Usar la clave pública/anon solo según el modelo de seguridad de Supabase y RLS. La clave `service_role` debe permanecer exclusivamente en servicios confiables del servidor y nunca exponerse al cliente.

No se deben añadir el SDK ni credenciales de Supabase hasta que la integración se implemente y se documenten sus variables de entorno.

## 12. Integración con inteligencia artificial

El navegador envía las tareas pendientes, sus fechas límite y la disponibilidad al endpoint local `POST /api/plan`. `server.js` valida la solicitud y delega en `planner.js`, que crea instrucciones en español para que el modelo:

- Priorice entregas próximas o vencidas sin modificar sus fechas límite.
- Considere prioridad, duración, horario disponible y descansos.
- Evite solapamientos y no reduzca duraciones para forzar actividades en el horario.
- Devuelva una respuesta JSON con tareas planificadas y tareas que no caben.

Antes de devolver el resultado al cliente, el servidor valida identificadores, duraciones, límites de disponibilidad, descansos y que cada tarea esté planificada o explícitamente pendiente de ubicar. La interfaz permite volver a solicitar un plan para pendientes y tareas vencidas.

La planificación requiere `OPENAI_API_KEY`. Si falta o el proveedor falla, se muestra el error; no se genera un plan simulado. Los nombres, descripciones, categorías, prioridades, fechas límite, duraciones y disponibilidad se envían al proveedor solo tras la confirmación del usuario.

## 13. Despliegue en Vercel

El despliegue completo en Vercel **está pendiente**. El proyecto actual usa `server.js` como servidor Node persistente para desarrollo local; Vercel espera funciones serverless para endpoints como `/api/plan`. No hay todavía `vercel.json` ni una función `/api/plan`.

Para desplegar la aplicación completa:

1. Crear una función serverless compatible con Vercel para `/api/plan`, reutilizando la validación y el prompt de `planner.js`.
2. Configurar el proyecto en Vercel para servir `index.html`, `styles.css` y `app.js` como archivos estáticos y enrutar `/api/plan` a esa función.
3. Añadir `OPENAI_API_KEY` y, si corresponde, `OPENAI_MODEL` y `OPENAI_API_URL` en las variables de entorno de Vercel para los entornos necesarios.
4. No definir secretos con prefijos que los expongan al navegador (por ejemplo, variables públicas del frontend).
5. Probar la página, la función, validación de errores y generación con una credencial válida antes de promover a producción.

Hasta completar esa adaptación, `npm start` es el método de ejecución soportado. Un despliegue estático por sí solo no incluye el endpoint de IA.

## 14. Estado actual y funcionalidades pendientes

**Implementado:** dashboard, tareas CRUD, fechas límite y prioridades, disponibilidad diaria, persistencia local, temporizador, vista semanal, endpoint de IA con validación estricta, reorganización de pendientes y pruebas unitarias del planificador.

**Pendiente:**

- Supabase Auth, base de datos, RLS y sincronización entre dispositivos.
- Función serverless y configuración de despliegue en Vercel.
- Pruebas automatizadas de interfaz y flujos completos en navegador.
- Configuración de disponibilidad por día de la semana y gestión avanzada de calendarios.
- Persistencia remota y sincronización de sesiones de concentración.
- Verificación de la integración con IA usando credenciales reales y el proveedor elegido para producción.
- Tratamiento de autenticación, límites de uso y controles de abuso antes de ofrecer el endpoint públicamente.

Las funcionalidades pendientes no deben describirse como disponibles hasta que se implementen y prueben.
