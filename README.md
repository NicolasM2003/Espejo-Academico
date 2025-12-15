
# 📘 Manual de Despliegue y Arquitectura del Sistema de Evaluaciones Automatizadas

Este documento describe en profundidad la **arquitectura técnica**, el **modelo de datos**, los **flujos de automatización** y el **proceso completo de despliegue** del Sistema de Evaluaciones Automatizadas.

El sistema fue desarrollado inicialmente en un **entorno local** y posteriormente desplegado en una **Máquina Virtual Universitaria**, integrando Google Sheets, Google Forms, Google Apps Script, n8n autoalojado y frontends web independientes.

---

## 🧠 1. Visión General de la Arquitectura

El sistema adopta una arquitectura **desacoplada y orientada a flujos**, donde cada componente cumple una responsabilidad clara:

- **Google Sheets:** Base de datos central del sistema.
- **Google Forms:** Interfaz de captura de evaluaciones para estudiantes.
- **Google Apps Script:** Capa de automatización y validación académica.
- **n8n (Self‑Hosted):** Backend de procesamiento, validación, persistencia y notificación.
- **Google Drive:** Almacenamiento y distribución de reportes PDF.
- **Frontends Web:** Interfaces HTML independientes para acceso y visualización.

Esta separación permite escalabilidad, trazabilidad, control de accesos y facilidad de mantenimiento.

---

## 🔐 2. Requisitos y Credenciales (completar por el administrador)

### 📦 2.1 Requisitos del Sistema

- **Docker Desktop** (entorno local)
- **Docker Engine** (entorno servidor Linux)
- **Ngrok** (exposición HTTPS de webhooks)
- **Credenciales OAuth necesarias para n8n:**
  - Google Sheets OAuth API
  - Gmail OAuth
- **PDF Generator API** (HTML → PDF)

### 🔑 2.2 Credenciales Necesarias

Las credenciales deben ser solicitadas a un desarrollador/administrador del proyecto:

- Google Cloud (OAuth Client ID / Secret)
- Gmail (OAuth / SMTP)
- API externa de conversión PDF
- Credenciales de acceso VPN e infraestructura universitaria

> ⚠️ Para entregas académicas o repositorios públicos, los valores sensibles deben ser anonimizados.

---

## ⭐ 3. Instalación y Configuración (Entorno Local)

Esta sección describe la configuración utilizada durante el **desarrollo y pruebas locales**.

### 🐳 3.1 Instalación de Docker Desktop

Descargar desde:

- [https://www.docker.com](https://www.docker.com)

---

### ▶️ 3.2 Levantar n8n en Local

```bash
docker run -d \
  --name n8n_local \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

Acceso:

- [http://localhost:5678](http://localhost:5678)

---

### 🌐 3.3 Configuración de Ngrok

Ngrok se utiliza para exponer el webhook de n8n a Google Apps Script y Google Forms.

```bash
ngrok http 5678
```

Se obtiene una URL del tipo:

- [https://xxxxx.ngrok-free.dev](https://xxxxx.ngrok-free.dev)

⚠️ No cerrar la consola de Ngrok durante las pruebas.

---

### ✏️ 3.4 Configuración de Apps Script

En el archivo `Code.gs`:

```js
const N8N_WEBHOOK_URL = 'https://NUEVO-LINK.ngrok-free.dev/webhook/evaluaciones';
```

Guardar y ejecutar el script para otorgar permisos.

---

## 🗂 4. Modelo de Datos – Google Sheets

Google Sheets actúa como la **base de datos lógica del sistema**, compuesta por múltiples hojas especializadas.

### 4.1 Hojas Principales

- **Banco\_Preguntas**: preguntas, alternativas, respuestas correctas y explicación.
- **Configuracion\_evaluaciones**: ID de evaluación, nombre, categoría y correo docente.
- **Registro\_Evaluaciones**: historial de evaluaciones creadas.
- **Autorizados**: correos habilitados para responder.

### 4.2 Hojas Generadas por el Flujo

- **Registro\_Informes**: resultados finales y links PDF.
- **Desglose\_Preguntas**: detalle por pregunta (correctas / incorrectas).
- **Resumen\_Preguntas**: métricas por intento.
- **Resumen**: consolidado por evaluación.
- **Logs\_Errores\_Correo**: auditoría de fallos.
- **FormConfig**: log automático de formularios creados.

---

## 🔁 5. Flujo Principal de Evaluaciones (n8n)

El flujo fue **iterativamente ampliado** hasta llegar a su versión productiva final.

### Etapas principales:

1. Recepción del Webhook desde Google Forms.
2. Validación de duplicados y correos autorizados.
3. Normalización de datos y respuestas.
4. Comparación con Banco\_Preguntas.
5. Registro por pregunta (desglose).
6. Cálculo de nota y métricas.
7. Generación de HTML → PDF.
8. Almacenamiento del PDF en Google Drive.
9. Generación y guardado de links en Google Sheets.
10. Envío de correos a estudiante y docente.
11. Registro final y auditoría.

---

## 📊 6. Flujos Complementarios

### 6.1 Flujo de Dashboard

- Flujo independiente dentro del mismo workflow.
- Lee datos desde `Resumen` y `Registro_Informes`.
- Calcula estadísticas agregadas.
- Genera CSV, gráficos y PDF.
- Envía reportes automáticos a docentes.

---

### 6.2 Flujo Frontend (Evaluaciones Activas)

- Flujo independiente orientado al frontend.
- Consulta `Configuracion_evaluaciones`.
- Filtra evaluaciones con estado **Activo = Sí**.
- Retorna JSON con:
  - ID evaluación
  - Nombre
  - Docente
  - Cantidad de preguntas
  - Link de acceso

---

## 🌍 7. Despliegue Final en Máquina Virtual Universitaria

Esta sección describe el **último paso del proyecto**, correspondiente al despliegue en infraestructura universitaria.

### 7.1 Entorno de Despliegue

- Sistema Operativo: Linux (Ubuntu)
- Acceso vía **VPN institucional**
- Credenciales de Intranet para VPN
- Usuario dedicado para acceso SSH al servidor

---

### 7.2 Levantamiento de n8n en Producción

```bash
docker run -d \
  --name n8n_prod \
  --restart always \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

---

### 7.3 Despliegue de Frontends Web

Durante el despliegue final se levantaron **dos frontends independientes**, cada uno ejecutándose en un puerto distinto del servidor:

- **Frontend de acceso a evaluaciones (principal)**

  - Servido desde un puerto dedicado (8080).
  - Contiene archivos HTML/JS para que los estudiantes visualicen evaluaciones activas.
  - Boton clickiable para que el estudiante que lo lleva a google form para responderlo

- **Frontend HTML (No funcional, la vista esta correcta pero en cuanto a informacion y creacion de formulario no disponible)**

  - Servido desde un segundo puerto (8090).
  - Permite acceso a vistas complementarias y enlaces generados.

Cada frontend contiene sus propios archivos HTML estáticos dentro del servidor y se comunica con n8n mediante HTTP.

---

### 7.4 Exposición de Servicios

- n8n expuesto mediante puerto `5678`.
- Webhooks accesibles mediante Ngrok o IP institucional.
- Frontends accesibles mediante IP del servidor y puertos definidos.

