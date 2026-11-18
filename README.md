# DailyPulse — Modelo C4 para el control de dailies en MS Teams

**Assignment 2 · Ejercicio de modelado arquitectónico**
Autor: Santiago Alejandro Jaimes Puerto



---

## 0. Preliminar: lo que entendí del modelo C4

Es la primera vez que trabajo con el modelo C4, así que dejo por escrito lo que entendí de los tres niveles que modelé. El modelo C4 describe un sistema como un mapa con zoom: cada nivel acerca la vista sobre una parte del nivel anterior.

- **C1 · Contexto:** el sistema es una sola caja. Alrededor están las personas que lo usan y los sistemas externos con los que se comunica, sin detalles de tecnología. Responde *qué hace el sistema y quién lo usa*.
- **C2 · Contenedores:** abre esa caja y muestra las piezas que se despliegan o ejecutan por separado (APIs, funciones, bases de datos), con su tecnología y cómo se comunican. Responde *dónde corre cada cosa*.
- **C3 · Componentes:** abre **un** contenedor y muestra sus bloques internos y cómo colaboran para cumplir los requerimientos. Responde *cómo se organiza el código* y sirve de guía para implementarlo.

---

## 1. Lógica de estructuración

El usuario nunca sale de Microsoft Teams: **Teams es la interfaz** y DailyPulse funciona como un bot. Por eso, en el contexto (C1), las personas solo interactúan con Teams, y DailyPulse se comunica con Teams en las dos direcciones: recibe las respuestas del formulario y envía recordatorios, alertas y publicaciones. En contenedores (C2) usé solo servicios administrados de AWS y el menor número de piezas posible. El sistema se activa de dos formas. Si alguien responde en Teams, el mensaje entra por **API Gateway**. Cuando llega la hora del recordatorio o del cierre, lo dispara **EventBridge Scheduler**. En los dos casos se invoca **una sola Lambda en Python** con toda la lógica, y esa Lambda guarda los datos en **DynamoDB**. Tener una sola función simplifica el despliegue y las pruebas, y si más adelante crece se puede dividir sin cambiar el diseño.

Dentro de la Lambda (C3) separé los componentes en tres grupos. Las **entradas** son `Lambda_Handler`, `BotController` y `JobsController`. La **lógica** está en `DailyService` y en dos funciones puras: `MissingController`, que detecta quién no reportó, y `RepeatedTopicDetector`, que detecta un tema repetido 3 veces seguidas. Las **salidas** son `TeamsClient` hacia Teams y `DynamoRepository` hacia la base de datos. Como las reglas de negocio no dependen de Teams ni de AWS, se pueden probar con pruebas unitarias simples, y cada requerimiento del enunciado tiene un componente responsable.

---

## 2. Diagramas

### C1 · Contexto
<img width="682" height="852" alt="C4Model-C1 - Contexto drawio" src="https://github.com/user-attachments/assets/9c2e3017-1891-45cf-b36c-7d4b5450fabb" />


### C2 · Contenedores
<img width="882" height="1242" alt="C4Model-C2 - Contenedores drawio" src="https://github.com/user-attachments/assets/3dcae75e-aeb1-4cbd-bf17-e5ad226b43ab" />



### C3 · Componentes del contenedor *Bot & API*
<img width="1762" height="1352" alt="C4Model-C3 - Componentes (Lambda) drawio" src="https://github.com/user-attachments/assets/17b8fc9d-6714-450d-87df-5aaa02de70ee" />



