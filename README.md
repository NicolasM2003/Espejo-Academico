[readme_manual_de_instalacion_y_despliegue_actualizado.md](https://github.com/user-attachments/files/24173974/readme_manual_de_instalacion_y_despliegue_actualizado.md)
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

## 🔐 2. Gestión de Credenciales y Accesos

El sistema opera con validaciones en múltiples niveles para asegurar la integridad de los datos.

### 2.1 Credenciales de Servicios (Aplicación)

Utilizadas por los flujos de n8n:

- **Google Cloud Platform (OAuth 2.0)**
  - Acceso a Google Sheets
  - Acceso a Google Drive
  - Acceso a Gmail

- **PDF Generator API**
  - Conversión de contenido HTML a PDF

- **Servicio de Correo**
  - OAuth Gmail o SMTP para notificaciones automáticas

> Estas credenciales se configuran directamente en el gestor de credenciales de n8n.

---

### 2.2 Credenciales de Infraestructura (Universidad)

Utilizadas para el despliegue institucional:

- **Acceso VPN UNAV**
  - Usuario genérico: `alumno`
  - Contraseña genérica: `UNAV.1025`

- **Acceso Personal**
  - Usuario y contraseña de Intranet UNAV

> ⚠️ *Las credenciales reales pueden ser anonimizadas en versiones públicas del repositorio.*

---

## 🗂 3. Modelo de Datos – Google Sheets

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

## 🔁 4. Arquitectura de Flujos en n8n

El sistema evolucionó desde un flujo monolítico a una **arquitectura lógica por flujos**, equivalente a micro‑servicios.

### 4.1 Flujo Backend – Evaluación y Retroalimentación

**Responsabilidad:** Procesar evaluaciones individuales.

**Etapas:**
1. Recepción Webhook (Google Forms)
2. Validación de correo autorizado
3. Control de duplicidad de intentos
4. Normalización de respuestas
5. Consulta al Banco de Preguntas
6. Comparación y corrección
7. Registro por pregunta
8. Cálculo de nota
9. Generación HTML
10. Conversión a PDF
11. Almacenamiento en Drive
12. Envío de correo
13. Registro final

---

### 4.2 Flujo Dashboard y Métricas

**Responsabilidad:** Generar estadísticas globales.

- Consolidación de resultados
- Cálculo de métricas
- Generación de reportes CSV/PDF
- Envío a docentes

---

### 4.3 Flujo Frontend Dinámico

**Responsabilidad:** Publicar evaluaciones activas.

- Consulta evaluaciones con estado **Activo = Sí**
- Retorno de JSON con:
  - Nombre evaluación
  - Docente
  - Cantidad de preguntas
  - Link de acceso

---

## 🌍 5. Despliegue en Producción (Máquina Virtual Universitaria)

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

