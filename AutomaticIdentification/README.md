# Sistema Automatizado de Auditoria y Recepcion Logistica en n8n

Este proyecto implementa un flujo de trabajo integral en n8n disenado para automatizar la recepcion, auditoria y control de inventario de paquetes en tiempo real. El sistema procesa fotografias de guias de envio y etiquetas comerciales tomadas en puerta de almacen, extrayendo datos mediante vision artificial y decodificacion optica para contrastarlos contra las ordenes de compra registradas en Google Sheets a traves de un modelo de lenguaje (LLM).

---

## Aspectos Tecnicos Destacados

El nucleo de procesamiento visual del flujo combina dos enfoques complementarios para garantizar una tasa de lectura maxima en entornos industriales:

1. **OCR de la Comunidad (Tesseract):** 
   Se integro un nodo comunitario de Tesseract en n8n para extraer texto plano, direcciones, nombres de transportistas y descripciones impresas en etiquetas deterioradas o formatos no estructurados.

2. **Lector Personalizado de Codigo de Barras y QR (`n8n-nodes-barcode-parser`):** 
   Debido a las limitaciones que presentan las bibliotecas genericas de Node.js al procesar codigos unidimensionales (1D) en entornos virtualizados, se desarrollo un nodo personalizado creado especificamente para este flujo. Este modulo implementa un pipeline estricto de decodificacion:
   - Deteccion prioritaria 2D (QR Code y DataMatrix) para evitar falsos positivos.
   - Decodificadores dedicados para simbologias 1D industriales (Code 128, EAN-13 y estandares tipo GS1-128 con identificadores de aplicacion).
   - Eliminacion de lectores permisivos (como Codabar o ITF no verificado) para erradicar capturas de ruido visual.

---

## Arquitectura del Flujo

El pipeline se ejecuta secuencialmente a traves de las siguientes etapas:

1. **Ingreso (Webhook):** Captura la peticion HTTP enviada desde la aplicacion web o dispositivo movil con la imagen del paquete y metadatos del receptor.
2. **Procesamiento Paralelo:** La imagen se envia simultaneamente al nodo OCR de Tesseract y al nodo personalizado de decodificacion de codigos de barras/QR.
3. **Sincronizacion (Merge):** Unifica los datos visuales y decodificados en un unico elemento de ejecucion.
4. **Consulta de Base de Datos:** Consulta en Google Sheets los pedidos activos y agrega los registros en un arreglo estructurado.
5. **Auditoria Logica con LLM (Gemini):** Realiza la validacion cruzada comparando la orden de compra, el SKU y las cantidades fisicas recibidas contra el sistema, aplicando una regla inquebrantable de doble factor (existencia de la orden + coincidencia exacta del producto).
6. **Enrutamiento Inteligente (Switch):** Desvia el flujo segun el dictamen del LLM en tres ramas operativas.

---

## Matriz de Decisiones y Salidas

- **Salida 0 (Aprobado - Coincidencia):**
  Aplica para entregas completas, parciales o excedentes autorizados. Envia una notificacion por correo, implementa un buffer de espera para la maniobra fisica, actualiza la bitacora de recepcion de paqueteria (marcando fecha, hora y responsable) e incrementa el stock fisico en el catalogo maestro de almacen.

- **Salida 1 (Discrepancia / Producto Incorrecto):**
  Se activa si la orden de compra existe en el sistema pero el producto fisico recibido (SKU o descripcion) no corresponde a lo solicitado. Envia una alerta inmediata a Compras y Almacen, instruye el aislamiento del paquete en cuarentena y bloquea cualquier alteracion al stock.

- **Salida 2 (Sin Coincidencia / Paquete No Registrado):**
  Se activa cuando la guia o paquete no cuenta con respaldo documental ni orden de compra activa. Envia una alerta de rechazo para que el personal en puerta no reciba el bulto del transportista.

---

## Requisitos de Instalacion

- Instancia de n8n desplegada en Docker o localmente.
- Nodo comunitario de Tesseract instalado en el entorno de n8n.
- Modulo personalizado `n8n-nodes-barcode-parser` compilado y montado en el directorio de extensiones personalizadas de n8n (`/home/node/.n8n/custom/`).
- Credenciales configuradas para Google Sheets, Google Gemini Chat Model y Gmail.

---
---

# Automated Logistics Receiving and Audit System in n8n

This project implements an end-to-end automated workflow in n8n designed to audit, receive, and manage warehouse inventory in real time. The system processes photographs of shipping labels and commercial waybills captured at the warehouse dock, extracting data via computer vision and optical decoding to cross-reference them against active purchase orders in Google Sheets using a Large Language Model (LLM).

---

## Key Technical Highlights

The visual processing core of this workflow combines two distinct mechanisms to achieve high reliability in industrial environments:

1. **Community OCR Node (Tesseract):** 
   A community-contributed Tesseract node was integrated into n8n to extract plain text, carrier details, addresses, and physical descriptions from damaged labels or unstructured documents.

2. **Custom-Built Barcode and QR Reader (`n8n-nodes-barcode-parser`):** 
   Because standard Node.js decoding libraries often struggle with 1D barcode scanline recognition inside containerized environments, a custom node was engineered specifically for this pipeline. This module incorporates a strict decoding hierarchy:
   - Priority 2D detection (QR Code and DataMatrix) to prevent false positives.
   - Dedicated readers for industrial 1D symbologies (Code 128, EAN-13, and GS1-128 formats using Application Identifiers).
   - Exclusion of overly permissive readers (such as unvalidated Codabar or ITF) to eliminate false detections from visual noise.

---

## Pipeline Architecture

The workflow progresses through the following stages:

1. **Ingestion (Webhook):** Listens for incoming HTTP POST requests from the receiving web application or mobile device, capturing the image buffer and intake metadata.
2. **Parallel Processing:** Dispatches the image simultaneously to the community Tesseract OCR node and the custom barcode/QR decoder.
3. **Synchronization (Merge):** Unifies textual OCR output and decoded barcode payloads into a single execution item.
4. **Database Query:** Fetches active purchase orders from Google Sheets and aggregates all rows into a unified structured array.
5. **Intelligent Audit via LLM (Gemini):** Cross-checks order IDs, tracking numbers, SKUs, and physical quantities against expected inventory data, enforcing a strict two-factor verification rule (valid order reference + matching physical product).
6. **Rule-Based Routing (Switch):** Segregates executions into three dedicated operational paths based on the audit verdict.

---

## Decision Routing and Outcomes

- **Output 0 (Approved - Match):**
  Triggered on complete deliveries, partial deliveries, or authorized over-deliveries. Sends a delivery confirmation email, executes an operational wait buffer, updates the shipping intake sheet (recording timestamp, courier, and receiver), and increments available stock in the master inventory catalog.

- **Output 1 (Discrepancy / Incorrect Item):**
  Triggered when a purchase order exists in the system but the received item (SKU or description) does not match what was requested. Dispatches an urgent alert to Procurement and Warehouse Management, instructs physical isolation in a quarantine area, and halts all inventory modifications.

- **Output 2 (No Match / Unregistered Package):**
  Triggered when the package or tracking number has no matching active purchase order in the system. Emits a rejection notification advising dock staff to decline the package from the carrier.

---

## Prerequisites and Setup

- An active n8n instance running via Docker or on-premise.
- Community Tesseract node installed in the n8n instance.
- Custom `n8n-nodes-barcode-parser` module compiled and mounted into the custom nodes directory (`/home/node/.n8n/custom/`).
- Valid credentials configured for Google Sheets, Google Gemini Chat Model, and Gmail.
