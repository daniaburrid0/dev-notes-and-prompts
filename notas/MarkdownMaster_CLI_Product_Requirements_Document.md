# MarkdownMaster CLI - Documento de Requerimientos del Producto

## Descripción General

MarkdownMaster CLI es una herramienta de línea de comandos que utiliza la API de Claude 3.5 Haiku para transformar y mejorar texto en documentos Markdown bien estructurados. La aplicación delega el procesamiento principal a la API de Claude, actuando como una interfaz eficiente entre el usuario y las capacidades de procesamiento de texto de la IA.

### Librerías Principales

- **anthropic**: Integración con la API de Claude
- **typer**: Manejo de la interfaz de línea de comandos
- **rich**: Formateo y estilizado de la salida en terminal
- **uv**: Gestión de dependencias y entornos virtuales
- **ruff**: Linting y formateo de código Python

## Características Principales

### 1. Procesamiento de Texto

- Entrada desde archivos o terminal
- Conversión a Markdown bien formateado
- Limpieza y estructuración automática del texto

### 2. Opciones de Procesamiento

- `--clean`: Conversión básica a Markdown
- `--summary`: Generación de resumen
- `--expand`: Expansión de contenido

### 3. Traducción

- Traducción entre inglés y español (`--to-es`, `--to-en`)
- Preservación del formato Markdown durante la traducción

### 4. Salida

- Guardado a archivo o salida en terminal
- Sobreescritura opcional de archivos existentes

## Arquitectura Técnica

### Gestión de Dependencias y Entorno

1. **Uso de uv**
   - Gestión de entornos virtuales
   - Instalación y gestión de dependencias
   - Gestión de versiones de Python

```bash
# Crear y activar entorno virtual
uv venv
source .venv/bin/activate  # Unix/MacOS
.venv\Scripts\activate     # Windows

# Inicializar proyecto
uv init

# Agregar dependencias del proyecto
uv add anthropic typer rich

# Generar archivo de bloqueo
uv lock

# Sincronizar dependencias
uv sync
```

2. **Gestión de Herramientas**
   - Instalación de herramientas de desarrollo
   - Actualización automática de shell

```bash
# Instalar herramientas de desarrollo
uv tool install ruff black mypy

# Actualizar shell para incluir herramientas
uv tool update-shell
```

### Componentes Principales

1. **Interfaz CLI** (`cli.py`)

   - Procesamiento de argumentos
   - Manejo de entrada/salida
   - Gestión de errores

2. **Integración con Claude** (`claude.py`)

   - Comunicación con API de Claude
   - Gestión de prompts
   - Procesamiento de respuestas

3. **Utilidades** (`utils.py`)
   - Funciones auxiliares
   - Manejo de archivos
   - Formateo de salida

## Estructura del Proyecto

```
markdownmaster-cli/
├── markdownmaster/
│   ├── __init__.py
│   ├── cli.py
│   ├── claude.py
│   └── utils.py
├── tests/
│   ├── test_cli.py
│   ├── test_claude.py
│   └── test_utils.py
├── .gitignore
├── pyproject.toml
└── README.md
```

## Detalles de Implementación

### Uso de Comandos

```bash
# Conversión básica
markdownmaster convert input.txt

# Conversión con opciones
markdownmaster convert input.txt --clean
markdownmaster convert input.txt --summary
markdownmaster convert input.txt --expand

# Traducción
markdownmaster convert input.txt --to-es
markdownmaster convert input.txt --to-en

# Opciones de salida
markdownmaster convert input.txt -o output.md
```

### Integración con API de Claude

