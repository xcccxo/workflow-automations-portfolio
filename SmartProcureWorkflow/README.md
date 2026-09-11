# Sistema Inteligente de Gestion y Automatizacion de Compras

Este documento detalla la arquitectura, el funcionamiento tecnico y el proposito de cada componente de un flujo de trabajo avanzado implementado en n8n. Este sistema automatiza de extremo a extremo el ciclo de vida de una solicitud de compra corporativa, combinando la captura de datos basada en eventos, el analisis semantico mediante Inteligencia Artificial, la aprobacion asincrona con operadores humanos y la generacion automatizada de documentacion fiscal o administrativa en formato PDF.

## Arquitectura y Desglose Detallado de Modulos

### 1. Fase de Ingesta y Disparadores

* **Google Sheets Trigger:** Actua como el punto de inicio absoluto del sistema. Se encuentra configurado para escuchar eventos de tipo "rowAdded", lo que significa que opera de forma totalmente autonoma cada vez que un empleado envia un formulario de solicitud. Captura variables crudas como el nombre completo del solicitante, el producto seleccionado de un menu desplegable, la cantidad requerida y los detalles o justificaciones adicionales.

### 2. El Modulo de Inteligencia Artificial (Analisis Semantico)

* **Basic LLM Chain y Google Gemini Chat Model:** En lugar de procesar la solicitud de forma lineal, el sistema alimenta a un modelo de lenguaje de Google Gemini con un prompt estructurado que incluye el catalogo autorizado de la empresa. Gemini actua como un analista de compras virtual. Lee la justificacion escrita por el usuario y la compara con el producto que eligio en el formulario.
* **Salida Estructurada (JSON):** El modelo devuelve un objeto JSON estricto que contiene variables de control criticas: la ruta a seguir (Directo o Aclaracion), el resumen del analisis, una advertencia formateada y, si detecta un error de seleccion por parte del usuario, un campo de producto sugerido basado estrictamente en la descripcion aportada.

### 3. El Enrutador de Decisiones (Logica Condicional)

* **Nodo If3:** Evalua la respuesta estructurada del LLM. Si la IA determina que el producto elegido coincide con la justificacion, el flujo se desplaza por la rama inferior (Ruta Directa). Si detecta inconsistencias, incongruencias o descripciones ambiguas, el flujo se desvia hacia la rama superior (Ruta de Aclaracion).

### 4. Ruta de Aclaracion (Rama Superior / Intervencion Inteligente)

* **Send a SolicitudCompra1 (Gmail):** Envia un correo electronico de alerta al gerente o aprobador. El cuerpo del mensaje esta diseñado en HTML avanzado para mostrar una comparativa visual: tacha el producto original erroneo y resalta en un tono de alerta la sugerencia generada por la IA, junto con la advertencia del sistema.
* **Wait1 y Webhooks Asincronos:** Congela la ejecucion del flujo de manera eficiente sin consumir recursos del servidor. El correo incluye botones interactivos (Aprobar Sugerencia, Pedir Aclaracion, Rechazar) enlazados a la URL de reanudacion dinamica de n8n mediante la variable `$execution.resumeUrl` y parametros GET especificos.
* **Nodo Switch (Enrutamiento Multidireccional):** Al hacer clic en el correo, el nodo Wait se despierta y el Switch evalua la accion elegida:
* *Salida 0 (Aprobar Sugerencia):* Pasa a un nodo Edit Fields que limpia la cadena de texto de la sugerencia de la IA utilizando metodos de manipulacion de strings para extraer unicamente el nombre limpio del producto.
* *Salida 1 (Pedir Aclaracion):* Diseñada para enrutar el caso de vuelta al empleado mediante comunicacion secundaria.
* *Salida 2 (Rechazar):* Deriva directamente al cierre del proceso sin compras.



### 5. Ruta Directa (Rama Inferior / Validacion Estandar)

* **Send a SolicitudCompra (Gmail):** Si la IA valido que todo es correcto desde el inicio, envia un correo estandar al aprobador mostrando los datos puros del formulario del usuario, con opciones para aprobar o rechazar directamente.
* **Wait y Nodo If Binario:** Pausa la ejecucion esperando la respuesta del gerente mediante botones similares. Si aprueba, el flujo asciende para unirse al tronco principal de procesamiento. Si rechaza, envia la notificacion de rechazo correspondiente.

### 6. Pipeline de Produccion y Cierre (Fase Final Unificada)

Independientemente de si la solicitud llego por la ruta directa o si fue corregida a traves de la sugerencia de la IA, ambas corrientes convergen gracias a una estrategia de programacion defensiva utilizando el operador logico de cortocircuito (`||`), garantizando que el sistema busque siempre el producto correcto (`Producto Final` o `Producto Requerido` original).

