# PML — PEPPO Markup Language

## Especificación completa v1.1

**PEPPO** — Platform for Engaging People and Processes via Omni-agents

Abril 2026

---

## Índice

```
 1. Visión general
 2. Reglas de sintaxis
 3. Atributos globales
 4. Componentes standalone (en chat)
    4.1  Text
    4.2  Markdown
    4.3  Facts
    4.4  DTable
    4.5  Image
    4.6  Media
    4.7  Divider
    4.8  Input
    4.9  Toggle
    4.10 Choice
    4.11 Confirm


 5. Modelo de interacción
 6. Reglas de composición
 7. Guías para LLMs / Agentes
 8. Guías de UX para móviles
 9. UX de componentes standalone (en chat)
   9.1 Text
   9.2 Markdown
   9.3 Facts
   9.4 DTable
   9.5 Image
   9.6 Media
   9.7 Divider
   9.8 Input
   9.9 Toggle
   9.10 Choice
   9.11 Confirm
 10. Manejo de errores
 11. Versionado y extensibilidad
```

---

## 1. Visión general

PML es un lenguaje **ligero, legible para LLMs y ultra-eficiente en tokens** diseñado para que los agentes conversacionales rendericen componentes UI interactivos dentro de interfaces de chat.

PML asume que todo chat es **multiusuario**. Una conversación 1:1 también es multiusuario: es un grupo mínimo de dos participantes. Una conversación grupal extiende el mismo modelo a más participantes. Por eso, las interacciones PML no son eventos privados del cliente local; afectan el historial compartido de la conversación.

**Filosofía:**

```
El agente DECLARA en PML.
La Chat App RENDERIZA y EJECUTA la lógica.
El usuario INTERACTÚA.
La Chat App CAPTURA la acción y la envía al agente.
El agente PROCESA y genera nuevo PML.
```

**Principios de diseño:**

```
Token-efficient    →  Más compacto que JSON, XML o cualquier alternativa
LLM-friendly       →  Sintaxis regular y predecible, fácil de generar sin errores
Chat-native        →  Diseñado para flujos conversacionales en móvil
Multi-user first   →  Toda interacción afecta un historial compartido
Renderer-powered   →  La lógica de negocio vive en el renderer, no en el agente
Extensible         →  Nuevos componentes sin romper compatibilidad
```

---

## 2. Reglas de sintaxis

### 2.1 Formato general

- PML se escribe como **texto plano + Markdown** dentro de mensajes de agente.
- Los componentes PML van entre corchetes.
- Se pueden mezclar libremente texto, Markdown y componentes PML en un mismo mensaje.

### 2.2 Componente autofinalizado

```
[PML:tipo atributo1="valor1" atributo2="valor2" /]
```

### 2.3 Componente contenedor

```
[PML:tipo atributo="valor"]
  contenido hijo
[/PML:tipo]
```

### 2.4 Reglas estrictas

| Regla           | Detalle                                                                                                                                             |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Comillas        | Los valores de atributos se delimitan con `" "` (preferido) o `' '` (cuando el valor contiene comillas dobles). Nunca mezclar en el mismo atributo. |
| Prohibido       | Nunca usar JSON, `< >`, ni caracteres especiales fuera de los atributos                                                                             |
| IDs             | Todo componente interactivo **debe** tener `id="identificador-unico"`                                                                               |
| Unicidad        | Los `id` deben ser únicos dentro de un mensaje                                                                                                      |
| Escapado        | No existen secuencias de escape. El LLM elige la comilla que no conflicte con el contenido.                                                         |
| Autofinalizados | **Siempre** terminan con ` /]` — el espacio antes de `/]` es obligatorio                                                                            |
| Cierre          | Los contenedores **siempre** se cierran con `[/PML:tipo]`                                                                                           |
| Case-sensitive  | Los nombres de componentes y atributos son en minúsculas                                                                                            |
| Encoding        | Todo el contenido es UTF-8                                                                                                                          |

### 2.5 Encoding de interacciones

Cuando la Chat App envía un mensaje de interacción al agente, **siempre** aplica URL encoding a los valores que provienen de texto libre del usuario. El agente **siempre** espera recibir valores codificados.

| Carácter | Codificación |
| -------- | ------------ |
| espacio  | `+`          |
| `&`      | `%26`        |
| `=`      | `%3D`        |
| `"`      | `%22`        |
| `'`      | `%27`        |
| `+`      | `%2B`        |
| `%`      | `%25`        |
| `\n`     | `%0A`        |

**Se codifica:** texto libre del usuario (input, textarea, etc.)

**No se codifica:** `id`, `value` de botones/choice/toggle, referencias de file, coordenadas de location — todos definidos por el agente o por la Chat App, sin caracteres especiales.

**Ejemplo:**

```
Usuario escribe:     Pedro & María
Chat App envía:      [[INTERACTION]] id=nombre value=Pedro+%26+María
```

### 2.6 Ejemplo de mensaje válido

```
Hola Pedro, ¿qué deseas hacer hoy?

[PML:choice id="acciones"]
  [PML:opt value="book" label="Reservar cita" /]
  [PML:opt value="prices" label="Ver precios" /]
[/PML:choice]
```

**Ejemplo con comillas simples (valor contiene comillas dobles):**

```
[[INTERACTION id=book value="book"]]
```

### 2.7 Tipos de componentes PML

```
tipo            = "text" | "markdown" | "image" | "media" | "input" | "toggle" | "choice" | "opt"
                | "facts" | "divider" | "dtable" | "confirm" ;
```

---

## 3. Atributos globales

Aplican a **todos** los componentes PML:

| Atributo   | Requerido            | Tipo             | Descripción                                     |
| ---------- | -------------------- | ---------------- | ----------------------------------------------- |
| `id`       | Sí (en interactivos) | string           | Identificador único. Formato: `kebab-case`.     |
| `disabled` | No                   | `true` / `false` | Deshabilita interacción. Default: `false`.      |
| `tooltip`  | No                   | string           | Texto de ayuda al mantener presionado (mobile). |
| `visible`  | No                   | `true` / `false` | Controla visibilidad. Default: `true`.          |
| `error`    | No                   | string           | Mensaje de error asociado. Se renderiza debajo. |

---

## 4. Componentes standalone (en chat)

Estos componentes se usan directamente en el flujo del chat.

---

### 4.1 Text

Bloque de texto estático con estilo.