```python
import anthropic
from typing import Dict, Optional

# Inicializar el cliente de Anthropic
client = anthropic.Anthropic()

# Modelo por defecto
DEFAULT_MODEL = "claude-3-haiku-20240307"

async def procesar_con_claude(
    contenido: str,
    tarea: str,
    opciones: Optional[Dict] = None,
    max_tokens: int = 1000,
    temperatura: float = 0
) -> str:
    """
    Procesa el contenido usando la API de Claude

    Args:
        contenido: Texto a procesar
        tarea: Tipo de tarea (convertir, traducir, etc.)
        opciones: Opciones adicionales de procesamiento
        max_tokens: Número máximo de tokens en la respuesta
        temperatura: Nivel de creatividad en las respuestas (0-1)

    Returns:
        str: Texto procesado
    """
    # Cargar el prompt específico según la tarea
    prompts = {
        "CONVERT": "<convert_prompt>...",
        "SUMMARY": "<summary_prompt>...",
        "EXPAND": "<expand_prompt>...",
        "TRANSLATE": "<translate_prompt>..."
    }

    base_prompt = prompts.get(tarea, "")
    if not base_prompt:
        raise ValueError(f"Tarea no válida: {tarea}")

    # Construir el prompt final
    prompt = base_prompt.replace("{{CONTENT}}", contenido)
    if opciones:
        for key, value in opciones.items():
            prompt = prompt.replace(f"{{{{{key}}}}}", str(value))

    try:
        # Configurar la llamada a la API
        response = client.messages.create(
            model=DEFAULT_MODEL,
            max_tokens=max_tokens,
            temperature=temperatura,
            system="Eres un asistente especializado en procesamiento y formateo de texto a Markdown.",
            messages=[
                {"role": "user", "content": prompt}
            ]
        )

        # Extraer y validar la respuesta
        if not response.content:
            raise ValueError("No se recibió respuesta del modelo")

        return response.content[0].text

    except anthropic.APIError as e:
        raise Exception(f"Error en la API de Claude: {str(e)}")
    except Exception as e:
        raise Exception(f"Error procesando el contenido: {str(e)}")

# Funciones específicas para cada tarea
async def convertir_a_markdown(contenido: str) -> str:
    return await procesar_con_claude(contenido, "CONVERT")

async def generar_resumen(contenido: str) -> str:
    return await procesar_con_claude(contenido, "SUMMARY")

async def expandir_contenido(contenido: str) -> str:
    return await procesar_con_claude(contenido, "EXPAND", temperatura=0.7)

async def traducir_texto(contenido: str, idioma_destino: str) -> str:
    opciones = {"TARGET_LANGUAGE": idioma_destino}
    return await procesar_con_claude(contenido, "TRANSLATE", opciones)
```

### Prompts por Tarea

#### Convert Prompt (`--clean`)

Purpose: Transform raw text into well-formatted Markdown while preserving content structure and enhancing readability.

````prompt
You are tasked with transforming raw text into well-formatted Markdown while preserving the content structure and enhancing readability. Here is the raw text you will be working with:

<raw_text>
{{RAW_TEXT}}
</raw_text>

Follow these guidelines to complete the task:

1. Preserve the overall structure of the content:
   - Maintain the hierarchy of headings
   - Keep paragraphs intact
   - Retain the order of list items

2. Enhance readability by:
   - Adding appropriate line breaks
   - Using consistent indentation for nested elements
   - Separating sections with blank lines

3. Apply the following Markdown formatting elements where appropriate:
   - Use # for main headings, ## for subheadings, and ### for lower-level headings
   - Format lists using - or * for unordered lists, and 1. 2. 3. for ordered lists
   - Emphasize important words or phrases with *italics* or **bold**
   - Create links using [link text](URL) syntax
   - Use `backticks` for inline code or technical terms
   - Format block quotes by prepending > to each line

