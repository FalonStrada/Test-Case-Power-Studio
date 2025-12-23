# Test Case Generator

Herramienta CLI que automatiza la generación de casos de prueba desde historias de usuario en Jira, aplicando técnicas formales de testing (Pairwise, Boundary Value Analysis, Decision Tables) y sincronizándolos con Qase.

## ¿Qué hace?

Convierte historias de usuario de Jira en casos de prueba completos automáticamente:

```bash
npm run dev PROJ-123
```

**Resultado:** Casos de prueba generados aplicando técnicas formales de QA, creados en Qase y vinculados a la historia original.

## Casos de uso

- **Testing basado en técnicas formales**: Aplica automáticamente Pairwise, Boundary Value Analysis y Decision Tables
- **API Testing especializado**: Heurísticas completas para endpoints (métodos HTTP, status codes, autenticación, seguridad)
- **Cobertura optimizada**: Genera casos positivos, negativos y de borde sin redundancia
- **Documentación técnica**: Test cases concisos enfocados en QUÉ se prueba, no en pasos de UI
- **Trazabilidad**: Vinculación automática con historias de usuario en Jira

## Requisitos

- Node.js 18+
- Cuenta de Jira Cloud con API access
- API Key de OpenAI
- Cuenta de Qase

## Instalación

```bash
# Clonar e instalar dependencias
git clone <repo-url>
cd test-case-generator
npm install

# Configurar credenciales
cp .env.example .env
```

### Configuración del .env

```bash
# Jira
JIRA_BASE_URL=https://tu-empresa.atlassian.net
JIRA_EMAIL=tu-email@empresa.com
JIRA_API_TOKEN=tu_token_jira

# OpenAI
OPENAI_API_KEY=sk-tu-clave-openai
OPENAI_MODEL=gpt-4-turbo

# Qase
QASE_API_TOKEN=tu_token_qase
QASE_PROJECT_CODE=PROJ

# Opcionales
LOG_LEVEL=info
NODE_ENV=development
```

### Obtener credenciales

**Jira API Token:**

- https://id.atlassian.com/manage-profile/security/api-tokens

**OpenAI API Key:**

- https://platform.openai.com/api-keys

**Qase API Token:**

- https://app.qase.io/user/api/token

## Uso

### Una User Story

```bash
npm run dev PROJ-123
```

### Múltiples User Stories (batch)

```bash
# Desde argumentos
npm run dev PROJ-123 PROJ-456 PROJ-789

# Desde archivo
npm run dev --file issues.txt
```

**Formato de `issues.txt`:**

```
PROJ-123
PROJ-456
PROJ-789
# Comentarios con # son ignorados
PROJ-790
```

### Ejemplo de salida (batch)

```bash
npm run dev PROJ-123 PROJ-456

======================================================================
📦 PROCESAMIENTO EN BATCH
   Total User Stories: 2
   Issues: PROJ-123, PROJ-456
======================================================================

──────────────────────────────────────────────────────────────────────
📄 User Story 1/2: PROJ-123
──────────────────────────────────────────────────────────────────────

[Proceso completo de PROJ-123...]

⏸️  Pausa de 3 segundos antes de la siguiente US...

──────────────────────────────────────────────────────────────────────
📄 User Story 2/2: PROJ-456
──────────────────────────────────────────────────────────────────────

[Proceso completo de PROJ-456...]

======================================================================
📊 RESUMEN DEL BATCH
======================================================================
⏱️  Duración total: 47.32s
📈 User Stories procesadas: 2
✅ Exitosas: 2
❌ Fallidas: 0

✅ User Stories exitosas:
   - PROJ-123
   - PROJ-456
======================================================================
```

### Producción

```bash
npm run build
npm start PROJ-456
```

### Ver logs detallados

```bash
# Logs en tiempo real
tail -f logs/combined.log

# Solo errores
tail -f logs/error.log
```

## 📊 ¿Cuántas User Stories procesa?

**Por defecto: Procesa las User Stories que le pases como argumento**

### Opciones de ejecución:

1. **Una US individual**

   ```bash
   npm run dev PROJ-123
   ```

   Procesa: 1 User Story

2. **Múltiples US en batch (CLI)**

   ```bash
   npm run dev PROJ-123 PROJ-456 PROJ-789
   ```

   Procesa: 3 User Stories

3. **Batch desde archivo**
   ```bash
   npm run dev --file sprint-12-stories.txt
   ```
   Procesa: Todas las US en el archivo (sin límite)

