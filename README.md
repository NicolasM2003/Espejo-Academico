[readme_manual_de_instalacion_y_despliegue_actualizado (1).md](https://github.com/user-attachments/files/24174012/readme_manual_de_instalacion_y_despliegue_actualizado.1.md)
# 📘 Manual de Despliegue y Arquitectura del Sistema de Evaluaciones Automatizadas

Este documento describe en profundidad la **arquitectura técnica**, el **modelo de datos**, los **flujos de automatización** y los **procedimientos de despliegue** del Sistema de Evaluaciones Automatizadas. El sistema fue desarrollado inicialmente en un entorno local y posteriormente desplegado en una **Máquina Virtual Universitaria**, integrando Google Sheets, Google Forms, Google Apps Script y n8n autoalojado.

---

## 🧠 1. Visión General de la Arquitectura

El sistema adopta una arquitectura **desacoplada y orientada a flujos**, donde cada componente cumple una responsabilidad clara:

- **Google Sheets:** Base de datos central del sistema.
- **Google Forms:** Interfaz de captura para estudiantes.
- **Google Apps Script:** Capa de automatización y validación académica.
- **n8n (Self‑Hosted):** Backend de procesamiento, validación, persistencia y notificación.
- **Google Drive:** Almacenamiento de reportes PDF.

Esta separación permite escalabilidad, trazabilidad y control total del proceso evaluativo.

---

## 🔐 2. Requisitos y Credenciales (completar por el administrador)

Este apartado describe los requisitos previos y las credenciales necesarias para la correcta operación del sistema.

### 📦 2.1 Requisitos del Sistema

- **Docker Desktop** (para ejecución local en Windows)
- **Ngrok** (exposición HTTPS del webhook)
- **Credenciales OAuth necesarias para n8n:**
  - Google Sheets OAuth API
  - Gmail OAuth
- **PDF Generator API** (Convert HTML to PDF)

### 🔑 2.2 Credenciales Necesarias

Las credenciales deben ser solicitadas a un desarrollador/administrador del proyecto, ya que contienen accesos a:
- Google Cloud (OAuth Client ID/Secret)
- Gmail (OAuth/SMTP)
- API externa de conversión PDF

> ⚠️ **Nota:** Para repositorios públicos o entregas académicas, se recomienda reemplazar secretos por variables de entorno y/o anonimizar valores sensibles.

---

## ⭐ 3. Instalación y Configuración de Scripts

Este proceso consta de 3 partes: instalación de **Docker Desktop**, configuración de **Ngrok** y configuración de **Apps Script**. Los pasos están enumerados de forma continua para asegurar que el proceso sea claro y fácil de seguir.

### 🐳 3.1) Instalación de Docker Desktop

Descargar Docker Desktop:
- https://www.docker.com

### ▶️ 3.2) Levantar n8n con Docker (Local)

Ejecutar en terminal:

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

Acceder en navegador:
- http://localhost:5678

### 🌐 3.3) Instalación y configuración de Ngrok

1. Descargar Ngrok:
   - https://ngrok.com/download/windows?tab=download
2. Extraer el archivo
3. Abrir una terminal donde se encuentre `ngrok.exe`
4. Crear una cuenta en Ngrok
5. Copiar el **Authtoken**
6. Ejecutar:

```bash
ngrok config add-authtoken <TOKEN>
```

#### ➤ Iniciar Ngrok

```bash
ngrok http 5678
```

Ngrok generará una URL como:
- `https://xxxxx.ngrok-free.dev`

⚠️ **IMPORTANTE:**
- **NO cerrar** la consola donde corre Ngrok mientras se realizan pruebas.

### ✏️ 3.4) Configurar Apps Script en Google Sheets

1. Abrir la hoja base de conocimiento (Google Sheets)
2. Ir a **Extensiones → Apps Script**

### 3.5) Cambiar la URL de Ngrok en Apps Script

En el archivo `Code.gs`, actualizar el webhook según el túnel vigente.

**Línea 6:**

```js
const N8N_WEBHOOK_URL = 'https://NUEVO-LINK.ngrok-free.dev/webhook/evaluaciones';
```

**Línea 324 (Fetch):**

```js
const response = UrlFetchApp.fetch('https://NUEVO-LINK.ngrok-free.dev/webhook/evaluaciones', options);
```

### 💾 3.6) Guardar y ejecutar

- Presionar **Ctrl + S**
- Clic en **▶ Ejecutar**

---

## 📬 4. Flujo desde Google Sheets hasta el correo final (Docente)

1. Abrir hoja `Configuracion_evaluaciones`
2. Completar `id_evaluacion` y `nombre_formulario`
3. Ir a hoja `Banco_Preguntas`
4. Marcar columna `Seleccionar` en las preguntas deseadas
5. Menú: **Evaluaciones → Crear formulario desde selección**
6. Esperar la generación automática de 2 links:
   - Link de edición
   - Link para estudiantes
7. Verificar que `Configuracion_evaluaciones` se actualice automáticamente con los links
8. Usar **Evaluaciones → Renombrar y mover hoja** (cuando aplique)

---

## 🗂 5. Modelo de Datos – Google Sheets

Google Sheets funciona como la **base de datos principal**, estructurada en múltiples hojas especializadas:

### 3.1 Hojas Principales

- **Banco_Preguntas**
  - Pregunta
  - Alternativas
  - Respuesta correcta
  - Explicación

- **Configuracion_evaluaciones**
  - ID de evaluación
  - Nombre de evaluación
  - Correo docente

- **Registro_Evaluaciones**
  - Historial de evaluaciones creadas

- **Autorizados**
  - Correos habilitados para responder evaluaciones

---

### 3.2 Hojas Generadas por el Flujo

- **Registro_Intentos**
  - Estudiante
  - Evaluación
  - Fecha
  - Docente
  - Link PDF (Drive)

- **Desglose_Preguntas**
  - Respuestas correctas (sin explicación)
  - Respuestas incorrectas (con explicación)

- **Resumen_Preguntas**
  - Total preguntas
  - Correctas
  - Porcentaje
  - Nota final

- **Resumen_Evaluaciones**
  - Totales por evaluación
  - Correctas / Incorrectas

- **Errores_Correo**
  - Registro de fallos de envío

- **FormConfig**
  - ID único del formulario
  - Nombre
  - Fecha de creación
  - Función de log histórico

---

## 🔁 6. Flujo Completo del Sistema de Evaluaciones (Google Sheets → n8n → PDF → Gmail)

A continuación se detalla paso a paso cómo funciona el flujo completo dentro de **n8n**, desde que llegan las respuestas del formulario hasta el envío final del correo con el PDF adjunto.

### Paso 1: WebHook – Recepción de datos desde Google Forms
**Nodo:** Webhook

- El flujo comienza con un trigger Webhook conectado al formulario generado.
- Cada vez que un estudiante responde el formulario, las respuestas llegan inmediatamente a este nodo.

### Paso 2: Code – Normalizar Respuestas
**Nodo:** Normalizar_Respuestas (Code)

- Procesa las respuestas crudas recibidas desde Google Forms.
- Separa preguntas y respuestas, limpia caracteres y estandariza textos para permitir la comparación posterior.

### Paso 3: Leer banco de preguntas desde Google Sheets
**Nodo:** Banco_Preguntas

- Obtiene las preguntas desde la base de conocimiento.
- Lee pregunta, alternativas, respuesta correcta y explicación asociada.

### Paso 4: Comparar – Determinar la calificación
**Nodo:** Comparador

- Normaliza el texto (mayúsculas/minúsculas, espacios, símbolos).
- Busca la pregunta del estudiante dentro del banco de preguntas.
- Compara la `respuesta_usuario` con la `respuesta_correcta`.
- Determina el estado: **Correcta** o **Incorrecta**.
- Si es incorrecta, adjunta la explicación correspondiente.

### Paso 5: Desglose – Generación de ID único por pregunta
**Nodo:** Desglose

- Crea un identificador único (`id_detalle`) combinando:
  - `id_intento + número_de_pregunta`
- Ejemplo: `Probando_2025_1`.

### Paso 6: Guardar resultados por pregunta
**Nodo:** Update_Desglose

- Registra cada pregunta evaluada en la hoja `Desglose_Preguntas`, incluyendo:
  - `estado`
  - `explicación`
  - `id_detalle`
  - `id_intento`
  - `pregunta` y `respuesta`

### Paso 7: Calcular nota del intento
**Nodo:** Calcular Nota

- Agrupa todas las preguntas del mismo intento y calcula:
  - `total_preguntas`
  - `respuestas_correctas`
  - `porcentaje_aciertos`
  - `nota_final`

### Paso 8: Guardar resumen del usuario
**Nodo:** Update_Resumen_Notas

- Guarda en la hoja `Resumen_Usuario`:
  - `correo`
  - `nota`
  - `porcentaje`
  - `fecha`
  - `id_intento`

### Paso 9: Leer datos nuevamente desde Sheets
**Nodo:** Read_Desglose

- Lee nuevamente los datos del desglose para construir el PDF.

### Paso 10: Separar – Limpieza de explicaciones
**Nodo:** Separador

- Si la respuesta es **Correcta**, se elimina la explicación.
- Si es **Incorrecta**, la explicación se mantiene.

### Paso 11: Merge – Unificación del desglose + resumen
**Nodo:** Merge

- Combina todas las preguntas evaluadas con el resumen del intento (nota final y porcentaje).

### Paso 12: Editar mensaje – Construcción del HTML del PDF
**Nodo:** Edit_Mensaje

- Genera un HTML con:
  - Nota final y porcentaje
  - Lista de preguntas
  - Estado Correcta/Incorrecta
  - Explicación para incorrectas

### Paso 13: Convertir HTML a PDF
**Nodo:** Convert HTML Content to PDF

- Envía el HTML a un servicio de conversión.
- Recibe el PDF codificado en **base64**, listo para enviarse por correo.

### Paso 14: Enviar correo con el PDF adjunto
**Nodo:** Send a message2 (Gmail)

- Envía al estudiante:
  - Asunto personalizado
  - Mensaje de retroalimentación
  - PDF adjunto con el análisis completo de su evaluación

---

## 🌍 7. Despliegue en Producción (Máquina Virtual Universitaria)

### 5.1 Entorno

- Sistema Operativo: Ubuntu
- Acceso vía VPN UNAV
- Usuario genérico + usuario personal

### 5.2 Despliegue con Docker

```bash
docker run -d \
  --name n8n_prod \
  --restart always \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

---

## 💻 6. Instalación Local (Desarrollo)

### 6.1 Docker

Instalar Docker y verificar versión.

### 6.2 Ejecutar n8n Local

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

Acceso:
```
http://localhost:5678
```

---

### 6.3 Ngrok (Local y Producción)

Ngrok se utiliza para exponer webhooks tanto en desarrollo como en despliegue institucional.

```bash
ngrok http 5678
```

---

## ✏️ 7. Configuración del Cliente (Google Sheets & Apps Script)

- Abrir Google Sheets
- Extensiones → Apps Script
- Configurar:

```js
const N8N_WEBHOOK_URL = 'https://[TU-URL]/webhook/evaluaciones';
```

Ejecutar el script y aceptar permisos iniciales.

---

## 📬 8. Gestión de Formularios (Docente)

1. Completar `Configuracion_evaluaciones`
2. Seleccionar preguntas en `Banco_Preguntas`
3. Menú **Evaluaciones → Crear formulario desde selección**
4. Generación automática de links

---

## ✅ 9. Conclusión

El sistema demuestra una evolución clara desde un entorno local hasta un despliegue institucional, integrando automatización avanzada, control de accesos, trazabilidad completa y generación automática de reportes.

> **Nota académica:** Google Forms actúa únicamente como capa de captura. Toda la lógica de negocio, validación, persistencia y notificación reside en n8n.

