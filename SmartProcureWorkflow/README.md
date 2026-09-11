# Sistema Inteligente de Gestión de Compras (Procurement Workflow)

Este repositorio documenta un flujo de trabajo avanzado implementado en n8n que automatiza el ciclo de vida completo de una solicitud de compra corporativa. El sistema combina disparadores basados en eventos, validación semántica mediante inteligencia artificial, aprobaciones asíncronas basadas en humanos en el bucle (human-in-the-loop) y generación automatizada de documentación en formato PDF.

## Arquitectura y Componentes Principales

1. **Disparador de Solicitudes (Google Sheets Trigger):**
El proceso inicia de forma automatizada cada vez que se registra una nueva fila en una hoja de cálculo, capturando los datos crudos enviados por el solicitante (nombre, producto requerido, cantidad y justificación).
2. **Validación Semántica con Inteligencia Artificial (Google Gemini):**
A través de una cadena LLM, el sistema compara el producto seleccionado por el usuario con un catálogo autorizado y analiza la coherencia de su justificación. Gemini clasifica la solicitud determinando si el flujo debe seguir una ruta directa o si requiere una corrección y aclaración.
3. **Enrutamiento Dinámico y Corrección de Errores:**
Dependiendo de la evaluación de la IA, el sistema enruta la solicitud. Si hay una discrepancia entre lo que el usuario seleccionó y lo que describió, la IA sugiere el producto correcto del catálogo y el flujo se desvía hacia una ruta de aclaración con comparativa visual.
4. **Aprobaciones Asíncronas por Correo (Webhooks y Nodos Wait):**
El sistema interactúa con los aprobadores (gerencia) mediante correos electrónicos enriquecidos en HTML que contienen botones de acción directa (Aprobar, Pedir Aclaración, Rechazar). Estos botones están vinculados a las URLs dinámicas de reanudación del nodo de espera en n8n, permitiendo pausar la ejecución de forma segura hasta que se emita una respuesta, sin consumir recursos de servidor.
5. **Generación Documental y Cierre de Ciclo:**
Una vez aprobada la solicitud (ya sea con el producto original o con la sugerencia validada de la IA), el flujo consulta bases de datos de productos y proveedores, registra la operación en un historial, duplica una plantilla maestra en Google Docs, inyecta las variables correspondientes, la convierte a PDF y actualiza el estado final en la hoja de cálculo de origen.

---

# Intelligent Procurement Management System (Procurement Workflow)

This repository documents an advanced workflow implemented in n8n that automates the complete lifecycle of a corporate purchase request. The system combines event-driven triggers, semantic validation through artificial intelligence, asynchronous human-in-the-loop approvals, and automated PDF document generation.

## Architecture and Main Components

1. **Request Trigger (Google Sheets Trigger):**
The process starts automatically whenever a new row is registered in a spreadsheet, capturing the raw data submitted by the requester (name, requested product, quantity, and justification).
2. **Semantic Validation with Artificial Intelligence (Google Gemini):**
Through an LLM chain, the system compares the product selected by the user against an authorized catalog and analyzes the coherence of the justification. Gemini classifies the request, determining whether the workflow should follow a direct path or require correction and clarification.
3. **Dynamic Routing and Error Correction:**
Depending on the AI's evaluation, the system routes the request. If there is a discrepancy between what the user selected and what they described, the AI suggests the correct product from the catalog, diverting the flow to a clarification path featuring a visual comparison.
4. **Asynchronous Email Approvals (Webhooks and Wait Nodes):**
The system interacts with approvers (management) via HTML-enriched emails containing direct action buttons (Approve, Request Clarification, Reject). These buttons link to the dynamic resume URLs of n8n's wait nodes, securely pausing execution until a response is triggered without consuming server resources.
5. **Document Generation and Cycle Closure:**
Once the request is approved (either with the original product or the AI's validated suggestion), the workflow queries product and supplier databases, logs the operation in a history sheet, duplicates a master template in Google Docs, injects the corresponding variables, converts it to PDF, and updates the final status in the source spreadsheet.