### Recomendaciones:

- **Sprints pequeños**: 5-10 US en un batch
- **Sprints grandes**: Dividir en múltiples ejecuciones o usar archivo
- **Rate limiting**: El tool incluye pausas de 3s entre US para evitar limitaciones de APIs

### Tiempo estimado por US:

- Simple: ~18-22 segundos
- Media: ~22-28 segundos
- Compleja: ~28-35 segundos

**Ejemplo:** 10 US en batch = ~4-5 minutos

## 📊 ¿Cuántos test cases genera por US?

**Rango típico: 8-15 test cases por historia**

La cantidad exacta depende de la complejidad:

| Complejidad | TCs esperados | Ejemplo                    |
| ----------- | ------------- | -------------------------- |
| Baja        | 8-10          | Simple CRUD endpoint       |
| Media       | 10-12         | Login con validaciones     |
| Alta        | 12-15         | Flujo de pago multi-estado |

**Distribución típica:**

- Positive cases: 20-30%
- Negative cases: 40-50%
- Boundary cases: 15-20%
- API/Security: 10-20%

📖 **Ver flujo completo detallado**: [WORKFLOW.md](./WORKFLOW.md)

## Flujo de trabajo

1. **Extracción**: Lee la historia de usuario de Jira (título, descripción, criterios de aceptación)
2. **Análisis**: Identifica el tipo de funcionalidad (API, UI, lógica de negocio)
3. **Aplicación de técnicas**:
   - Pairwise para múltiples variables
   - Boundary Value Analysis para rangos numéricos
   - Decision Tables para lógica condicional
   - API Testing heuristics para endpoints
4. **Generación**: Crea test cases optimizados (8-15 casos según complejidad)
5. **Creación**: Sube automáticamente a Qase con metadata completa
6. **Vinculación**: Añade comentario en Jira con links a los test cases

## Formato de salida

Los test cases generados incluyen:

- **ID secuencial** (TC-001, TC-002, ...)
- **Escenario**: Nombre conciso del caso
- **Descripción**: QUÉ se está probando (sin pasos de UI)
- **Resultado esperado**: Salida verificable (incluye códigos HTTP para APIs)
- **Prioridad sugerida**: high, medium, low (guardada en tags y custom fields, no asignada automáticamente)
- **Tipo**: positive, negative, boundary, api, security
- **Tags**: Clasificación automática + Jira issue key + prioridad sugerida
- **Análisis de técnicas**: Documentación de qué técnica formal se aplicó y por qué

**Nota sobre prioridad/severidad en Qase:**

- La herramienta crea todos los casos con prioridad=Medium y severidad=Normal por defecto
- La prioridad sugerida por la IA se guarda en tags y custom fields para referencia
- Debes ajustar manualmente prioridad/severidad en Qase según tu criterio y contexto del proyecto

### Ejemplo de test case generado (API)

**Scenario:** Successful user creation with valid data  
**Description:** Verify that POST /api/users endpoint creates a user when all required fields are sent with valid format  
**Expected Result:** Status code 201, response body contains generated user_id, confirmed email, creation timestamp  
**Priority (suggested):** High  
**Type:** positive (api)  
**Tags:** api, user-creation, authentication, high, PROJ-123

_In Qase, this case will be created with Priority=Medium and Severity=Normal. The "High" priority suggested by AI will be available in tags and custom fields for manual adjustment._