4. Handle special cases:
   - For code blocks, use triple backticks (```) before and after the code, and specify the language if known
   - If tables are present, format them using | for columns and - for the header row separator

5. Ensure consistent spacing and formatting throughout the document

After applying these transformations, provide the final output in Markdown format. Begin your response with <markdown> and end it with </markdown>. Do not include any explanations or comments outside of these tags.
````

#### Summary Prompt (`--summary`)

Purpose: Generate a concise summary of the input text while maintaining key points and converting to Markdown format.

```prompt
You will be given a text to summarize and convert to Markdown format. Your task is to create a concise summary that maintains the key points of the original text while presenting it in a clear, structured Markdown format.

Here is the text to summarize:

<text>
{{TEXT}}
</text>

Follow these guidelines to create your summary:

1. Carefully read and analyze the input text.
2. Identify the main ideas, key points, and essential information.
3. Create a concise summary that captures the essence of the original text.
4. Ensure that your summary is significantly shorter than the original text while retaining the most important information.
5. Use your own words to rephrase the content, avoiding direct quotes unless absolutely necessary.
6. Organize the information in a logical and coherent manner.

When converting your summary to Markdown format, follow these rules:

1. Use appropriate Markdown syntax for headings, subheadings, lists, and emphasis.
2. Use level 1 heading (#) for the main title of the summary.
3. Use level 2 headings (##) for major sections or topics.
4. Use level 3 headings (###) for subsections if needed.
5. Use bullet points (-) or numbered lists (1., 2., 3.) for listing items or steps.
6. Use bold (**text**) or italic (*text*) formatting to emphasize important points or terms.
7. If applicable, use blockquotes (>) for any direct quotes you decide to include.

Present your final summary in Markdown format within <summary> tags. Ensure that your summary is well-structured, easy to read, and effectively captures the main points of the original text.

<summary>
[Your Markdown-formatted summary goes here]
</summary>
```

#### Expand Prompt (`--expand`)

Purpose: Enhance and elaborate the input text with additional details and examples while ensuring proper Markdown formatting.

````prompt
You will be given a piece of text to enhance and elaborate. Your task is to expand on the original content by adding more details, examples, and explanations while maintaining the overall message and tone. Additionally, you should ensure that the output is properly formatted using Markdown.

Here is the original text:

<input_text>
{{INPUT_TEXT}}
</input_text>

Follow these steps to enhance and elaborate the text:

1. Read and understand the main points of the original text.
2. For each main point or concept, add additional supporting details or explanations.
3. Include relevant examples to illustrate key ideas.
4. Expand on any technical terms or concepts that may benefit from further clarification.
5. Add transitions between paragraphs and sections to improve flow.
6. Ensure that the enhanced text is at least 50% longer than the original, but no more than three times the original length.

When formatting the text, follow these Markdown guidelines:

1. Use appropriate headings (##, ###, ####) to structure the content hierarchically.
2. Utilize **bold** and *italic* text for emphasis where appropriate.
3. Create bulleted or numbered lists for presenting multiple items or steps.
4. Use blockquotes (>) for important quotes or highlighted information.
5. Include code blocks (```) for any technical or code-related content.
6. Add horizontal rules (---) to separate major sections if necessary.

Write your enhanced and elaborated version of the text within <enhanced_text> tags. Ensure that the output is well-structured, informative, and properly formatted using Markdown.
````

#### Translation Prompts (`--to-es`, `--to-en`)

Purpose: Translate content between English and Spanish while preserving Markdown formatting and maintaining the original text's tone and structure.

````prompt
You are tasked with translating content between English and Spanish while preserving Markdown formatting and maintaining the original text's tone and structure. You will be given the following inputs:

<source_text>
{{SOURCE_TEXT}}
</source_text>

<target_language>
{{TARGET_LANGUAGE}}
</target_language>

Follow these instructions carefully:

1. Preserve Markdown formatting:
   - Maintain all Markdown syntax elements (e.g., #, ##, *, **, _, __, `, ```).
   - Keep links intact, translating only the link text if applicable.
   - Preserve any HTML tags present in the source text.

2. Maintain tone and structure:
   - Keep the same paragraph structure as the original text.
   - Preserve the original text's tone (formal, informal, technical, etc.).
   - Maintain any lists, bullet points, or numbered items in the same format.

3. Handle language-specific elements:
   - For idioms or expressions, try to find an equivalent in the target language that conveys the same meaning.
   - If there's no direct equivalent, provide a translation that captures the essence of the original phrase.
   - For cultural references, provide a brief explanation in parentheses if necessary.

4. Translation process:
   - First, analyze the source text to understand its content, tone, and structure.
   - Then, translate the content into the target language, ensuring accuracy and fluency.
   - Finally, review your translation to ensure it maintains the original meaning and style.

5. Output format:
   - Provide your translation within <translation> tags.
   - If you need to explain any translation choices or provide notes, use <translator_note> tags after the translation.

Here are two examples to illustrate the expected input and output:

Example 1:
<source_text>
# Welcome to My Blog

This is a **bold** statement about _markdown_ formatting.

1. First item
2. Second item

Check out [this link](https://www.example.com) for more information.
</source_text>

<target_language>Spanish</target_language>

<translation>
# Bienvenido a Mi Blog

Esta es una afirmación **audaz** sobre el formato _markdown_.

1. Primer elemento
2. Segundo elemento

Consulta [este enlace](https://www.example.com) para obtener más información.
</translation>

Example 2:
<source_text>
## How to Make a PB&J Sandwich

1. Take two slices of bread
2. Spread peanut butter on one slice
3. Spread jelly on the other slice
4. Put the slices together
5. Enjoy your sandwich!
</source_text>

<target_language>Spanish</target_language>

<translation>
## Cómo Hacer un Sándwich de Mantequilla de Cacahuete y Mermelada

1. Toma dos rebanadas de pan
2. Unta mantequilla de cacahuete en una rebanada
3. Unta mermelada en la otra rebanada
4. Junta las rebanadas
5. ¡Disfruta tu sándwich!
</translation>

<translator_note>
"PB&J" se ha traducido como "mantequilla de cacahuete y mermelada" para mayor claridad, ya que el acrónimo no es común en español.
</translator_note>

Now, proceed with the translation task using the provided source text and target language.
````

## Guías de Desarrollo

### Estilo de Código

- Usar type hints
- Seguir PEP 8
- Documentar funciones principales

### Pruebas

- Pruebas unitarias para componentes principales
- Pruebas de integración para flujos completos
- Simulación de llamadas a API

### Manejo de Errores

- Manejo claro de errores de API
- Mensajes de error descriptivos
- Recuperación apropiada de fallos

## Consideraciones Futuras

- Soporte para más idiomas
- Procesamiento por lotes
- Modo interactivo
- Integración con editores de texto