| Atributo | Requerido | Tipo                                                 | Descripción                           |
| -------- | --------- | ---------------------------------------------------- | ------------------------------------- |
| `type`   | No        | `header` / `title` / `paragraph` / `muted` / `label` | Tipo semántico. Default: `paragraph`. |
| `style`  | No        | `bold` / `italic` / `muted` / `normal`               | Estilo visual. Default: `normal`.     |
| `size`   | No        | `small` / `medium` / `large`                         | Tamaño. Default: `medium`.            |
| `align`  | No        | `left` / `center` / `right`                          | Alineación. Default: `left`.          |

**Ejemplo:**

```
[PML:text type="header" size="large" style="bold"]
Tu pedido está en camino
[/PML:text]

[PML:text type="muted" size="small"]
Estimamos entrega en 3-5 días hábiles
[/PML:text]
```

---

### 4.2 Markdown

Bloque de texto con formato Markdown completo.

| Atributo | Requerido | Tipo   | Descripción   |
| -------- | --------- | ------ | ------------- |
| `id`     | No        | string | Identificador |

**Ejemplo:**

```
[PML:markdown]
### Instrucciones de envío

1. **Empaca** el producto en su caja original
2. **Pega** la etiqueta de envío en la parte exterior
3. **Lleva** el paquete a cualquier centro de envío

> El reembolso se procesa en 5-7 días hábiles.

Para más info visita nuestra [guía de envíos](https://peppo.com/envios).
[/PML:markdown]
```

**Soporta:** Negritas, cursivas, listas, enlaces, código inline, blockquotes, encabezados.

---

### 4.3 Facts

Tabla de pares clave-valor.

| Atributo  | Requerido | Tipo      | Descripción                       |
| --------- | --------- | --------- | --------------------------------- |
| `id`      | No        | string    | Identificador                     |
| `columns` | No        | `1` / `2` | Columnas de layout. Default: `1`. |

**Formato del contenido:** Cada entrada va envuelta en `[PML:fact]Clave: valor[/PML:fact]`. Este formato es obligatorio y funciona tanto en una sola línea como en múltiples líneas.

**Ejemplo:**

```
[PML:facts columns="2"]
[PML:fact]Producto: Nike Air Max 90[/PML:fact]
[PML:fact]Talla: 42[/PML:fact]
[PML:fact]Precio: $2,499 MXN[/PML:fact]
[PML:fact]Envío: Gratis[/PML:fact]
[/PML:facts]
```

---

### 4.4 DTable

Tabla de datos

| Atributo  | Requerido | Tipo   | Descripción                            |
| --------- | --------- | ------ | -------------------------------------- |
| `id`      | Sí        | string | Identificador único                    |
| `columns` | Sí        | string | Nombres de columnas separados por "\|" |

**Formato del contenido:** Cada fila va envuelta en `[PML:dtrow]cel1|cel2|cel3[/PML:dtrow]`. Este formato es obligatorio y funciona tanto en una sola línea como en múltiples líneas.

**Ejemplo:**

```
[PML:dtable id="comparativa" columns="Plan|Precio|Agentes|Mensajes"]
[PML:dtrow]Básico|$9/mes|1|100/día[/PML:dtrow]
[PML:dtrow]Pro|$29/mes|10|Ilimitado[/PML:dtrow]
[PML:dtrow]Enterprise|Custom|Ilimitado|Ilimitado[/PML:dtrow]
[/PML:dtable]
```

---

### 4.5 Image

Imagen estática.

| Atributo  | Requerido | Tipo                                  | Descripción                |
| --------- | --------- | ------------------------------------- | -------------------------- |
| `url`     | Sí        | url                                   | URL de la imagen           |
| `alt`     | No        | string                                | Texto alternativo          |
| `size`    | No        | `small` / `medium` / `large` / `full` | Tamaño. Default: `medium`. |
| `caption` | No        | string                                | Pie de imagen              |

**Ejemplo:**

```
[PML:image url="https://cdn.peppo.com/producto.jpg" alt="Nike Air Max 90"
  size="large" caption="Nuevo modelo 2026" /]
```

---

### 4.6 Media

Reproductor de video o audio.

| Atributo  | Requerido | Tipo              | Descripción                    |
| --------- | --------- | ----------------- | ------------------------------ |
| `url`     | Sí        | url               | URL del archivo                |
| `type`    | Sí        | `video` / `audio` | Tipo de medio                  |
| `poster`  | No        | url               | Imagen de portada (solo video) |
| `caption` | No        | string            | Pie de medio                   |

**Ejemplo:**

```
[PML:media url="https://cdn.peppo.com/demo.mp4" type="video"
  poster="https://cdn.peppo.com/demo-poster.jpg" /]
```

---

### 4.7 Divider

Línea separadora.

```
[PML:divider /]
```

Con texto:

```
[PML:divider label="O continúa con" /]
```

---

### 4.8 Input

Campo de entrada de datos.

**Tipos disponibles:**

`text` / `number` / `email` / `tel` / `password` / `date` / `time` / `location` / `file`

| Atributo       | Requerido | Tipo             | Descripción                     |
| -------------- | --------- | ---------------- | ------------------------------- |
| `id`           | Sí        | string           | Identificador único             |
| `type`         | Sí        | string           | Tipo de campo                   |
| `label`        | No        | string           | Etiqueta visible                |
| `placeholder`  | No        | string           | Texto de ayuda                  |
| `value`        | No        | string           | Valor inicial                   |
| `required`     | No        | `true` / `false` | Obligatorio. Default: `false`.  |
| `validate`     | No        | string           | Reglas declarativas (ver 4.8.2) |
| `errorMessage` | No        | string           | Mensaje si validación falla     |
| `maxLength`    | No        | number           | Máximo de caracteres            |
| `minLength`    | No        | number           | Mínimo de caracteres            |
| `prefix`       | No        | string           | Texto fijo antes del valor      |
| `suffix`       | No        | string           | Texto fijo después del valor    |
| `readonly`     | No        | `true` / `false` | Solo lectura. Default: `false`. |

#### 4.8.1 Subtipos especiales

**location:**

```
[PML:input id="ubicacion" type="location" label="Tu ubicación"
  placeholder="Toca para compartir ubicación" /]
```

Renderiza un selector de ubicación que puede:

- Obtener coordenadas GPS del dispositivo
- Permitir buscar una dirección
- Mostrar un mini mapa con pin

**Interacción generada:**

```
[[INTERACTION]] id=ubicacion value=19.4326,-99.1332
```

