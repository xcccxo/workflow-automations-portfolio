
# AI Lead Automation System

Sistema de automatización inteligente para gestión y clasificación de leads desarrollado con n8n, Google Gemini AI, Google Sheets y Slack.

El proyecto implementa una arquitectura basada en eventos utilizando webhooks para capturar leads en tiempo real desde formularios web. Posteriormente, los datos son analizados mediante inteligencia artificial (Gemini) para evaluar la calidad del lead, asignar un score y clasificarlo automáticamente como “Caliente”, “Tibio” o “Frío”.

Dependiendo de la clasificación obtenida, el sistema ejecuta distintos flujos automatizados:

* Registro de leads en Google Sheets
* Separación automática de spam o leads de baja prioridad
* Notificaciones instantáneas en Slack
* Generación de enlaces dinámicos de contacto vía WhatsApp
* Integración con Calendly para agendamiento de llamadas

## Tecnologías utilizadas

* n8n
* Google Gemini AI API
* Webhooks
* JSON Parsing
* Google Sheets API
* Slack API
* WhatsApp wa.me links
* Calendly

## Funcionalidades principales

* Captura de leads en tiempo real
* Clasificación automática mediante IA
* Lead scoring
* Automatización de workflows
* Routing lógico con nodos IF
* Persistencia de datos
* Notificaciones comerciales automatizadas

## Objetivo del proyecto

Reducir el tiempo de respuesta comercial (“Lead to Speed”) mediante automatización e inteligencia artificial, optimizando la gestión de oportunidades de negocio y el seguimiento de clientes potenciales.

## Arquitectura del sistema

Formulario → Webhook → Gemini AI → JSON Parser → IF Router → Google Sheets / Slack / WhatsApp

## Caso de uso

Desarrollado como solución de automatización comercial para una agencia de marketing y tecnología enfocada en generación y gestión de leads.
<img width="1379" height="912" alt="Captura de pantalla 2026-05-12 a las 22 28 37" src="https://github.com/user-attachments/assets/3bfb486a-b453-4af0-bbbb-cd6d6e27f104" />
<img width="532" height="360" alt="FC04C890-1AA6-4BFF-B220-B8E7D6780B97_4_5005_c" src="https://github.com/user-attachments/assets/d468da89-9556-4583-befd-c8278f775cdc" />
<img width="532" height="360" alt="CD2010A7-9215-46A5-8307-CBC2EDD4DBC1_4_5005_c" src="https://github.com/user-attachments/assets/56cb7390-ec85-4c40-82d0-517e778ffc68" />
<img width="2048" height="1535" alt="B83CB2D7-B464-4FCD-9B04-8ACE1921E939_1_102_o" src="https://github.com/user-attachments/assets/c0b27086-b23a-4925-8dc9-4e5ee78bc240" />


<img width="544" height="359" alt="0BE1FA89-12CA-4497-8CDF-4B986D67ED28_4_5005_c" src="https://github.com/user-attachments/assets/f6be42e3-635c-4cb9-bd30-4c1c68c728b6" />
