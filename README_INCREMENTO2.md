# 🔐 Requisitos y Credenciales (completar por el administrador)

Este documento describe los requisitos previos y las credenciales necesarias para la operación del sistema.  

---

## 📦 Requisitos del Sistema

- **Docker Desktop**
- **Ngrok**
- **Credenciales OAuth necesarias para n8n:**
  - Google Sheets OAuth API
  - Gmail OAuth
  - PDF Generator API (Convert HTML to PDF)

---

## 🔑 Credenciales Necesarias  
Se tiene que solicitar a un dev, para que te entrega todas las credenciales.

---


# ⭐ Instalación y Configuración de Scripts

Este proceso consta de 3 partes: instalación de **Docker Desktop**, **Ngrok** y configuración de **Apps Script**.  
Los pasos están enumerados de forma continua para asegurar que el proceso sea claro y fácil de seguir.

---

## 🐳 1) Instalación de **Docker Desktop**

Descargar Docker Desktop:  
https://www.docker.com

---

## ▶️ 2) Levantar n8n con Docker

Ejecutar en terminal:

```
docker run -it --rm -p 5678:5678 -v $env:USERPROFILE\n8n_data:/home/node/.n8n n8nio/n8n
```

Acceder en navegador:  
**http://localhost:5678**

---

## 🌐 3) Instalación y configuración de **Ngrok**

1. Descargar Ngrok: https://ngrok.com/download/windows?tab=download  
2. Extraer archivo  
3. Abrir terminal donde está **ngrok.exe**  
4. Crear cuenta en Ngrok  
5. Copiar **Authtoken**  
6. Ejecutar:

```
ngrok config add-authtoken <TOKEN>
```

### 📸 Imagen ejemplo del token
<img width="940" height="73" alt="image" src="https://github.com/user-attachments/assets/5038c450-d154-4a9e-9358-e666ddc5d5ce" />


---

### ➤ Iniciar Ngrok

```
ngrok http 5678
```

Ngrok generará una URL como:  
`https://xxxxx.ngrok-free.dev`

### <img width="940" height="340" alt="image" src="https://github.com/user-attachments/assets/8d382e10-fbf3-4864-8b93-a38e8168fbbe" />

⚠️ **IMPORTANTE:**  
**NO cerrar la consola donde corre Ngrok.**

---

# ✏️ 4) Configurar **Apps Script** en Google Sheets

1. Abrir hoja **base de conocimiento**  
2. Ir a **Extensiones → Apps Script**

### <img width="940" height="206" alt="image" src="https://github.com/user-attachments/assets/8def047c-04f8-4785-bee0-22bb95c20d1a" />
<img width="940" height="229" alt="image" src="https://github.com/user-attachments/assets/ca91a08a-413a-46eb-b77b-cb2c7e6f058a" />
```


---

# 5) Cambiar la URL de Ngrok en Apps Script



### Línea 6
```js
const N8N_WEBHOOK_URL = 'https://NUEVO-LINK.ngrok-free.dev/webhook/evaluaciones';
```

### <img width="940" height="400" alt="image" src="https://github.com/user-attachments/assets/66ec0e65-127e-4a8b-a185-dfe52998ecb1" />
### Línea 324
```js
const response = UrlFetchApp.fetch('https://NUEVO-LINK.ngrok-free.dev/webhook/evaluaciones', options);
```
###<img width="940" height="210" alt="image" src="https://github.com/user-attachments/assets/796dc03e-7fd4-4b71-a8f7-f95648d12b2e" />



---

## 💾 6) Guardar y ejecutar

- Presionar **Ctrl + S**  
- Clic en **▶ Ejecutar**

### <img width="940" height="267" alt="image" src="https://github.com/user-attachments/assets/9a063d87-40e1-44ec-8db4-388fb2a437cb" />
<img width="940" height="206" alt="image" src="https://github.com/user-attachments/assets/27932cc0-e07d-4e35-a735-8217a0779d3d" />


---

# 📬 7) Flujo desde Google Sheets hasta el correo final

1. Abrir hoja **Configuracion_evaluaciones**
2. Completar **id_evaluacion** y **nombre_formulario**
### <img width="752" height="111" alt="image" src="https://github.com/user-attachments/assets/5d7c448a-f0bb-46a6-a3b6-2b24e02dc5cf" />```

4. Ir a hoja **Banco_Preguntas** a la derecha del todo
### <img width="940" height="250" alt="image" src="https://github.com/user-attachments/assets/81eb949d-069e-46ff-a51a-ecfa1aaf890c" />```

6. Marcar columna **Seleccionar**
### <img width="233" height="258" alt="image" src="https://github.com/user-attachments/assets/b2c878d8-8ad2-4f55-ab27-19c2b341fbe9" />```

8. Menú: **Evaluaciones → Crear formulario desde selección**
<img width="940" height="173" alt="image" src="https://github.com/user-attachments/assets/6664827b-f4e1-4ca5-b01c-d4763f7c2447" />
<img width="733" height="239" alt="image" src="https://github.com/user-attachments/assets/4905f47f-7d56-4340-9b35-a513c6fa9754" />