**file:**

```
[PML:input id="foto" type="file" label="Sube tu identificación"
  accept="image/*" maxSize="5MB" /]
```

| Atributo adicional | Tipo             | Descripción                                              |
| ------------------ | ---------------- | -------------------------------------------------------- |
| `accept`           | string           | Tipos MIME permitidos (ej: `image/*`, `application/pdf`) |
| `maxSize`          | string           | Tamaño máximo (ej: `5MB`, `10MB`)                        |
| `multiple`         | `true` / `false` | Permitir múltiples archivos. Default: `false`.           |

**Interacción generada:**

```
[[INTERACTION]] id=foto value=file_ref_abc123
```

#### 4.8.2 Validación declarativa

El atributo `validate` acepta reglas separadas por coma:

| Regla          | Formato             | Ejemplo                    |
| -------------- | ------------------- | -------------------------- |
| `minLength`    | `minLength:N`       | `minLength:8`              |
| `maxLength`    | `maxLength:N`       | `maxLength:100`            |
| `exactLength`  | `exactLength:N`     | `exactLength:10`           |
| `min`          | `min:N`             | `min:18`                   |
| `max`          | `max:N`             | `max:120`                  |
| `pattern`      | `pattern:regex`     | `pattern:^[A-Z]{2}\\d{4}$` |
| `hasUppercase` | `hasUppercase:true` |                            |
| `hasNumber`    | `hasNumber:true`    |                            |
| `hasSpecial`   | `hasSpecial:true`   |                            |
| `matches`      | `matches:campoId`   | `matches:password`         |

**Ejemplo:**

```
[PML:input id="password" type="password" label="Contraseña"
  required="true" validate="minLength:8,hasUppercase:true,hasNumber:true"
  errorMessage="Mínimo 8 caracteres, una mayúscula y un número" /]
```

**Interacción generada:**

```
[[INTERACTION]] id=email value=pedro@email.com
```

---

### 4.9 Toggle

Interruptor sí/no.

| Atributo   | Requerido | Tipo             | Descripción                                |
| ---------- | --------- | ---------------- | ------------------------------------------ |
| `id`       | Sí        | string           | Identificador único                        |
| `label`    | Sí        | string           | Texto descriptivo                          |
| `checked`  | No        | `true` / `false` | Estado inicial. Default: `false`.          |
| `valueOn`  | No        | string           | Valor cuando activo. Default: `"true"`.    |
| `valueOff` | No        | string           | Valor cuando inactivo. Default: `"false"`. |
| `required` | No        | `true` / `false` | Obligatorio. Default: `false`.             |

**Ejemplo:**

```
[PML:toggle id="terminos" label="Acepto los términos y condiciones"
  checked="false" required="true" /]
```

---

### 4.10 Choice

Selección entre opciones.

| Atributo      | Requerido | Tipo                             | Descripción                           |
| ------------- | --------- | -------------------------------- | ------------------------------------- |
| `id`          | Sí        | string                           | Identificador único                   |
| `style`       | No        | `buttons` / `dropdown` / `radio` | Renderizado. Default: `buttons`.      |
| `multi`       | No        | `true` / `false`                 | Selección múltiple. Default: `false`. |
| `placeholder` | No        | string                           | Texto por defecto (solo dropdown)     |
| `required`    | No        | `true` / `false`                 | Obligatorio. Default: `false`.        |

### 4.10.1 Opciones

Las opciones tienen el formato **[PML:opt value="valor" /]** y son hijos directos de la etiqueta `choice`. El `value` es a la vez el texto visible para el usuario **y** el valor enviado al agente — no hay `label` separado.

**Atributos de `opt`:**

| Atributo | Requerido | Tipo   | Descripción                                                                          |
| -------- | --------- | ------ | ------------------------------------------------------------------------------------ |
| `value`  | Sí        | string | Texto visible y valor enviado al agente como `[[INTERACTION]]`.                      |
| `id`     | No        | string | Identificador único de la opción. Si falta, el renderer puede generar uno.           |
| `label`  | No        | string | Texto visible alternativo. Si se omite, se usa `value`. Reservado para uso avanzado. |

**Ejemplo:**

```
[PML:choice id="talla" style="buttons"]
  [PML:opt value="Pequeña" /]
  [PML:opt value="Mediana" /]
  [PML:opt value="Grande" /]
  [PML:opt value="Extra Grande" /]
[/PML:choice]
```

---

### 4.14 Confirm

Diálogo de confirmación.

| Atributo       | Requerido | Tipo                             | Descripción                                    |
| -------------- | --------- | -------------------------------- | ---------------------------------------------- |
| `id`           | Sí        | string                           | Identificador único                            |
| `title`        | Sí        | string                           | Título del diálogo                             |
| `message`      | No        | string                           | Mensaje descriptivo                            |
| `confirmLabel` | No        | string                           | Texto de confirmación. Default: `"Confirmar"`. |
| `cancelLabel`  | No        | string                           | Texto de cancelación. Default: `"Cancelar"`.   |
| `style`        | No        | `default` / `danger` / `warning` | Estilo. Default: `default`.                    |

**Ejemplo:**

```
[PML:confirm id="eliminar-cuenta" title="¿Eliminar tu cuenta?"
  message="Se borrarán todos tus datos."
  confirmLabel="Sí, eliminar" cancelLabel="No, conservar" style="danger" /]
```

**Interacciones generadas:**

Si el usuario confirma:

```
[[INTERACTION]] id=eliminar-cuenta value=confirmed
```

Si el usuario cancela:

```
[[INTERACTION]] id=eliminar-cuenta value=cancelled
```

---

## 5. Modelo de interacción

### 5.1 Formatos de interacción

| Formato                        | Cuándo se usa                                  |
| ------------------------------ | ---------------------------------------------- |
| `[[INTERACTION]] id=X value=Y` | bo, choice, toggle, confirmación, tab, section |

### 5.2 Flujo completo

```
┌──────────┐        PML         ┌──────────┐
│          │ ─────────────────→  │          │
│  AGENTE  │                     │ CHAT APP │
│          │ ←─────────────────  │(renderer)│
└──────────┘   [[INTERACTION]]  └──────────┘
```

```
1. Agente genera PML con componentes
2. Chat App renderiza componentes
3. Chat App gestiona lógica (visibilidad, aceptación, cálculos)
4. Usuario interactúa
5. Chat App captura acción → envía interacción como texto
6. Agente recibe, procesa, genera nuevo PML
7. Chat App renderiza
8. Repetir
```