* **Get row(s) Productos (Google Sheets):** Consulta la base de datos central de inventario/productos para extraer el precio unitario y el proveedor asignado al articulo validado.
* **Get row(s) in Proveedores (Google Sheets):** Toma los datos del proveedor obtenido en el paso anterior para recopilar informacion logistica y de contacto.
* **Get row(s) in Historial (Append Row):** Inserta un registro permanente y auditable en la hoja de calculo de historial de compras, guardando la fotografia exacta de la transaccion (usuario, producto aprobado, cantidad, proveedor y estado).
* **Copy file y Update a document (Google Drive y Docs):** Duplica una plantilla maestra preexistente de orden de compra y reemplaza dinamicamente todas las etiquetas de texto con los datos validados de la operacion.
* **Download file (Google Drive):** Convierte el documento editado de Google Docs a un archivo PDF listo para distribucion formal.
* **Send a message1 (Gmail):** Distribuye el PDF generado a los departamentos involucrados o al solicitante como comprobante oficial de adquisicion.
* **Update request state1 (Google Sheets):** Actualiza el estado de la fila original en la hoja de calculo del formulario, marcandola oficialmente como "Aprobado" y cerrando el ciclo operativo.

---

# Intelligent Procurement Management and Automation System

This document details the architecture, technical operation, and purpose of each component in an advanced workflow implemented in n8n. This system automates the entire lifecycle of a corporate purchase request from end to end, combining event-driven data capture, semantic validation via Artificial Intelligence, asynchronous human-in-the-loop approvals, and automated generation of administrative or fiscal documentation in PDF format.

## Architecture and Detailed Module Breakdown

### 1. Ingestion and Trigger Phase

* **Google Sheets Trigger:** Acts as the absolute starting point of the system. It is configured to listen for "rowAdded" events, operating completely autonomously whenever an employee submits a request form. It captures raw variables such as the requester's full name, the product selected from a dropdown menu, the requested quantity, and additional details or justifications.

### 2. The Artificial Intelligence Module (Semantic Analysis)

* **Basic LLM Chain and Google Gemini Chat Model:** Rather than processing requests linearly, the system feeds Google Gemini with a structured prompt that includes the company's authorized catalog. Gemini acts as a virtual procurement analyst. It reads the user's justification and cross-references it with the product chosen in the form.
* **Structured Output (JSON):** The model returns a strict JSON object containing critical control variables: the path to follow (Direct or Clarification), the analysis summary, a formatted warning, and—if it detects an incorrect user selection—a suggested product field based strictly on the provided description.

### 3. The Decision Router (Conditional Logic)

* **If3 Node:** Evaluates the structured response from the LLM. If the AI determines that the chosen product matches the justification, the workflow moves down the lower branch (Direct Path). If it detects inconsistencies, incongruences, or ambiguous descriptions, the flow diverts to the upper branch (Clarification Path).

### 4. Clarification Path (Upper Branch / Intelligent Intervention)

* **Send a SolicitudCompra1 (Gmail):** Sends an alert email to the manager or approver. The message body uses advanced HTML to display a visual comparison: it strikes through the original erroneous product and highlights the AI-suggested product alongside the system warning.
* **Wait1 and Asynchronous Webhooks:** Efficiently pauses workflow execution without consuming server resources. The email includes interactive buttons (Approve Suggestion, Request Clarification, Reject) linked to n8n's dynamic resume URL via the `$execution.resumeUrl` variable and specific GET parameters.
* **Switch Node (Multidirectional Routing):** Upon clicking the email, the Wait node wakes up and the Switch evaluates the chosen action:
* *Output 0 (Approve Suggestion):* Passes to an Edit Fields node that cleans the AI suggestion string using string manipulation methods to extract solely the clean product name.
* *Output 1 (Request Clarification):* Designed to route the case back to the employee via secondary communication.
* *Output 2 (Reject):* Derives directly to the process closure without purchases.



### 5. Direct Path (Lower Branch / Standard Validation)

* **Send a SolicitudCompra (Gmail):** If the AI validated that everything was correct from the start, it sends a standard email to the approver showing the user's raw form data, with options to approve or reject directly.
* **Wait and Binary If Node:** Pauses execution awaiting the manager's response via similar buttons. If approved, the flow ascends to merge into the main processing trunk. If rejected, it triggers the corresponding rejection notification.

### 6. Production and Closing Pipeline (Unified Final Phase)

Regardless of whether a request arrived via the direct path or was corrected through the AI suggestion, both streams converge thanks to defensive programming logic using the short-circuit logical operator (`||`), ensuring the system always seeks the correct product (`Producto Final` or the original `Producto Requerido`).

* **Get row(s) Productos (Google Sheets):** Queries the central inventory/product database to extract the unit price and assigned supplier for the validated item.
* **Get row(s) in Proveedores (Google Sheets):** Takes the supplier data obtained in the previous step to gather logistical and contact information.
* **Get row(s) in Historial (Append Row):** Inserts a permanent, auditable record into the purchase history spreadsheet, capturing an exact snapshot of the transaction (user, approved product, quantity, supplier, and status).
* **Copy file and Update a document (Google Drive and Docs):** Duplicates a pre-existing master purchase order template and dynamically replaces all text tags with the validated operation data.
* **Download file (Google Drive):** Converts the edited Google Docs document into a PDF file ready for formal distribution.
* **Send a message1 (Gmail):** Distributes the generated PDF to involved departments or the requester as an official acquisition receipt.
* **Update request state1 (Google Sheets):** Updates the original row status in the form spreadsheet, officially marking it as "Approved" and closing the operational cycle.
