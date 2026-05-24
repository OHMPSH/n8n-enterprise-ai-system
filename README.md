# n8n-enterprise-ai-system

Asistente de IA corporativo con validación cruzada Zero-Trust. Construido sobre n8n, integrando Telegram, MySQL y Cohere RAG.

![Workflow Diagram](n8n_full_workflow.jpeg)

## Características Clave

### 1. Enrutamiento Híbrido Avanzado (Dual-Routing Cost Optimization)
Para optimizar la infraestructura y el consumo de tokens en modelos de lenguaje (LLMs), el sistema pre-procesa el texto entrante mediante un switch de expresiones regulares (Regex) optimizado en JavaScript:

* **Ruta de Saludos (Capa 0):** Ataja interacciones triviales (saludos, ayuda básica) y las deriva a un modelo ligero y veloz (Cohere Command-Light), reduciendo la latencia a milisegundos.
* **Ruta de Consultas Críticas (Capa Fallback):** Deriva las peticiones complejas directamente al pipeline de seguridad y razonamiento profundo.

### 2. Validación Cruzada de Seguridad Zero-Trust
El sistema mitiga la suplantación de identidad en bots conversacionales corporativos mediante:

* **Extracción Inmutable:** Se captura el `message.from.id` directamente de las cabeceras de la API de Telegram, un identificador numérico único generado a nivel de servidor.
* **Validación de Token:** El nodo IF intercepta el flujo y compara el `telegram_id` registrado contractualmente en la base de datos contra el `telegramUserId` real del dispositivo emisor.
* **Aislamiento:** Si los IDs no coinciden, el sistema aborta la ejecución antes de invocar la capa de IA generativa, emitiendo una alerta de seguridad.

### 3. Agente de IA con RAG y Memoria
Una vez superado el control de accesos:

* El Agente de IA se alimenta dinámicamente con los datos del empleado extraídos de MySQL, inyectados en el prompt del sistema.
* Se integra un *Simple Vector Store* conectado a embeddings vectoriales para consultar políticas de empresa, contratos y normativas internas en tiempo real, garantizando respuestas precisas.

---

## Stack Tecnológico

* **Orquestador de Backend:** n8n (Arquitectura basada en nodos, flujos asíncronos y gestión avanzada de estados JSON).
* **Motor de Base de Datos:** MySQL (Almacenamiento relacional para persistencia de datos de empleados y llaves de comunicación).
* **Capa de Inteligencia Artificial:**
    * Cohere API (Razonamiento lógico y procesamiento de lenguaje natural).
    * Cohere Embeddings (Vectorización de la base de conocimientos corporativa).
* **Interfaz de Usuario:** Telegram Bot API (Canal cifrado con soporte TLS).

---

## Diseño de la Base de Datos

Se utiliza la siguiente estructura relacional para mapear los identificadores únicos de la plataforma de mensajería con las identidades de la organización:

```sql
CREATE TABLE empleados (  
    id INT AUTO_INCREMENT PRIMARY KEY,  
    nombre VARCHAR(100) NOT NULL,  
    email VARCHAR(150) NOT NULL UNIQUE,  
    departamento VARCHAR(100) NOT NULL,  
    puesto VARCHAR(100) NOT NULL,  
    dias_vacaciones INT NOT NULL DEFAULT 0,  
    salario_neto DECIMAL(10, 2) NOT NULL DEFAULT 0.00,  
    telegram_id BIGINT UNIQUE NOT NULL -- Huella digital inmutable del dispositivo
);
'''

Consideraciones de Seguridad (Auditoría de Producción)
Mitigación de Inyección SQL: Las llamadas en el nodo MySQL están parametrizadas y procesadas mediante sintaxis segura nativa de n8n, impidiendo ataques de inyección por caracteres especiales.

Protección del Contexto (Prompt Injection): El sistema utiliza delimitadores estructurales (###) para separar las instrucciones del rol, los datos del empleado inyectados desde la BD y el input de usuario, previniendo ataques de Jailbreaking.