### 5.3 Ejemplo de conversación completa

```
USUARIO: Quiero comprar unos zapatos

AGENTE: ¡Perfecto! Tenemos varios modelos disponibles.

        [PML:image url="https://cdn.peppo.com/catalogo.jpg"
          alt="Catálogo de zapatos" size="full" /]

        ¿Cuál te interesa?

        [PML:choice id="catalogo"]
          [PML:opt value="nike" label="Nike Air Max — $2,499" /]
          [PML:opt value="adidas" label="Adidas Ultraboost — $3,199" /]
        [/PML:choice]

--- El usuario toca "Nike Air Max" ---

USUARIO: [[INTERACTION]] id=nike value=nike

AGENTE: Excelente elección. ¿Qué talla necesitas?

        [PML:choice id="talla" style="buttons"]
          [PML:opt value="38" /]
          [PML:opt value="39" /]
          [PML:opt value="40" /]
          [PML:opt value="41" /]
          [PML:opt value="42" /]
          [PML:opt value="43" /]
        [/PML:choice]

--- El usuario selecciona "42" ---

USUARIO: [[INTERACTION]] id=talla value=42

AGENTE: Nike Air Max, talla 42. Para completar tu compra
        necesito tus datos de envío. Toca el botón cuando estés listo.

        [PML:choice id="start-checkout"]
          [PML:opt value="start" label="Completar datos" /]
        [/PML:choice]

--- El usuario toca el botón ---

USUARIO: [[INTERACTION]] id=start-checkout value=start

AGENTE: ¡Listo, Pedro! Aquí está tu resumen:

        [PML:facts columns="2"]
          Producto: Nike Air Max 90
          Talla: 42
          Precio: $2,499 MXN
          Envío: Gratis
          Nombre: Pedro García
          Dirección: Reforma 123, CDMX, 06600
          Teléfono: +52 555 123 4567
          Entrega estimada: 3-5 días hábiles
        [/PML:facts]

        [PML:choice id="post-compra"]
          [PML:opt value="track" label="Rastrear pedido" /]
          [PML:opt value="continue" label="Seguir comprando" /]
        [/PML:choice]
```

---

## 6. Reglas de composición

### 6.1 Tabla de nesting permitido

```
┌──────────────────┬─────────────────────────────────────────────────┐
│ Contenedor        │ Puede contener                                   │
├──────────────────┼─────────────────────────────────────────────────┤
│ (raíz del mensaje)│ Todos los componentes standalone          │
│ choice            │ opt (uno o más)                                  │
└──────────────────┴─────────────────────────────────────────────────┘
```

### 6.2 Reglas de anidación

1. **Profundidad máxima** 2 niveles (choice > opt)

### 6.3 Reglas de IDs

- Únicos dentro de un mensaje
- Formato: `kebab-case`
- No espacios, no caracteres especiales, no empezar con números
- IDs duplicados: renderizar solo el primero, loggear warning

---

## 7. Guías para LLMs / Agentes

### 7.1 Reglas obligatorias

1. **Siempre** incluir `id` en todo componente interactivo
2. **Nunca** usar JSON, `< >` ni HTML para componentes
3. Máximo **5 botones** por `[PML:buttons]`
4. Usar `style="primary"` solo para la acción principal

### 7.2 Generación tolerante a errores

| Error del agente             | Comportamiento del renderer         |
| ---------------------------- | ----------------------------------- |
| Falta `id` en interactivo    | Generar `auto-{tipo}-{n}`           |
| Atributo desconocido         | Ignorar silenciosamente             |
| Tag sin cerrar               | Intentar autocerrar; si no, truncar |
| Más de 5 botones             | Renderizar 5, ignorar resto         |
| Componente hijo no permitido | Renderizar como texto plano         |
| ID duplicado                 | Renderizar el primero               |
| Profundidad > 2              | Aplanar al nivel 2                  |

---

## 8. Guías de UX para móviles

### 8.1 Principios

```
1. TODO CHAT ES MULTIUSUARIO
   Una conversación 1:1 es un grupo mínimo de dos participantes.
   Una conversación grupal usa el mismo modelo con más participantes.
   Toda interacción afecta el historial compartido.

2. EL CHAT INPUT ES SAGRADO
   Solo desaparece cuando no hay un input activo.
   Se restaura inmediatamente después del envío o cancelación.

3. NADA SE PIERDE
   Los datos enviados quedan como resumen inline en el historial.
```

### 8.1.1 Zonas y fases del renderer

Esta subsección define el **modelo canónico** que la ChatApp usa para renderizar un mensaje PML. Es la base de toda la sección 9: cualquier diagrama por componente (§9.5–§9.18) describe cómo ese componente se reparte entre las **zonas** y se mueve entre las **fases** definidas aquí.

#### 8.1.1.1 Modelo del mensaje

El mensaje PML se descompone en dos clases de contenido:

```
mensaje PML = narrativa + tarea

  narrativa  → text, markdown, image, media, facts, dividers, dtable
               (contenido pasivo; no requiere acción del usuario)

  tarea      → único componente interactivo del mensaje
               (input, choice, toggle, confirm)

tarea = task_narrative + task_controls
```

**Una tarea PML** es un componente Call To Action (CTA) que requiere una acción del usuario para completarse. No un componente de salida de información.

**Invariante:** un mensaje contiene **0 o 1 tarea**, y **N piezas de narrativa**. Esta es la traducción operativa de la regla "un único punto de captura por turno": el renderer nunca tiene que resolver conflictos entre dos componentes interactivos compitiendo por la atención del usuario.

#### 8.1.1.2 Las tres zonas

El renderer divide la superficie del chat en tres zonas con propiedades distintas:

| Zona             | Visibilidad             | Persistencia          | Interactiva | Contenido                                                         |
| ---------------- | ----------------------- | --------------------- | ----------- | ----------------------------------------------------------------- |
| `shared_history` | Todos los participantes | Permanente, inmutable | No          | Mensajes pasados, narrativa, snapshots de tareas resueltas        |
| `task_narrative` | Privada al disparador   | Efímera               | No          | Narrativa que enmarca la tarea en curso (texto, imagen, contexto) |
| `task_controls`  | Privada al disparador   | Efímera               | Sí          | El componente interactivo vivo de la tarea (único por turno)      |