**Qase ID:** Assigned automatically by Qase (e.g., #1501)

### Ejemplo de análisis de técnicas

```
ANÁLISIS:
Se detectaron 4 variables en el formulario de registro (email, password, age, country).
Se aplicó PAIRWISE TESTING para optimizar las 16 combinaciones posibles a 6 casos representativos.

Se identificó rango numérico en el campo "age" (18-100).
Se aplicó BOUNDARY VALUE ANALYSIS: 17, 18, 50, 100, 101.

Combinaciones elegidas (Pairwise):
1. email válido + password fuerte + age 18 + country US
2. email válido + password débil + age 100 + country MX
3. email inválido + password fuerte + age 50 + country CA
... (documentación completa de la estrategia)
```

## Estructura del proyecto

```
src/
├── config/
│   └── environment.ts           # Configuración y variables de entorno
├── models/
│   ├── jira.types.ts           # Tipos TypeScript para Jira
│   ├── openai.types.ts         # Tipos para respuestas de ChatGPT
│   └── qase.types.ts           # Tipos para API de Qase
├── services/
│   ├── jira.service.ts         # Comunicación con Jira API
│   ├── openai.service.ts       # Generación con ChatGPT
│   └── qase.service.ts         # Integración con Qase
├── utils/
│   └── logger.ts               # Sistema de logging
├── prompts/
│   └── test-case-generator.prompt.ts  # Prompts para ChatGPT
└── index.ts                    # Punto de entrada principal
```

## Personalización

### Ajustar el prompt de generación

Edita `src/prompts/test-case-generator.prompt.ts` para cambiar:

- Cantidad de casos a generar (actualmente 8-15)
- Técnicas formales a priorizar
- Heurísticas adicionales para API testing
- Nivel de detalle en el análisis

### Cambiar modelo de OpenAI

En `.env`:

```bash
OPENAI_MODEL=gpt-4              # Más preciso, más caro
OPENAI_MODEL=gpt-4-turbo        # Balance velocidad/calidad
OPENAI_MODEL=gpt-3.5-turbo      # Más rápido, más económico
```

### Configurar logging

```bash
LOG_LEVEL=debug   # Máximo detalle
LOG_LEVEL=info    # Normal (recomendado)
LOG_LEVEL=warn    # Solo warnings y errores
LOG_LEVEL=error   # Solo errores
```

Logs disponibles en:

- `logs/combined.log` - Todos los eventos
- `logs/error.log` - Solo errores

## Consideraciones

### 💰 Costos de OpenAI

**Precios aproximados (Enero 2025):**

| Modelo        | Costo por US | Sprint (10 US) | Proyecto (100 US) |
| ------------- | ------------ | -------------- | ----------------- |
| gpt-4-turbo   | ~$0.08       | ~$0.80         | ~$7.85            |
| gpt-4         | ~$0.24       | ~$2.40         | ~$23.70           |
| gpt-3.5-turbo | ~$0.004      | ~$0.04         | ~$0.40            |

**Recomendado:** gpt-4-turbo (mejor relación calidad/precio)

**Controles de costo implementados:**

- ✅ Límite de batch size (configurable en MAX_BATCH_SIZE)
- ✅ Estimación de costo antes de procesar batch
- ✅ Alertas al 80% del threshold configurado
- ✅ Tracking en tiempo real de tokens y costos
- ✅ Límite máximo de tokens por request

**Configurar límites de seguridad:**

```bash
# En .env
MAX_BATCH_SIZE=50                    # Máximo 50 US por batch
COST_WARNING_THRESHOLD=10.00         # Alerta si supera $10
ENABLE_COST_CONFIRMATION=true        # Pedir confirmación
```

**Límite en OpenAI Dashboard:**

1. Ve a https://platform.openai.com/settings/organization/billing/limits
2. Configura "Monthly budget: $20" (ejemplo)
3. Activa email notifications

📖 **Ver análisis completo de costos**: [COST_ANALYSIS.md](./COST_ANALYSIS.md)

### Rate limits

- Qase: 600 requests/hora en plan Team
- OpenAI: Según tu tier (https://platform.openai.com/account/rate-limits)

### Mejores prácticas

- **API endpoints**: Escribe criterios de aceptación claros con ejemplos de request/response
- **Rangos numéricos**: Especifica límites exactos en la historia (ej: "edad entre 18-100")
- **Lógica condicional**: Documenta todas las condiciones y sus combinaciones
- **Variables múltiples**: Identifica claramente todos los campos y sus validaciones
- Revisar los test cases generados, especialmente el análisis de técnicas aplicadas
- Ajustar el prompt si el tipo de proyecto requiere heurísticas específicas

## Troubleshooting

### Error: "No se pudo obtener la issue de Jira"

- Verificar que el JIRA_API_TOKEN sea válido
- Confirmar que tienes permisos de lectura en el proyecto
- Validar que el issue key exista

### Error: "La respuesta de OpenAI no tiene el formato esperado"

- Revisar que OPENAI_API_KEY sea válida
- Verificar quota disponible en OpenAI
- Intentar con un modelo diferente

### Error: "Fallo al crear test case en Qase"

- Verificar QASE_API_TOKEN
- Confirmar que QASE_PROJECT_CODE corresponda a un proyecto existente
- Revisar que no se exceda el rate limit

## Licencia

MIT

## Stack técnico

- TypeScript 5.7
- Node.js 18+
- OpenAI API (GPT-4 Turbo)
- Jira REST API v3
- Qase API v1
- Winston (logging)
- Axios (HTTP client)
# Test-Case-Power-Studio
