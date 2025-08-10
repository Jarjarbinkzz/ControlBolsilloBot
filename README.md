# ControlBolsilloBot
Bot de Telegram que registra tus gastos, te deja elegir categoría con botones y guarda todo en Google Sheets (listo para entrenar IA y hacer análisis).
✨ Características

Input natural: escribe "Jumbo $12.990 débito (semana)" o "uber 3455 tarjeta".

Botones de categoría (inline keyboard) y anti‑duplicados de eventos.

Persistencia en Google Sheets con timestamp, comercio, método de pago, categoría y nota.

Deployment gratis: Apps Script + Telegram, sin servidores.

Listo para escalar: resumen diario, autocategorización, ingesta Gmail, dashboards.

🧱 Stack

Telegram Bot API

Google Apps Script (GAS) + Google Sheets

Idioma y formato por defecto: es‑CL / CLP.

📁 Estructura sugerida del repo

controlbolsillobot/
├─ src/
│  └─ Code.gs                # Código del bot (GAS)
├─ docs/
│  ├─ troubleshooting.md     # Errores comunes y fixes
│  └─ screenshots/           # Capturas (webhook, executions, etc.)
├─ .gitignore
├─ LICENSE
└─ README.md

.gitignore (mínimo recomendado):

node_modules/
.DS_Store
Thumbs.db
.clasp.json        # si usas clasp y no quieres exponer el scriptId
.env               # por si guardas notas locales

No subas tokens (BOT_TOKEN) ni IDs sensibles.

✅ Requisitos previos

Cuenta de Google (Sheets + Apps Script).

Telegram (móvil o desktop) y haber creado un bot con @BotFather (/newbot).

Una Hoja de cálculo en Google Sheets (nombre sugerido: Gastos Bot), con pestaña gastos y columnas:

timestamp | chat_id | message_id | fecha | monto | moneda | comercio | metodo_pago | categoria | nota

🚀 Quickstart (10 pasos)

Crear bot en @BotFather → guarda el TOKEN (no lo subas al repo).

Crear Sheet → pestaña gastos con las columnas anteriores (zona horaria Archivo → Configuración → America/Santiago).

En la Sheet → Extensiones → Apps Script → pega src/Code.gs (de este repo).

Configura variables (2 opciones):

A. Rápida (en el código):

const BOT_TOKEN = 'TU_TOKEN_AQUI';              // ¡no lo subas a GitHub!
const SPREADSHEET_ID = 'ID_DE_TU_SHEET';
const SHEET_NAME = 'gastos';

B. Segura (recomendada): usar Script Properties y leerlas en tiempo de ejecución.

Apps Script → ⚙️ Project settings → Script properties → añade BOT_TOKEN, SPREADSHEET_ID, SHEET_NAME.

En el código, en vez de constantes, usa:

const PROPS = PropertiesService.getScriptProperties();
const BOT_TOKEN = PROPS.getProperty('BOT_TOKEN');
const SPREADSHEET_ID = PROPS.getProperty('SPREADSHEET_ID');
const SHEET_NAME = PROPS.getProperty('SHEET_NAME') || 'gastos';

Deploy → New deployment → Web app

Execute as: Me

Who has access: Anyone

Copia la Web app URL (termina en /exec).

Setear webhook (desde el mismo editor):

Ejecuta deleteWebhook() y luego setWebhookExec().

setWebhookExec() usa: drop_pending_updates=true y allowed_updates=["message","callback_query"].

Verificar webhook:

Abre https://api.telegram.org/botTU_TOKEN/getWebhookInfo y confirma:

"url" apunta a .../exec (no /dev).

pending_update_count: 0.

sin last_error_message.

Probar en Telegram: /start → uber 3455 tarjeta.

Elegir categoría → se agrega la fila en tu Sheet.

(Opcional) Restringir uso: añade /id, captura tu chat_id y define ALLOWED_IDS = [TU_CHAT_ID].

🧩 Notas técnicas del código (resumen)

Anti‑duplicados por update_id con CacheService (1h).

LockService al escribir en Sheets para evitar duplicados por concurrencia.

Teclado dinámico de categorías (3 por fila) para que puedas editar CATEGORIES sin tocar el builder.

Sanitizado de comillas “inteligentes” y espacios no‑break antes de parsear.

DEBUG con console.log y mensajes in‑chat (desactívalo en prod).

🔐 Seguridad

No subas BOT_TOKEN al repo ni lo pegues en issues.

Revoca tokens viejos con @BotFather /revoke.

Usa Script Properties para secretos.

Activa ALLOWED_IDS para que solo tú (o tu familia) puedan usarlo.

🧰 Troubleshooting (rápido)

"Error 401: deleted_client" al autorizar: crea proyecto nuevo de Apps Script (o cambia el proyecto de GCP) y vuelve a desplegar.

Pantalla "Google hasn’t verified this app": Advanced → Go to <app> (unsafe) o configura OAuth consent screen y añade tu correo como Test user.

getWebhookInfo con /dev o 401 Unauthorized: usa la URL /exec del último deployment. Resetea con deleteWebhook() + setWebhookExec().

Mensajes duplicados: usa drop_pending_updates=true y la deduplicación por update_id (incluida).

El /exec pide login: tu Web App no es público → redeploy con Who has access: Anyone.

No se escribe en la hoja: revisa SPREADSHEET_ID, nombre de pestaña gastos, permisos y LockService.

No aparecen logs: abre Executions (icono reloj) y entra al último doPost.

Más casos en docs/troubleshooting.md.

🗺️ Roadmap

Autocategorización por comercio/keywords.

Resumen diario/semanal por Telegram.

Ingesta automática desde Gmail (avisos de compra del banco).

Dashboard (Looker Studio) y modelo de predicción simple por categoría.

📝 Licencia

MIT (incluye LICENSE).

🤝 Contribuir

Issues y PRs bienvenidos.

Evita subir secretos.

Añade capturas en docs/screenshots/.

📦 Tips opcionales (clasp)

Si quieres llevar el código entre GAS y tu repo con clasp:

npm i -g @google/clasp
clasp login
clasp create --type standalone --title "ControlBolsilloBot"
# copia src/Code.gs al proyecto local y:
clasp push

Puedes decidir no versionar .clasp.json si no quieres exponer el scriptId.