```
┌──────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────────────────────────────┐    │
│  │   shared_history                                         │    │
│  │   (compartido · inmutable · scrollable)                  │    │
│  │                                                          │    │
│  │   Mensajes pasados, narrativa, tareas resueltas          │    │
│  │   como snapshots inmutables                              │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │   task_narrative                                         │    │
│  │   (privado · efímero)                                    │    │
│  │                                                          │    │
│  │   Pregunta, encabezado, contexto que enmarca la tarea    │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │   task_controls                                          │    │
│  │   (privado · efímero · interactivo · único por turno)    │    │
│  │                                                          │    │
│  │   El componente vivo: input, choice, confirm, etc.       │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

**Reglas de las zonas:**

- Solo `shared_history` es persistente y visible para todos los participantes. `task_narrative` y `task_controls` son siempre privados al usuario disparador y efímeros (existen mientras la tarea no ha resuelto).
- Solo puede haber **un `task_controls` activo** por conversación por usuario disparador en un momento dado. Esto se deriva de la invariante "0 o 1 tarea por mensaje".
- Las zonas `task_*` no se anidan, no se apilan, no permiten scroll independiente. Son un overlay efímero sobre `shared_history`.

#### 8.1.1.3 Las tres fases

Toda tarea PML atraviesa el mismo lifecycle de renderización. Los nombres son canónicos:

| Fase              | Quién la dispara             | Qué ocurre                                                                                                                                      |
| ----------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `first_rendering` | El agente (emite el mensaje) | El mensaje llega. Narrativa pinta en `task_narrative`. Controles pintan en `task_controls`. Privado al disparador.                              |
| `live_capture`    | El usuario disparador        | El usuario interactúa con los controles. Feedback visual en `task_controls`. Sigue privado.                                                     |
| `final_rendering` | La ChatApp                   | La tarea cierra. Snapshot inmutable de narrativa + tarea resuelta se promueve a `shared_history`. `task_narrative` y `task_controls` se vacían. |

```
   first_rendering        live_capture          final_rendering
   ──────────────         ─────────────         ────────────────
   El agente emite        El usuario actúa      El renderer congela
   narrativa + tarea      sobre los controles   la tarea, promueve
   a las zonas B y C      en zona C             snapshot a zona A
   (privado)              (privado)             (compartido)
```

#### 8.1.1.4 Matriz zona × fase

Resumen operativo de qué hay en cada zona en cada fase:

```
                  first_rendering         live_capture            final_rendering
                  ─────────────────       ──────────────────      ──────────────────────────
shared_history    (sin cambios)           (sin cambios)           snapshot inmutable de la
                                                                  narrativa + tarea resuelta

task_narrative    narrativa del mensaje   narrativa del mensaje   (vacío)
                  (texto, imagen, etc.)   (sin cambios)

task_controls     componente interactivo  componente + feedback   (vacío)
                  vivo                    de la interacción
