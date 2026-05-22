 Características Clave
1. Enrutamiento Híbrido Avanzado (Dual-Routing Cost Optimization)
Para evitar el gasto innecesario de infraestructura y tokens en modelos de lenguaje masivos (LLMs), el sistema pre-procesa el texto entrante mediante un Switch por Expresiones Regulares (Regex) optimizado en JavaScript:

Ruta de Saludos (Capa 0): Ataja interacciones triviales (hola, buenos días, ayuda) y las deriva a un modelo ligero y veloz (Cohere Command-Light). Evita consultas a bases de datos y reduce la latencia a milisegundos.

Ruta de Consultas Críticas (Capa Fallback): Deriva las peticiones complejas directamente al pipeline de seguridad.

2. Validación Cruzada de Seguridad Zero-Trust
El mayor riesgo en los bots conversacionales corporativos es la suplantación de identidad (ej. un usuario solicitando información financiera diciendo "Dame las vacaciones de Eric Monné"). Este sistema mitiga el ataque de manera nativa:

Extracción Inmutable: Se captura el message.from.id directamente de las cabeceras de la API de Telegram (un ID numérico único generado por hardware/servidor, imposible de falsear por el usuario).

Búsqueda por Parámetros: Se busca en la tabla de empleados usando el texto ingresado.

Validación de Token: El nodo IF intercepta el flujo y compara el telegram_id registrado contractualmente en la base de datos contra el telegramUserId real del dispositivo que emitió el mensaje.

Aislamiento: Si los IDs no son idénticos, el sistema aborta inmediatamente la ejecución antes de tocar la capa de Inteligencia Artificial generativa, emitiendo una alerta de seguridad.

3. Agente de IA con RAG (Retrieval-Augmented Generation) y Memoria
Una vez superado el control de accesos:

El AI Agent se alimenta dinámicamente con los datos huerfanos del empleado extraídos de MySQL de manera inyectada en el prompt del sistema.

Se integra un Simple Vector Store conectado a embeddings vectoriales para consultar políticas de la empresa, contratos y normativas internas en tiempo real, garantizando respuestas precisas y libres de alucinaciones.

 Stack Tecnológico
Orquestador de Backend: n8n (Arquitectura basada en nodos, flujos asíncronos y gestión avanzada de estados JSON).

Motor de Base de Datos: MySQL (Almacenamiento relacional para la persistencia de datos de empleados y llaves criptográficas/IDs de comunicación).

Capa de Inteligencia Artificial:

Cohere API (Modelos optimizados para procesamiento de lenguaje natural y razonamiento lógico).

Cohere Embeddings (Para la vectorización de la base de conocimientos corporativa).

Interfaz de Usuario: Telegram Bot API (Canal seguro con soporte de cifrado TLS).

 Diseño de la Base de Datos (Esquema SQL de Ejemplo)
Para replicar el entorno de validación del proyecto, se utiliza la siguiente estructura relacional para mapear los identificadores únicos de la plataforma de mensajería con las identidades de la organización (esquema corregido y normalizado):

SQL
CREATE TABLE empleados (  
    id INT AUTO_INCREMENT PRIMARY KEY,  
    nombre VARCHAR(100) NOT NULL,  
    email VARCHAR(150) NOT NULL UNIQUE,  
    departamento VARCHAR(100) NOT NULL,  
    puesto VARCHAR(100) NOT NULL,  
    dias_vacaciones INT NOT NULL DEFAULT 0,  
    salario_neto DECIMAL(10, 2) NOT NULL DEFAULT 0.00,  
    telegram_id BIGINT UNIQUE NOT NULL -- Huella digital inmutable del dispositivo del empleado
);

-- Datos Mock para pruebas de penetración y validación de flujo
INSERT INTO empleados (nombre, email, departamento, puesto, dias_vacaciones, salario_neto, telegram_id)
VALUES 
('Eric Monné', 'eric.monne@chocolatech.com', 'Ingeniería', 'Data Engineer', 14, 45000.00, 6722445528),
('Christian Velasco', 'christian.v@chocolatech.com', 'Desarrollo', 'Software Developer', 12, 42000.00, 9988776655);
 Guía de Despliegue e Integración
Requisitos Previos
Instancia de n8n (Self-hosted o Cloud).

Token de Bot generado mediante BotFather en Telegram.

Credenciales de acceso a una base de datos MySQL (Local o en la nube mediante servicios como Railway/AWS).

API Key de Cohere.

Instrucciones
Descarga el archivo workflow_hr_agent.json de este repositorio.

En tu panel de n8n, crea un flujo nuevo, haz clic en el menú de opciones (esquina superior derecha) y selecciona Import from File.

Configura tus credenciales globales dentro de n8n para:

Telegram Receiver Trigger (Ingresa tu Bot Token).

MySQL Node (Host, puerto, usuario y contraseña de tu DB).

Cohere Chat Model (Tu API Key de Cohere).

Ejecuta el nodo Telegram Trigger en modo de escucha para poblar los esquemas de prueba JSON internos y activa el flujo.

 Consideraciones de Seguridad (Auditoría de Producción)
Mitigación de Inyección SQL: Las llamadas en el nodo MySQL del pipeline principal están parametrizadas y procesadas a través de sintaxis segura nativa de n8n, impidiendo ataques de escape por comillas simples (').

Protección del Contexto (Prompt Injection): El prompt del sistema del Agente de IA utiliza delimitadores estructurales estrictos (###) para separar las instrucciones del rol, los datos del empleado inyectados desde la BD y el input de texto libre del usuario, evitando desbordamientos de instrucciones (Jailbreaking).