10. Pasaran unos segundos y se generan dos links (editar y estudiante)
<img width="927" height="142" alt="image" src="https://github.com/user-attachments/assets/3dc70465-e884-45d9-a99a-b28909f6fa22" />
<img width="940" height="683" alt="image" src="https://github.com/user-attachments/assets/886ad2bb-9e95-45c9-be08-c56e4acc5907" />

12. Hoja Configuracion_evaluacion se actualizara automáticamente colocando los links 
<img width="940" height="38" alt="image" src="https://github.com/user-attachments/assets/862f1bff-94c0-45fa-be5e-7ef2449438e8" />

14. Usar **Evaluaciones → Renombrar y mover hoja**
<img width="940" height="865" alt="image" src="https://github.com/user-attachments/assets/871a17fc-77a7-4012-ab17-cf245541f9a5" />



# Flujo Completo del Sistema de Evaluaciones  
**Google Sheets → n8n → PDF → Gmail**

A continuación se detalla paso a paso cómo funciona el flujo completo dentro de **n8n**, desde que llegan las respuestas del formulario hasta el envío final del correo con el PDF adjunto.

---

## Paso 1: WebHook – Recepción de datos desde Google Forms  
**Nodo:** *Webhook*

- El flujo comienza con un **trigger Webhook** conectado al formulario generado.  
- Cada vez que un estudiante responde el formulario, las respuestas llegan inmediatamente a este nodo.

---

## Paso 2: Code – Normalizar Respuestas  
**Nodo:** *Normalizar_Respuestas (Code)*

- Procesa las respuestas crudas recibidas desde Google Forms.  
- Separa preguntas y respuestas, limpia caracteres y estandariza textos para permitir la comparación posterior.

---

## Paso 3: Leer banco de preguntas desde Google Sheets  
**Nodo:** *Banco_Preguntas*

- Obtiene las preguntas desde la base de conocimiento.  
- Lee pregunta, alternativas, respuesta correcta y explicación asociada.

---

## Paso 4: Comparar – Determinar la calificación  
**Nodo:** *Comparador*

- Normaliza el texto (mayúsculas/minúsculas, espacios, símbolos).  
- Busca la pregunta del estudiante dentro del banco de preguntas.  
- Compara la **respuesta_usuario** con la **respuesta_correcta**.  
- Determina el **estado**: Correcta o Incorrecta.  
- Si es incorrecta, adjunta la explicación correspondiente.

---

## Paso 5: Desglose – Generación de ID único por pregunta  
**Nodo:** *Desglose*

- Crea un identificador único (**id_detalle**) combinando:  
  `id_intento + número_de_pregunta`  
  Ejemplo: `Probando_2025_1`.

---

## Paso 6: Guardar resultados por pregunta  
**Nodo:** *Update_Desglose*

Se registra cada pregunta evaluada en la hoja **Desgloce_pregunta**, incluyendo:

- estado  
- explicación  
- id_detalle  
- id_intento  
- pregunta y respuesta

---

## Paso 7: Calcular nota del intento  
**Nodo:** *Calcular Nota*

Agrupa todas las preguntas pertenecientes al mismo intento y calcula:

- total_preguntas  
- respuestas_correctas  
- porcentaje_aciertos  
- nota_final  

---

## Paso 8: Guardar resumen del usuario  
**Nodo:** *Update_Resumen_Notas*

Guarda en la hoja **Resumen_Usuario**:

- correo del estudiante  
- nota  
- porcentaje  
- fecha  
- id_intento  

---

## Paso 9: Leer datos nuevamente desde Sheets  
**Nodo:** *Read_Desglose*

- Lee nuevamente los datos del desglose para construir el PDF.

---

## Paso 10: Separar – Limpieza de explicaciones  
**Nodo:** *Separador*

- Si la respuesta es **Correcta**, se elimina la explicación.  
- Si es incorrecta, la explicación se mantiene.

---

## Paso 11: Merge – Unificación del desglose + resumen  
**Nodo:** *Merge*

- Combina todas las preguntas evaluadas con el resumen del intento (nota final y porcentaje).

---

## Paso 12: Editar mensaje – Construcción del HTML del PDF  
**Nodo:** *Edit_Mensaje*

Genera un archivo HTML que incluye:

- Nota final y porcentaje  
- Lista de preguntas  
- Estado Correcta/Incorrecta  
- Explicación para respuestas incorrectas  

Este HTML será la base del PDF.

---

## Paso 13: Convertir HTML a PDF  
**Nodo:** *Convert HTML Content to PDF*

- Envía el HTML a un servicio de conversión.  
- Recibe el PDF codificado en **base64**, listo para enviarse por correo.

---

## Paso 14: Enviar correo con el PDF adjunto  
**Nodo:** *Send a message2 (Gmail)*

Envía al estudiante:

- Asunto personalizado  
- Mensaje de retroalimentación  
- **PDF adjunto** con el análisis completo de su evaluación  