```

#### 8.1.1.5 Casos borde

- **Mensaje sin tarea (solo narrativa).** No entra al lifecycle de tarea: la narrativa se renderiza directo en `shared_history` y es visible para todos los participantes desde el primer momento. `task_narrative` y `task_controls` no se usan. Esto refleja la convención universal de chat: un mensaje sin acción pendiente aparece inmediatamente en el historial común.
- **Mensaje sin narrativa (solo tarea).** `task_narrative` queda vacía durante `first_rendering` y `live_capture`. En `final_rendering`, el snapshot en `shared_history` contiene solo la tarea resuelta. La spec permite emitir `[PML:buttons]` sin texto encima.
- **Tarea cancelada o expirada.** Subcaso de `final_rendering`: las zonas `task_*` se vacían, pero el snapshot en `shared_history` registra el estado cancelado (no la tarea como si se hubiera resuelto). El renderer decide la apariencia visual del snapshot cancelado.

#### 8.1.1.6 Lo que NO está en PML

Las decisiones de **audiencia** (quién es el "disparador", quién ve las zonas privadas, quién recibe el snapshot final) **no se expresan en PML**. PML solo declara qué renderizar. La asignación de audiencia y visibilidad por participante la resuelve el runner del agente vía metadata del turno (envelope), antes de que el PML llegue al renderer.

Esto implica que un mismo bloque PML puede renderizarse simultáneamente como "tarea viva" para un usuario y como "placeholder" o nada para otros participantes, sin que el contenido PML cambie. El renderer lee el envelope del turno para saber qué pintar a quién.

> **Implementación de referencia:** ver `Peppo-runner.md` (sección de routing multiusuario) para el modelo de envelope y la política por tipo de conversación. PML asume que esa capa está resuelta cuando el mensaje llega al renderer.

### 8.2 Aplicación del modelo de zonas y fases a los diagramas

Los diagramas por componente (§8.5–§8.18) usan el modelo canónico definido en §8.1.1:

- **Componentes pasivos** (sin tarea: text, markdown, image, media, facts, divider, dtable) se renderizan **directo en `shared_history`**. Un único diagrama; no atraviesan `task_narrative` ni `task_controls`.
- **Componentes interactivos** (con tarea: input, toggle, choice, confirm) atraviesan el lifecycle completo: `first_rendering` → `live_capture` → `final_rendering`. Se diagraman tres bloques, uno por fase, mostrando las tres zonas separadas con líneas horizontales.

**Convención visual de las zonas en los diagramas:**

```
┌─────────────────────────┐
│ shared_history          │ ← compartido, inmutable
│   ...                   │
├─────────────────────────┤
│ task_narrative          │ ← privado al disparador, narrativa
│   ...                   │
├─────────────────────────┤
│ task_controls           │ ← privado al disparador, control vivo
│   ...                   │
└─────────────────────────┘
```

La línea horizontal `├──┤` separa las tres zonas. En `final_rendering`, las zonas `task_narrative` y `task_controls` aparecen vacías; el snapshot resuelto vive en `shared_history`.

**Caracterización de `live_capture` por componente:**

La fase `live_capture` se manifiesta visualmente distinto según el componente. Cuando un diagrama quiera enfatizar qué clase de actividad ocurre, puede etiquetarla así:

| Componente | Manifestación de `live_capture`                                    |
| ---------- | ------------------------------------------------------------------ |
| `input`    | Teclado/selector/GPS/archivo abierto, captura en curso             |
| `toggle`   | Fila táctil, cambio de estado binario                              |
| `choice`   | Selección de una o varias opciones                                 |
| `confirm`  | Diálogo modal explícito; el chat input queda pausado mientras dura |

**Lo que pierde el chat input durante `live_capture`:**

El chat input persistente (composer global de la ChatApp) sigue activo durante `live_capture` excepto cuando la tarea exige interacción exclusiva: `confirm` modal, `input type="file"` con selector abierto. En esos casos, el chat input se restaura tras `final_rendering`.

**Multiusuario:**

Las zonas `task_narrative` y `task_controls` que aparecen en los diagramas son **privadas al disparador**. Los demás participantes ven `shared_history` igual, pero las dos zonas inferiores pueden mostrar un placeholder (_"Pedro está respondiendo…"_) o quedar ocultas, según la política del runner. La asignación de quién es el disparador no se expresa en PML — viene del envelope del turno (ver §9.1.1.6).

### 8.3 Gestos soportados

| Gesto               | Acción                |
| ------------------- | --------------------- |
| Tap en botón/opción | Seleccionar / activar |
| Mantener presionado | Mostrar tooltip       |

## 9. UX de componentes standalone (en chat)

### 9.1 Text

Componente **pasivo** (narrativa). Render directo a `shared_history`.

- Usar `type="header"` o `type="title"` solo para iniciar bloques importantes; evitar varios títulos grandes consecutivos.
- Mantener párrafos cortos en móvil: idealmente 1-3 líneas visibles.
- Usar `muted` para contexto secundario, no para información crítica.
- Evitar centrar texto largo; `align="center"` funciona mejor en mensajes breves o estados vacíos.

```
render directo a shared_history
┌─────────────────────────┐
│ shared_history          │
│   turnos previos...     │
│   ┌───────────────────┐ │
│   │ Pedido en camino  │ │ ← header/title
│   │ Entrega estimada  │ │
│   │ 3-5 días hábiles  │ │ ← paragraph
│   │ Última act.       │ │ ← muted
│   └───────────────────┘ │
│   (visible para todos)  │
└─────────────────────────┘
```

### 9.2 Markdown

Componente **pasivo** (narrativa). Render directo a `shared_history`.

- Usar Markdown cuando el contenido necesite estructura real: pasos, listas, citas, enlaces o énfasis.
- Nunca usar tablas Markdown; preferir `facts` o `dtable` según el caso.
- Los encabezados dentro de Markdown no deben competir visualmente con el mensaje principal del agente.
- Mantener bloques de código y citas suficientemente cortos para no forzar scroll horizontal.

```
render directo a shared_history
┌─────────────────────────┐
│ shared_history          │
│   turnos previos...     │
│   ┌───────────────────┐ │
│   │ Instrucciones     │ │ ← encabezado
│   │ 1. Empaca         │ │
│   │ 2. Pega etiqueta  │ │ ← lista
│   │ > Reembolso...    │ │ ← cita breve
│   │ Ver guía          │ │ ← enlace
│   └───────────────────┘ │
│   (visible para todos)  │
└─────────────────────────┘
```

### 9.3 Facts

Componente **pasivo** (narrativa). Render directo a `shared_history`.

- Usar `facts` para resúmenes, confirmaciones y datos de solo lectura.
- En móvil, preferir `columns="1"` salvo que las claves y valores sean muy cortos.
- Mantener claves consistentes y breves.
- No usar `facts` para datos editables; usar `input` o `choice`.

```
render directo a shared_history
┌─────────────────────────┐
│ shared_history          │
│   turnos previos...     │
│   Resumen               │
│   ┌───────────────────┐ │
│   │ Producto  Nike    │ │
│   │ Talla     42      │ │ ← clave-valor
│   │ Precio    $2,499  │ │
│   └───────────────────┘ │
│   (visible para todos)  │
└─────────────────────────┘
```

### 9.4 DTable

Componente **pasivo** (narrativa). Render directo a `shared_history`. El scroll horizontal es interacción interna del componente; no produce `[[INTERACTION]]` ni cambia el estado de la tarea.

- Usar `dtable` solo cuando la comparación por filas y columnas sea esencial.
- En móvil, permitir scroll horizontal o transformar a filas tipo lista si la tabla es ancha.
- Mantener encabezados cortos y valores escaneables.
- Evitar celdas con párrafos largos; si el contenido crece, preferir `facts` o Markdown estructurado.

```
render directo a shared_history
┌─────────────────────────┐
│ shared_history          │
│   turnos previos...     │
│   Comparativa       →   │ ← scroll horizontal
│   ┌───────────────────┐ │
│   │ Plan │ Precio │... │
│   │ Basic│ $9     │... │
│   │ Pro  │ $29    │... │
│   └───────────────────┘ │
│   Desliza para ver más  │
│   (visible para todos)  │
└─────────────────────────┘
```

### 9.5 Image

Componente **pasivo** (narrativa). Render directo a `shared_history`.

- Siempre respetar `alt` cuando exista; si falta, el renderer debe tratar la imagen como decorativa o inferir un texto mínimo seguro.
- Usar `size="full"` solo cuando la imagen sea el contenido principal del mensaje.
- Mantener proporción de aspecto y evitar recortes que oculten información relevante.
- Mostrar `caption` debajo de la imagen con estilo secundario.
- Mientras carga, reservar espacio para evitar saltos en el historial.

```
render directo a shared_history
┌─────────────────────────┐
│ shared_history          │
│   turnos previos...     │
│   ┌───────────────────┐ │
│   │                   │ │
│   │      IMAGEN       │ │ ← proporción estable
│   │                   │ │
│   └───────────────────┘ │
│   Nuevo modelo 2026     │ ← caption
│   (visible para todos)  │
└─────────────────────────┘
```

### 9.6 Media

Componente **pasivo** (narrativa). Render directo a `shared_history`. La reproducción es interacción interna del componente (no emite `[[INTERACTION]]` al agente), por lo que no atraviesa el lifecycle de tarea.

- No reproducir audio o video automáticamente.
- Mostrar controles nativos o equivalentes accesibles para reproducir, pausar y controlar volumen.
- En video, usar `poster` cuando esté disponible para evitar un bloque vacío.
- Mantener el reproductor dentro del ancho del chat y con proporción estable.
- Si falla la carga, mostrar una acción alternativa para abrir o descargar el recurso.

```
render directo a shared_history
┌─────────────────────────┐
│ shared_history          │
│   turnos previos...     │
│   ┌───────────────────┐ │
│   │    POSTER VIDEO   │ │
│   │        ▶          │ │ ← play explícito
│   └───────────────────┘ │
│   0:00 ━━━━━━━ 2:30     │ ← controles media
│   (visible para todos;  │
│    reproducción local)  │
└─────────────────────────┘
```

### 9.7 Divider

Componente **pasivo** (narrativa). Render directo a `shared_history`. Suele aparecer dentro de un mensaje que también contiene una tarea (`buttons`, etc.), en cuyo caso el divider acompaña a la narrativa en el snapshot final.

- Usar `divider` para separar grupos de contenido, no después de cada elemento.
- Si tiene `label`, el texto debe ser corto y secundario.
- Mantener contraste bajo para que no compita con acciones o información principal.
- Evitar varios divisores consecutivos dentro del mismo mensaje.

```
render directo a shared_history (ejemplo con narrativa adyacente)
┌─────────────────────────┐
│ shared_history          │
│   turnos previos...     │
│   Hay una opción        │
│   principal y otras     │
│   alternativas.         │
│   ─── O continúa con ───│ ← divider
│   (visible para todos)  │
└─────────────────────────┘
```

### 9.8 Input

Componente **interactivo** (tarea). Lifecycle completo. `live_capture` abre teclado, selector, GPS o picker de archivo según `type`.

- En móvil, enfocar un campo solo cuando el usuario lo activa explícitamente.
- Usar teclado adecuado según `type`: email, tel, number, date, time, etc.
- Mostrar `label` persistente cuando el valor pueda quedar ambiguo; no depender solo de `placeholder`.
- Validar lo antes posible sin interrumpir la escritura; mostrar `errorMessage` cerca del campo.
- Para `location`, pedir permiso con contexto claro antes de invocar GPS.
- Para `file`, mostrar restricciones de tipo y tamaño antes de abrir el selector.

```
first_rendering (privado al disparador)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Necesito tu correo    │
│   para continuar.       │
├─────────────────────────┤
│ task_controls           │
│   Email                 │ ← label
│  ┌───────────────────┐  │
│  │                   │  │ ← campo vacío
│  └───────────────────┘  │
│                   ┌───┐ │
│                   │ > │ │ Botón enviar (deshabilitado)
│                   └───┘ │
└─────────────────────────┘

live_capture (teclado activo)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Necesito tu correo    │
│   para continuar.       │
├─────────────────────────┤
│ task_controls           │
│   Email                 │
│  ┌───────────────────┐  │
│  │                   │  │ ← campo vacío
│  └───────────────────┘  │
│                   ┌───┐ │
│                   │ > │ │ Botón enviar  (deshabilitado)
│                   └───┘ │
│   Usa un correo válido  │ ← help inline
└─────────────────────────┘
Cuando el usuario escribe en el campo, el componente debe actualizar el valor del campo y mostrar el valor en el campo con el cursor en la posición actual. Además debe habilitar el botón de enviar. Si el valor del campo es inválido, el componente debe mostrar un mensaje de error inline y deshabilitar el botón de enviar.

┌─────────────────────────┐
│  ┌───────────────────┐  │
│  │ pedro@email.com▏ │  │ ← cursor + teclado
│  └───────────────────┘  │
│                   ┌───┐ │
│                   │ > │ │ Botón enviar (habilitado)
│                   └───┘ │
└─────────────────────────┘



final_rendering (snapshot compartido)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
│   Necesito tu correo    │
│   para continuar.       │
│   Usuario:              │ ← snapshot
│   pedro@email.com       │
│   Agente: Correo        │
│   recibido.             │
├─────────────────────────┤
│ task_narrative   (vacío)│
├─────────────────────────┤
│ task_controls    (vacío)│
└─────────────────────────┘
```

### 9.9 Toggle

Componente **interactivo** (tarea). Lifecycle completo. `live_capture` es un tap binario que cambia el estado.

- Usar toggle para estados binarios claros, no para elegir entre varias opciones.
- El `label` debe describir el estado activado de forma inequívoca.
- Mantener toda la fila como área táctil, no solo el switch visual.
- Si `required="true"`, explicar con `error` o texto cercano qué condición debe cumplirse.

```
first_rendering (privado al disparador)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Confirma si aceptas   │
│   los términos.         │
├─────────────────────────┤
│ task_controls           │
│  ┌───────────────────┐  │
│  │ Acepto términos ○ │  │ ← off
│  └───────────────────┘  │
└─────────────────────────┘

live_capture (tap en la fila)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Confirma si aceptas   │
│   los términos.         │
├─────────────────────────┤
│ task_controls           │
│  ┌───────────────────┐  │
│  │ Acepto términos ● │  │ ← on
│  └───────────────────┘  │
└─────────────────────────┘

final_rendering (snapshot compartido)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
│   Confirma si aceptas   │
│   los términos.         │
│   Usuario: true         │ ← snapshot
│   Agente: Gracias,      │
│   continuamos.          │
├─────────────────────────┤
│ task_narrative   (vacío)│
├─────────────────────────┤
│ task_controls    (vacío)│
└─────────────────────────┘
```

### 9.10 Choice

Componente **interactivo** (tarea). Lifecycle completo. En `multi="true"`, `live_capture` puede extenderse mientras el usuario marca/desmarca opciones; cierra en submit explícito.

- Usar `style="buttons"` para 2-5 opciones cortas.
- Usar `style="radio"` cuando convenga ver todas las opciones con más texto.
- Usar `style="dropdown"` cuando haya muchas opciones o el espacio sea limitado.
- Para `multi="true"`, mostrar claramente las opciones seleccionadas y permitir deseleccionar.
- Evitar opciones con textos demasiado parecidos; el usuario debe poder decidir rápido.

```
first_rendering (privado al disparador)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Elige una talla       │
│   disponible.           │
├─────────────────────────┤
│ task_controls           │
│  ┌────┐ ┌────┐ ┌────┐   │
│  │ 40 │ │ 41 │ │ 42 │   │
│  └────┘ └────┘ └────┘   │
└─────────────────────────┘

live_capture (selección)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Elige una talla       │
│   disponible.           │
├─────────────────────────┤
│ task_controls           │
│  ┌────┐ ┌────┐ ┌────┐   │
│  │ 40 │ │ 41 │ │ 42●│   │ ← seleccionada
│  └────┘ └────┘ └────┘   │
│   Seleccionada: 42      │
└─────────────────────────┘

final_rendering (snapshot compartido)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
│   Elige una talla       │
│   disponible.           │
│   Usuario: talla 42     │ ← snapshot
│   Agente: Perfecto,     │
│   aparto talla 42.      │
├─────────────────────────┤
│ task_narrative   (vacío)│
├─────────────────────────┤
│ task_controls    (vacío)│
└─────────────────────────┘
```

### 9.11 Confirm

Componente **interactivo** (tarea). Lifecycle completo. `live_capture` se materializa como un diálogo modal explícito; el chat input persistente queda pausado mientras dura.

- Usar `confirm` para decisiones explícitas que puedan tener consecuencia clara.
- El `title` debe nombrar la decisión, no solo decir "Confirmar".
- El `message` debe explicar la consecuencia en una frase breve.
- Usar `style="danger"` solo para acciones destructivas.
- La acción de cancelar debe estar siempre visible y emitir `[[INTERACTION]]` igual que la confirmación.

```
first_rendering (privado al disparador)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Esta acción requiere  │
│   confirmación.         │
├─────────────────────────┤
│ task_controls           │
│  ╔═══════════════════╗  │
│  ║ ¿Eliminar cuenta? ║  │
│  ║ Se borrarán tus   ║  │
│  ║ datos.            ║  │
│  ║ Sí, eliminar      ║  │ ← confirmed
│  ║ Cancelar          ║  │ ← cancelled
│  ╚═══════════════════╝  │
└─────────────────────────┘

live_capture (modal activo, chat input pausado)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
├─────────────────────────┤
│ task_narrative          │
│   Esta acción requiere  │
│   confirmación.         │
├─────────────────────────┤
│ task_controls           │
│  ╔═══════════════════╗  │
│  ║ ¿Eliminar cuenta? ║  │
│  ║ ...               ║  │
│  ║ Sí, eliminar      ║  │
│  ║ Cancelar      [●] ║  │ ← foco/tap
│  ╚═══════════════════╝  │
└─────────────────────────┘

final_rendering (snapshot compartido, chat input restaurado)
┌─────────────────────────┐
│ shared_history          │
│   ...                   │
│   Esta acción requiere  │
│   confirmación.         │
│   Usuario: cancelled    │ ← snapshot
│   Agente: No eliminé la │
│   cuenta.               │
├─────────────────────────┤
│ task_narrative   (vacío)│
├─────────────────────────┤
│ task_controls    (vacío)│
└─────────────────────────┘
```

---

## 10. Manejo de errores

### 10.1 Errores del agente (PML malformado)

| Error                  | Severidad | Acción del renderer                 |
| ---------------------- | --------- | ----------------------------------- |
| Tag de cierre faltante | Alta      | Intentar autocerrar, si no, truncar |
| ID faltante            | Media     | Generar ID automático               |
| ID duplicado           | Media     | Renderizar el primero               |
| Atributo desconocido   | Baja      | Ignorar                             |
| Valor inválido         | Media     | Usar default                        |
| Hijo no permitido      | Media     | Renderizar como texto               |
| Más de 5 botones       | Media     | Renderizar 5                        |
| Profundidad > 3        | Media     | Aplanar                             |
| PML ilegible           | Alta      | Renderizar como texto plano         |

### 10.2 Errores de recursos

| Error           | Acción                      |
| --------------- | --------------------------- |
| Imagen no carga | Mostrar `alt` + placeholder |
| Media no carga  | Mensaje + botón reintentar  |

### 10.3 Errores de validación (usuario)

| Error                 | Acción                      |
| --------------------- | --------------------------- |
| Campo requerido vacío | Borde rojo + `errorMessage` |
| Validación falla      | Borde rojo + mensaje        |

---

## 11. Versionado y extensibilidad

### 11.1 Versionado

```
PML v1.1 — Mayo 2026
  - 14 componentes standalone
  - Layout responsivo
```

### 11.2 Proceso de extensión

1. Componentes nuevos en versiones minor (v1.1, v1.2...)
2. Cambios de sintaxis solo en major (v2.0)
3. Renderers deben ignorar componentes desconocidos y renderizar como texto

### 11.3 Candidatos para v1.2

| Componente | Descripción | Prioridad |
| ---------- | ----------- | --------- |
| `form`     | Formulario  | Alta      |

---

## Apéndice A — Referencia rápida

```
┌────────────────┬───────────┬────────────────────────────────────────┐
│ Componente      │ Tipo      │ Uso principal                          │
├────────────────┼───────────┼────────────────────────────────────────┤
│ text            │ Standalone│ Texto estático con estilo              │
│ markdown        │ Standalone│ Texto con formato Markdown             │
│ image           │ Standalone│ Imagen estática                        │
│ media           │ Standalone│ Video o audio                          │
│ input           │ Ambos     │ Campo de entrada (+ ref, auto, calc)   │
│ toggle          │ Ambos     │ Interruptor sí/no                      │
│ choice          │ Ambos     │ Selección múltiple                     │
│ opt             │ Choice only│ Opción individual de un choice        │
│ facts           │ Ambos     │ Tabla clave-valor                      │
│ divider         │ Ambos     │ Separador horizontal                   │
│ dtabl           │ Standalone│ Tabla de datos                         │
│ confirm         │ Standalone│ Diálogo de confirmación                │
└────────────────┴───────────┴────────────────────────────────────────┘
```

---

## Apéndice B — EBNF simplificado

```ebnf
message         = { text | markdown | component } ;

component       = self_closing | container ;

self_closing    = "[PML:" TYPE attr_list " /]" ;
container       = "[PML:" TYPE attr_list "]"
                  { child } "[/PML:" TYPE "]" ;

field           = "[PML:input" attr_list " /]"
                | "[PML:toggle" attr_list " /]"
                | choice ;

choice          = "[PML:choice" attr_list "]" { opt } "[/PML:choice]" ;
opt             = "[PML:opt" attr_list " /]" ;

row             = "[PML:row]" { col } "[/PML:row]" ;
col             = "[PML:col" attr_list "]" field "[/PML:col]" ;


focus           = "[PML:focus" attr_list " /]" ;
snapshot        = "[PML:snapshot" attr_list " /]" ;

child           = text | markdown | self_closing | container ;

attr_list       = { ATTR_NAME "=" "\"" ATTR_VALUE "\"" } ;

TYPE            = "text" | "markdown" | "image" | "media" | "input" | "toggle" | "choice" | "opt"
                | "facts" | "divider" | "dtable" | "confirm" ;
```

---

_PML v1.1 — PEPPO Platform — Mayo 2026_
