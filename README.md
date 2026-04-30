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
    4.3  URL
    4.4  Button
    4.5  Buttons
    4.6  Image
    4.7  Media
    4.8  Input
    4.9  Toggle
    4.10 Choice
    4.11 Facts
    4.12 Divider
    4.13 Table
    4.14 Confirm
 5. Form (componente contenedor principal)
    5.1  Estructura general
    5.2  Header
    5.3  Data Sources
    5.4  Components (campos del formulario)
    5.5  Layout (rows y columns)
    5.6  Components Interactions
    5.7  Footer
    5.8  Submit
    5.9  Modos de presentación
    5.10 Ejemplo completo
 6. Modelo de interacción
 7. Reglas de composición
 8. Guías para LLMs / Agentes
 9. Guías de UX para móviles
10. Manejo de errores
11. Versionado y extensibilidad
```

---

## 1. Visión general

PML es un lenguaje **ligero, legible para LLMs y ultra-eficiente en tokens** diseñado para que los agentes conversacionales rendericen componentes UI interactivos dentro de interfaces de chat.

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

[PML:buttons id="acciones"]
  [PML:button id="b1" label="Reservar cita" value="book" style="primary" /]
  [PML:button id="b2" label="Ver precios" value="prices" /]
[/PML:buttons]
```

**Ejemplo con comillas simples (valor contiene comillas dobles):**

```
[PML:button id="c" label='Dijo: "hola"' value="confirm" style="primary" /]
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

Estos componentes se usan **fuera de un Form**, directamente en el flujo del chat.

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

### 4.3 URL

Enlace clickeable.

| Atributo | Requerido | Tipo                               | Descripción                                         |
| -------- | --------- | ---------------------------------- | --------------------------------------------------- |
| `id`     | No        | string                             | Identificador                                       |
| `href`   | Sí        | url                                | Destino del enlace                                  |
| `label`  | Sí        | string                             | Texto visible                                       |
| `style`  | No        | `primary` / `secondary` / `subtle` | Estilo visual. Default: `secondary`.                |
| `icon`   | No        | string                             | Icono predefinido (`external`, `download`, `copy`). |

**Ejemplo:**

```
[PML:url id="ver-factura" href="https://cdn.peppo.com/factura-001.pdf"
  label="Descargar factura" style="primary" icon="download" /]
```

**Interacción generada:**

```
[[INTERACTION]] id=ver-factura value=https://cdn.peppo.com/factura-001.pdf
```

---

### 4.4 Button

Botón individual.

| Atributo      | Requerido | Tipo                                          | Descripción                                |
| ------------- | --------- | --------------------------------------------- | ------------------------------------------ |
| `id`          | Sí        | string                                        | Identificador único                        |
| `label`       | Sí        | string                                        | Texto visible                              |
| `value`       | Sí        | string                                        | Valor enviado al agente                    |
| `style`       | No        | `primary` / `secondary` / `danger` / `subtle` | Estilo. Default: `secondary`.              |
| `iconUrl`     | No        | url                                           | Icono antes del label                      |
| `loading`     | No        | `true` / `false`                              | Spinner + deshabilitado. Default: `false`. |
| `confirmText` | No        | string                                        | Diálogo de confirmación antes de enviar    |

**Ejemplo:**

```
[PML:button id="confirmar" label="Sí, confirmar pago" value="confirmed" style="primary" /]
```

**Interacción generada:**

```
[[INTERACTION]] id=confirmar value=confirmed
```

---

### 4.5 Buttons

Fila de botones. Máximo **5**.

| Atributo | Requerido | Tipo                        | Descripción                  |
| -------- | --------- | --------------------------- | ---------------------------- |
| `id`     | Sí        | string                      | Identificador del grupo      |
| `align`  | No        | `left` / `center` / `right` | Alineación. Default: `left`. |

**Ejemplo:**

```
[PML:buttons id="post-compra"]
  [PML:button id="track" label="Rastrear pedido" value="track" style="primary" /]
  [PML:button id="continue" label="Seguir comprando" value="continue" /]
[/PML:buttons]
```

---

### 4.6 Image

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

### 4.7 Media

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

### 4.8 Input

Campo de entrada de datos.

**Tipos disponibles fuera de Form:**

`text` / `number` / `email` / `tel` / `url` / `password` / `date` / `time` / `textarea` / `location` / `file`

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

### 4.11 Facts

Tabla de pares clave-valor.

| Atributo  | Requerido | Tipo      | Descripción                       |
| --------- | --------- | --------- | --------------------------------- |
| `id`      | No        | string    | Identificador                     |
| `columns` | No        | `1` / `2` | Columnas de layout. Default: `1`. |

**Formato del contenido:** Una línea por par, `Clave: valor`.

**Ejemplo:**

```
[PML:facts columns="2"]
Producto: Nike Air Max 90
Talla: 42
Precio: $2,499 MXN
Envío: Gratis
[/PML:facts]
```

---

### 4.12 Divider

Línea separadora.

```
[PML:divider /]
```

Con texto:

```
[PML:divider label="O continúa con" /]
```

---

### 4.13 Table

Tabla de datos.

| Atributo    | Requerido | Tipo   | Descripción                                       |
| ----------- | --------- | ------ | ------------------------------------------------- |
| `id`        | Sí        | string | Identificador único                               |
| `columns`   | Sí        | string | Nombres de columnas separados por coma            |
| `highlight` | No        | string | Valor de la primera columna de la fila a resaltar |

**Formato del contenido:** Cada línea es una fila, valores separados por coma.

**Ejemplo:**

```
[PML:table id="comparativa" columns="Plan,Precio,Agentes,Mensajes" highlight="Pro"]
Básico,$9/mes,1,100/día
Pro,$29/mes,10,Ilimitado
Enterprise,Custom,Ilimitado,Ilimitado
[/PML:table]
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

```
[[CONFIRM]] id=eliminar-cuenta value=confirmed
[[CONFIRM]] id=eliminar-cuenta value=cancelled
```

---

## 5. Form — Componente contenedor principal

El `Form` es el componente más complejo y poderoso de PML. No es solo un contenedor de inputs — es un **mini motor de lógica declarativa** que maneja:

- Campos con dependencias entre sí
- Visibilidad condicional
- Auto-populado desde fuentes de datos
- Campos calculados
- Validación en cascada
- Layout responsivo
- Condición de submit

**La lógica de negocio vive en el renderer (Chat App), no en el agente.**

---

### 5.1 Estructura general

```
[PML:form id="..." mode="..." submitLabel="..." ...]

  [PML:header]
    ...
  [/PML:header]

  [PML:body]

    [PML:source ... /]

    [PML:input ... /]
    [PML:toggle ... /]
    [PML:choice ... /]
    ...

    [PML:row]
      [PML:col ...] ... [/PML:col]
    [/PML:row]

    [PML:interactions]
      [PML:acceptation ... /]
      [PML:visibility ... /]
    [/PML:interactions]

    [PML:button subtype="submit" ... /]

  [/PML:body]

  [PML:footer]
    ...
  [/PML:footer]

[/PML:form]
```

**Atributos del Form:**

| Atributo      | Requerido | Tipo                           | Descripción                                        |
| ------------- | --------- | ------------------------------ | -------------------------------------------------- |
| `id`          | Sí        | string                         | Identificador único                                |
| `mode`        | No        | `inline` / `stepped` / `modal` | Presentación. Default: `inline`.                   |
| `submitLabel` | No        | string                         | Texto del botón enviar. Default: `"Enviar"`.       |
| `cancelLabel` | No        | string                         | Texto del botón cancelar. Default: `"Cancelar"`.   |
| `showSummary` | No        | `true` / `false`               | Resumen post-envío. Default: `true`.               |
| `language`    | No        | `es` / `en`                    | Idioma de labels y mensajes. Default: `es`.        |
| `autoSave`    | No        | `true` / `false`               | Guardado automático de borrador. Default: `false`. |

---

### 5.2 Header

Sección superior del formulario con título y descripción.

```
[PML:header]
  [PML:text type="title" size="large" style="bold"]
    Crear Bitácora de Nómina
  [/PML:text]
  [PML:text type="paragraph" style="muted" size="small"]
    Registra las horas trabajadas por los empleados,
    incluyendo detalles de órdenes de trabajo y clientes.
  [/PML:text]
[/PML:header]
```

**Hijos permitidos:** `Text`, `Markdown`, `Image`, `URL`, `Badge`

---

### 5.3 Data Sources

Definiciones de fuentes de datos para campos de tipo `reference` (búsqueda con datos externos).

| Atributo  | Requerido | Tipo   | Descripción                                             |
| --------- | --------- | ------ | ------------------------------------------------------- |
| `id`      | Sí        | string | Identificador de la fuente                              |
| `table`   | Sí        | string | Nombre de la tabla/colección                            |
| `fields`  | Sí        | string | Campos a retornar, separados por coma                   |
| `idField` | No        | string | Campo identificador. Default: primer campo de `fields`. |
| `search`  | No        | string | Campos donde buscar, separados por coma                 |
| `sort`    | No        | string | Ordenamiento. Formato: `campo:DIRECCION` (ASC/DESC).    |
| `filter`  | No        | string | Filtros. Formato: `campo:valor`.                        |
| `top`     | No        | number | Máximo de resultados. Default: `10`.                    |

**Ejemplo:**

```
[PML:source id="empleados" table="empleados"
  fields="id,nombre,apellidos"
  search="nombre,apellidos"
  sort="apellidos:ASC,nombre:ASC"
  filter="activo:true" top="3" /]

[PML:source id="ordenes_trabajo" table="ordenes_de_trabajo"
  fields="folio,cliente,numero_de_parte"
  search="folio,cliente"
  sort="folio:ASC"
  filter="activo:true" top="3" /]
```

**Nota:** Los `source` son declaraciones. La Chat App es la que ejecuta la consulta real contra la base de datos o API. El agente solo describe **qué** datos necesita, no **cómo** obtenerlos.

---

### 5.4 Components (campos del formulario)

Dentro de un Form, los campos tienen una estructura enriquecida. Cada campo puede tener:

- **type** — qué tipo de componente es
- **behavior** — cómo se comporta
- **look_and_feel** — cómo se ve y qué textos muestra
- **validation** — qué reglas cumple y qué mensajes muestra

#### 5.4.1 Tipos de campo dentro de Form

| Tipo              | Descripción                   | Ejemplo de uso           |
| ----------------- | ----------------------------- | ------------------------ |
| `text`            | Texto libre                   | Nombre, dirección        |
| `number`          | Numérico                      | Cantidad, edad           |
| `email`           | Email                         | Correo electrónico       |
| `tel`             | Teléfono                      | Número telefónico        |
| `password`        | Contraseña                    | Clave secreta            |
| `date`            | Fecha                         | Fecha de nacimiento      |
| `time`            | Hora                          | Hora de inicio/fin       |
| `textarea`        | Texto largo                   | Comentarios, notas       |
| `location`        | Ubicación GPS                 | Dirección, coordenadas   |
| `file`            | Archivo                       | Foto, documento          |
| `toggle`          | Sí/no                         | Acepto términos          |
| `choice`          | Selección                     | País, talla, categoría   |
| `reference`       | Búsqueda con fuente de datos  | Empleado, producto       |
| `auto_populate`   | Auto-llenado desde otro campo | Cliente, número de parte |
| `calculated`      | Valor computado               | Horas trabajadas, total  |
| `button` (submit) | Botón de envío                | Guardar, enviar          |

#### 5.4.2 Campo estándar (input, toggle, choice)

```
[PML:input id="fecha" type="date"
  label="Fecha"
  placeholder="Selecciona una fecha"
  helpText="Fecha en que se realizó el trabajo"
  default="today"
  required="true"
  errorMessage="La fecha es obligatoria" /]
```

**Atributos extendidos dentro de Form (adicionales a los de standalone):**

| Atributo   | Tipo             | Descripción                                       |
| ---------- | ---------------- | ------------------------------------------------- |
| `helpText` | string           | Texto de ayuda debajo del campo                   |
| `default`  | string           | Valor por defecto (`today`, `now`, valor literal) |
| `readonly` | `true` / `false` | Solo lectura. Default: `false`.                   |

#### 5.4.3 Campo `reference` (búsqueda con data source)

Se usa para buscar y seleccionar datos de una fuente externa.

```
[PML:input id="empleado" type="reference"
  source="empleados"
  display="{{nombre}} {{apellidos}}"
  searchMinChars="2"
  label="Empleado"
  placeholder="Buscar empleado..."
  helpText="Seleccione el empleado que realizó el trabajo"
  required="true"
  errorMessage="Debe seleccionar un empleado" /]
```

| Atributo adicional | Tipo   | Descripción                                               |
| ------------------ | ------ | --------------------------------------------------------- |
| `source`           | string | Referencia al `id` de un `[PML:source]`                   |
| `display`          | string | Template de visualización con `{{campo}}`                 |
| `searchMinChars`   | number | Mínimo de caracteres para activar búsqueda. Default: `2`. |

**Interacción generada al seleccionar:**

```
[[INTERACTION]] id=empleado value=3
```

El valor es el `idField` del registro seleccionado.

#### 5.4.4 Campo `auto_populate` (auto-llenado)

Se llena automáticamente cuando otro campo cambia.

```
[PML:input id="cliente" type="auto_populate"
  trigger="folio"
  display="{{cliente}}"
  label="Cliente"
  placeholder="Cliente asociado"
  helpText="Se llena al seleccionar la orden de trabajo"
  readonly="true" /]
```

| Atributo adicional | Tipo   | Descripción                                        |
| ------------------ | ------ | -------------------------------------------------- |
| `trigger`          | string | `id` del campo que dispara el auto-llenado         |
| `display`          | string | Template con `{{campo}}` del registro seleccionado |

**Regla del renderer:**
Cuando el campo `trigger` acepta un valor, el renderer busca el registro correspondiente en el data source y llena este campo con el valor del campo indicado en `display`.

#### 5.4.5 Campo `calculated` (calculado)

Su valor se computa a partir de otros campos.

```
[PML:input id="horas_trabajadas" type="calculated"
  triggerFields="horario_inicio,horario_fin"
  formula="(horario_fin - horario_inicio) / 3600"
  precision="2"
  label="Horas Trabajadas"
  suffix="hrs"
  helpText="Calculado automáticamente"
  readonly="true"
  errorMessage="Las horas deben ser entre 0.25 y 24"
  validate="min:0.25,max:24" /]
```

| Atributo adicional | Tipo   | Descripción                             |
| ------------------ | ------ | --------------------------------------- |
| `triggerFields`    | string | Campos que disparan el recálculo        |
| `formula`          | string | Expresión de cálculo usando `{{campo}}` |
| `precision`        | number | Decimales a mostrar                     |

**Regla del renderer:**
Cuando cualquiera de los `triggerFields` cambia de valor, el renderer recalcula la fórmula y actualiza el valor del campo.

#### 5.4.6 Botón submit

```
[PML:button id="submit" subtype="submit"
  label="Guardar Bitácora"
  style="primary"
  confirmTitle="Confirmar guardado"
  confirmMessage="¿Estás seguro de que deseas guardar esta bitácora?"
  confirmLabel="Sí, guardar"
  cancelLabel="Cancelar" /]
```

| Atributo adicional | Tipo     | Descripción                                      |
| ------------------ | -------- | ------------------------------------------------ |
| `subtype`          | `submit` | Marca este botón como el botón de envío del form |
| `confirmTitle`     | string   | Título del diálogo de confirmación               |
| `confirmMessage`   | string   | Mensaje del diálogo                              |
| `confirmLabel`     | string   | Botón de confirmación                            |
| `cancelLabel`      | string   | Botón de cancelación                             |

**Regla del renderer:**
El botón submit solo se habilita cuando **todos los campos visibles están aceptados** (ver sección 5.6).

---

### 5.5 Layout (rows y columns)

Organiza los campos en filas y columnas responsivas.

```
[PML:row]
  [PML:col mobile="12" tablet="6" desktop="4"]
    [PML:input id="horario_inicio" type="time" label="Hora de Inicio" /]
  [/PML:col]
  [PML:col mobile="12" tablet="6" desktop="4"]
    [PML:input id="horario_fin" type="time" label="Hora de Fin" /]
  [/PML:col]
  [PML:col mobile="12" tablet="12" desktop="4"]
    [PML:input id="horas_trabajadas" type="calculated" ... /]
  [/PML:col]
[/PML:row]
```

**Atributos de `col`:**

| Atributo  | Requerido | Tipo     | Descripción                      |
| --------- | --------- | -------- | -------------------------------- |
| `mobile`  | No        | `1`-`12` | Ancho en móvil. Default: `12`.   |
| `tablet`  | No        | `1`-`12` | Ancho en tablet. Default: `12`.  |
| `desktop` | No        | `1`-`12` | Ancho en desktop. Default: `12`. |

**Sistema de columnas:** Basado en grid de 12 columnas (estándar CSS).

**Reglas:**

- Si no se especifica layout, los campos se apilan verticalmente (una columna)
- En móvil, cualquier columna > 6 se colapsa a 12 automáticamente
- `PML:row` y `PML:col` solo existen dentro de `PML:body` de un Form
- Cada `PML:col` contiene exactamente **un** componente

---

### 5.6 Components Interactions

Esta es la sección de **lógica declarativa** del formulario. El renderer la evalúa en tiempo real mientras el usuario interactúa.

#### 5.6.1 Acceptation Rules

Define **cuándo un componente se considera "aceptado"**.

```
[PML:interactions]
  [PML:acceptation id="submit" when="fecha,empleado,horario_inicio,horario_fin,folio" /]
  [PML:acceptation id="horas_trabajadas" when="horario_inicio,horario_fin" /]
  [PML:acceptation id="cliente" when="folio" /]
  [PML:acceptation id="numero_de_parte" when="folio" /]
[/PML:interactions]
```

**Sintaxis:**

```
[PML:acceptation id="campo_objetivo" when="campo1,campo2,..." /]
```

**Significado:** El campo `campo_objetivo` se considera aceptado cuando **todos** los campos listados en `when` están aceptados.

**Ejemplo interpretado:**

```
[PML:acceptation id="submit" when="fecha,empleado,horario_inicio,horario_fin,folio" /]
```

> El botón submit se habilita solo cuando fecha, empleado, horario_inicio, horario_fin Y folio están todos aceptados.

#### 5.6.2 Visibility Rules

Define **cuándo un componente se muestra u oculta**.

```
[PML:visibility id="horas_trabajadas" showWhen="horario_inicio,horario_fin" /]
[PML:visibility id="cliente" showWhen="folio" /]
[PML:visibility id="numero_de_parte" showWhen="folio" /]
```

**Sintaxis:**

```
[PML:visibility id="campo_objetivo" showWhen="campo1,campo2,..." /]
```

**Significado:** El campo `campo_objetivo` se muestra solo cuando **todos** los campos listados en `showWhen` están aceptados.

**Ejemplo interpretado:**

```
[PML:visibility id="cliente" showWhen="folio" /]
```

> El campo "cliente" solo aparece cuando el campo "folio" ha sido aceptado (el usuario seleccionó una orden de trabajo válida).

#### 5.6.3 Regla de aceptación (definición formal)

```
Un componente se considera "ACEPTADO" cuando:

  1. Está OCULTO (visible="false" o su visibility rule no se cumple)
     → Se acepta AUTOMÁTICAMENTE

  2. Está VISIBLE y CUMPLE todas sus reglas de validación individuales
     → Se acepta por VALIDACIÓN

Un componente NO aceptado:
  - Está visible
  - Y tiene al menos una regla de validación sin cumplir
```

#### 5.6.4 Orden de evaluación del renderer

```
Paso 1: Evaluar visibilidad de componentes SIN dependencias
        (los que no tienen visibility rules → siempre visibles)

Paso 2: Evaluar validación de componentes visibles
        → Determinar aceptación individual de cada uno

Paso 3: Evaluar visibilidad de componentes CON dependencias
        (usando la aceptación de los campos del paso 2)

Paso 4: Re-evaluar aceptación considerando campos ocultos
        (ocultos = aceptados automáticamente)

Paso 5: Evaluar acceptation_rules del submit
        → Habilitar o deshabilitar botón de envío

Este ciclo se repite CADA VEZ que el usuario modifica un campo.
```

#### 5.6.5 Ejemplo de flujo de evaluación

```
Estado inicial:
  fecha: visible, vacío → NO aceptado
  empleado: visible, vacío → NO aceptado
  horario_inicio: visible, vacío → NO aceptado
  horario_fin: visible, vacío → NO aceptado
  horas_trabajadas: OCULTO (horario_inicio y horario_fin no aceptados) → aceptado automáticamente
  folio: visible, vacío → NO aceptado
  cliente: OCULTO (folio no aceptado) → aceptado automáticamente
  numero_de_parte: OCULTO (folio no aceptado) → aceptado automáticamente
  submit: DESHABILITADO (fecha, empleado, horario_inicio, horario_fin, folio no aceptados)

El usuario selecciona horario_inicio = "08:00":
  horario_inicio: VISIBLE, válido → ACEPTADO
  horas_trabajadas: sigue OCULTO (horario_fin no aceptado) → aceptado automáticamente
  submit: sigue DESHABILITADO

El usuario selecciona horario_fin = "17:00":
  horario_fin: VISIBLE, válido → ACEPTADO
  horas_trabajadas: ambos triggers aceptados → SE MUESTRA → se calcula = 9 hrs → ACEPTADO
  submit: sigue DESHABILITADO (faltan fecha, empleado, folio)

El usuario selecciona folio = "OT-001":
  folio: VISIBLE, válido → ACEPTADO
  cliente: folio aceptado → SE MUESTRA → auto-poblado = "Empresa A" → ACEPTADO
  numero_de_parte: folio aceptado → SE MUESTRA → auto-poblado = "NP-123" → ACEPTADO
  submit: sigue DESHABILITADO (faltan fecha, empleado)

El usuario selecciona fecha y empleado:
  fecha: ACEPTADO
  empleado: ACEPTADO
  submit: TODOS los campos requeridos aceptados → HABILITADO ✓
```

---

### 5.7 Footer

Sección inferior del formulario.

```
[PML:footer]
  [PML:text type="muted" size="small" align="right"]
    Desarrollado por PEPPO
  [/PML:text]
[/PML:footer]
```

**Hijos permitidos:** `Text`, `Markdown`, `URL`

---

### 5.8 Submit — Flujo completo

```
1. Usuario presiona submit (habilitado)
2. Si confirmTitle está definido → mostrar diálogo de confirmación
3. Si confirma → enviar [[FORM_SUBMIT]]
4. Si showSummary="true" → renderer muestra resumen inline
5. Chat input del chat se restaura
6. Agente continúa la conversación
```

**Interacción generada:**

```
[[FORM_SUBMIT]] id=bitacora&fecha=2026-04-20&empleado=3&horario_inicio=08:00&horario_fin=17:00&horas_trabajadas=9.00&folio=OT-001&cliente=Empresa+A&numero_de_parte=NP-123
```

**Si el usuario cancela el formulario en cualquier momento:**

```
[[INTERACTION]] id=bitacora value=cancelled
```

---

### 5.9 Modos de presentación

#### `mode="inline"` (default)

Todos los campos visibles simultáneamente, scrollable. Ideal para 1-3 campos.

```
[PML:form id="contacto" mode="inline" submitLabel="Enviar"]
  [PML:input id="nombre" type="text" label="Nombre" required="true" /]
  [PML:input id="email" type="email" label="Email" required="true" /]
[/PML:form]
```

#### `mode="stepped"`

Un campo a la vez con navegación secuencial. Ideal para 3+ campos en móvil.

```
[PML:form id="registro" mode="stepped" title="Crear cuenta"
  submitLabel="Crear cuenta" cancelLabel="Volver al chat"]
  [PML:input id="nombre" type="text" label="¿Cuál es tu nombre?" required="true" /]
  [PML:input id="email" type="email" label="¿Tu email?" required="true" /]
  [PML:input id="telefono" type="tel" label="¿Tu teléfono?" prefix="+52" required="true" /]
[/PML:form]
```

**Flujo stepped:**

```
Paso 1: Nombre    → [Siguiente] → valida required
Paso 2: Email     → [Siguiente] → valida required + formato
Paso 3: Teléfono  → [Crear cuenta] → valida required
```

#### `mode="modal"`

Bottom sheet modal. Chat oscurecido detrás. Ideal para formularios largos o complejos.

```
[PML:form id="solicitud" mode="modal" title="Solicitud de crédito"
  submitLabel="Enviar solicitud"]
  ...
[/PML:form]
```

---

### 5.10 Ejemplo completo del JSON de referencia traducido a PML

Este es el equivalente PML del formulario de bitácora de nómina que se compartió como referencia:

```
[PML:form id="bitacora_nomina" mode="inline"
  submitLabel="Guardar Bitácora" showSummary="true" language="es"]

  [PML:header]
    [PML:text type="title" size="large" style="bold"]
      Crear Bitácora de Nómina
    [/PML:text]
    [PML:text type="paragraph" style="muted" size="small"]
      Registra las horas trabajadas por los empleados,
      incluyendo detalles de órdenes de trabajo y clientes.
    [/PML:text]
  [/PML:header]

  [PML:body]

    [PML:source id="empleados" table="empleados"
      fields="id,nombre,apellidos" idField="id"
      search="nombre,apellidos"
      sort="apellidos:ASC,nombre:ASC"
      filter="activo:true" top="3" /]

    [PML:source id="ordenes_trabajo" table="ordenes_de_trabajo"
      fields="folio,cliente,numero_de_parte" idField="folio"
      search="folio,cliente"
      sort="folio:ASC"
      filter="activo:true" top="3" /]

    [PML:input id="fecha" type="date"
      label="Fecha"
      placeholder="Seleccione una fecha"
      helpText="Fecha en que se realizó el trabajo"
      default="today"
      required="true"
      errorMessage="La fecha es obligatoria" /]

    [PML:input id="empleado" type="reference"
      source="empleados"
      display="{{nombre}} {{apellidos}}"
      searchMinChars="2"
      label="Empleado"
      placeholder="Buscar empleado..."
      helpText="Seleccione el empleado que realizó el trabajo"
      required="true"
      errorMessage="Debe seleccionar un empleado" /]

    [PML:row]
      [PML:col mobile="12" tablet="6" desktop="4"]
        [PML:input id="horario_inicio" type="time"
          label="Hora de Inicio"
          placeholder="00:00"
          helpText="Hora en que comenzó el trabajo"
          format="12h"
          required="true"
          errorMessage="La hora de inicio es obligatoria" /]
      [/PML:col]
      [PML:col mobile="12" tablet="6" desktop="4"]
        [PML:input id="horario_fin" type="time"
          label="Hora de Fin"
          placeholder="00:00"
          helpText="Hora en que finalizó el trabajo"
          format="12h"
          required="true"
          after="horario_inicio"
          errorMessage="La hora de fin debe ser posterior a la hora de inicio" /]
      [/PML:col]
      [PML:col mobile="12" tablet="12" desktop="4"]
        [PML:input id="horas_trabajadas" type="calculated"
          triggerFields="horario_inicio,horario_fin"
          formula="(horario_fin - horario_inicio) / 3600"
          precision="2"
          label="Horas Trabajadas"
          suffix="hrs"
          helpText="Calculado automáticamente"
          readonly="true"
          validate="min:0.25,max:24"
          errorMessage="Las horas deben ser entre 0.25 y 24" /]
      [/PML:col]
    [/PML:row]

    [PML:row]
      [PML:col mobile="12" tablet="12" desktop="4"]
        [PML:input id="folio" type="reference"
          source="ordenes_trabajo"
          display="{{folio}} - {{cliente}}"
          searchMinChars="1"
          label="Orden de Trabajo"
          placeholder="Buscar orden de trabajo..."
          helpText="Seleccione la orden de trabajo asociada"
          required="true"
          errorMessage="Debe seleccionar una orden de trabajo" /]
      [/PML:col]
      [PML:col mobile="12" tablet="6" desktop="4"]
        [PML:input id="cliente" type="auto_populate"
          trigger="folio"
          display="{{cliente}}"
          label="Cliente"
          placeholder="Cliente asociado"
          helpText="Se llena al seleccionar la orden de trabajo"
          readonly="true" /]
      [/PML:col]
      [PML:col mobile="12" tablet="6" desktop="4"]
        [PML:input id="numero_de_parte" type="auto_populate"
          trigger="folio"
          display="{{numero_de_parte}}"
          label="Número de Parte"
          placeholder="Número de parte asociado"
          helpText="Se llena al seleccionar la orden de trabajo"
          readonly="true" /]
      [/PML:col]
    [/PML:row]

    [PML:interactions]
      [PML:acceptation id="submit"
        when="fecha,empleado,horario_inicio,horario_fin,folio" /]
      [PML:acceptation id="horas_trabajadas"
        when="horario_inicio,horario_fin" /]
      [PML:acceptation id="cliente" when="folio" /]
      [PML:acceptation id="numero_de_parte" when="folio" /]
      [PML:visibility id="horas_trabajadas"
        showWhen="horario_inicio,horario_fin" /]
      [PML:visibility id="cliente" showWhen="folio" /]
      [PML:visibility id="numero_de_parte" showWhen="folio" /]
    [/PML:interactions]

    [PML:button id="submit" subtype="submit"
      label="Guardar Bitácora"
      style="primary"
      confirmTitle="Confirmar guardado"
      confirmMessage="¿Estás seguro de que deseas guardar esta bitácora de nómina?"
      confirmLabel="Sí, guardar"
      cancelLabel="Cancelar" /]

  [/PML:body]

[/PML:form]
```

**Comparación de tamaño:**

```
JSON original:     ~8,500 caracteres
PML equivalente:   ~3,200 caracteres

Reducción: ~62% menos tokens
```

---

## 6. Modelo de interacción

### 6.1 Formatos de interacción

| Formato                                            | Cuándo se usa                                    |
| -------------------------------------------------- | ------------------------------------------------ |
| `[[INTERACTION]] id=X value=Y`                     | Botón, choice, toggle, rating, URL, tab, section |
| `[[FORM_SUBMIT]] id=X&campo1=valor1&campo2=valor2` | Envío de formulario                              |
| `[[CONFIRM]] id=X value=confirmed`                 | Confirmación aceptada                            |
| `[[CONFIRM]] id=X value=cancelled`                 | Confirmación rechazada                           |
| `[[CHANGE]] id=X value=Y`                          | Cambio en tiempo real (solo si `live="true"`)    |

### 6.2 Flujo completo

```
┌──────────┐        PML         ┌──────────┐
│          │ ─────────────────→  │          │
│  AGENTE  │                     │ CHAT APP │
│          │ ←─────────────────  │(renderer)│
└──────────┘   [[INTERACTION]]  └──────────┘
               [[FORM_SUBMIT]]
               [[CONFIRM]]
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

### 6.3 Ejemplo de conversación completa

```
USUARIO: Quiero comprar unos zapatos

AGENTE: ¡Perfecto! Tenemos varios modelos disponibles.

        [PML:image url="https://cdn.peppo.com/catalogo.jpg"
          alt="Catálogo de zapatos" size="full" /]

        ¿Cuál te interesa?

        [PML:buttons id="catalogo"]
          [PML:button id="nike" label="Nike Air Max — $2,499" value="nike" style="primary" /]
          [PML:button id="adidas" label="Adidas Ultraboost — $3,199" value="adidas" /]
        [/PML:buttons]

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

        [PML:button id="start-checkout" label="Completar datos"
          value="start" style="primary" /]

--- El usuario toca el botón ---

USUARIO: [[INTERACTION]] id=start-checkout value=start

AGENTE: [PML:form id="checkout" mode="stepped"
          submitLabel="Confirmar compra" showSummary="true"]

          [PML:header]
            [PML:text type="title" style="bold"]Datos de envío[/PML:text]
          [/PML:header]

          [PML:body]
            [PML:input id="nombre" type="text" label="Nombre completo"
              required="true" errorMessage="El nombre es obligatorio" /]
            [PML:input id="direccion" type="text" label="Dirección"
              required="true" errorMessage="La dirección es obligatoria" /]
            [PML:input id="ciudad" type="text" label="Ciudad"
              required="true" /]
            [PML:input id="cp" type="text" label="Código postal"
              validate="exactLength:5" required="true"
              errorMessage="El CP debe tener 5 dígitos" /]
            [PML:input id="telefono" type="tel" label="Teléfono"
              prefix="+52" required="true" /]

            [PML:button id="submit" subtype="submit"
              label="Confirmar compra" style="primary"
              confirmTitle="Confirmar compra"
              confirmMessage="¿Confirmas tu compra de Nike Air Max 90, talla 42?"
              confirmLabel="Sí, comprar" cancelLabel="Revisar datos" /]
          [/PML:body]

        [/PML:form]

--- El usuario completa paso a paso y envía ---

USUARIO: [[FORM_SUBMIT]] id=checkout&nombre=Pedro+García&direccion=Reforma+123&ciudad=CDMX&cp=06600&telefono=5551234567

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

        [PML:buttons id="post-compra"]
          [PML:button id="track" label="Rastrear pedido" value="track" style="primary" /]
          [PML:button id="continue" label="Seguir comprando" value="continue" /]
        [/PML:buttons]
```

---

## 7. Reglas de composición

### 7.1 Tabla de nesting permitido

```
┌──────────────────┬─────────────────────────────────────────────────┐
│ Contenedor        │ Puede contener                                   │
├──────────────────┼─────────────────────────────────────────────────┤
│ (raíz del mensaje)│ Todos los componentes standalone + Form          │
│ buttons           │ button (máx 5)                                   │
│ choice            │ opt (uno o más)                                  │
│ form > header     │ text, markdown, image, url, badge                │
│ form > body       │ source, input, toggle, choice, facts, divider,   │
│                   │ row, interactions, button (submit)               │
│ form > body > row │ col                                              │
│ form > body > col │ input, toggle, choice, facts, divider            │
│ form > footer     │ text, markdown, url                              │
└──────────────────┴─────────────────────────────────────────────────┘
```

### 7.2 Reglas de anidación

1. **Profundidad máxima fuera de Form:** 2 niveles (buttons > button)
2. **Profundidad máxima dentro de Form:** 4 niveles (form > body > row > col > input)
3. **Prohibiciones:**
    - `form` dentro de `form`
    - `form` dentro de `buttons`
    - `row` fuera de `form`

### 7.3 Reglas de IDs

- Únicos dentro de un mensaje
- Formato: `kebab-case`
- No espacios, no caracteres especiales, no empezar con números
- IDs duplicados: renderizar solo el primero, loggear warning

---

## 8. Guías para LLMs / Agentes

### 8.1 Reglas obligatorias

1. **Siempre** incluir `id` en todo componente interactivo
2. **Nunca** usar JSON, `< >` ni HTML para componentes
3. Máximo **5 botones** por `[PML:buttons]`
4. Para formularios con 3+ campos, usar `[PML:form]`
5. Usar `style="primary"` solo para la acción principal
6. **Antes** de mostrar un formulario, preparar al usuario con un botón de activación
7. Dentro de un Form, usar `[PML:interactions]` para declarar dependencias entre campos
8. No anidar más de 4 niveles dentro de Form

### 8.2 Selección de modo de Form

```
¿Cuántos campos?
  │
  ├── 1-2 campos simples ──→ mode="inline"
  │
  ├── 3-5 campos ──────────→ mode="stepped"
  │
  └── 5+ campos o complejo ─→ mode="modal"
```

### 8.3 Cuándo usar cada tipo de campo

```
¿El usuario escribe libremente?
  └── Sí → text, number, email, tel, textarea

¿El usuario selecciona de una lista fija?
  └── Sí → choice (buttons/dropdown/radio)

¿El usuario busca en datos externos?
  └── Sí → type="reference" + source

¿Un campo depende de otro?
  └── Llenado automático → type="auto_populate" + trigger
  └── Cálculo → type="calculated" + formula + triggerFields

¿El usuario activa/desactiva?
  └── toggle

¿El usuario sube un archivo?
  └── type="file"

¿El usuario comparte ubicación?
  └── type="location"

¿El usuario elige fecha/hora?
  └── type="date" / type="time"
```

### 8.4 Generación tolerante a errores

| Error del agente             | Comportamiento del renderer         |
| ---------------------------- | ----------------------------------- |
| Falta `id` en interactivo    | Generar `auto-{tipo}-{n}`           |
| Atributo desconocido         | Ignorar silenciosamente             |
| Tag sin cerrar               | Intentar autocerrar; si no, truncar |
| Más de 5 botones             | Renderizar 5, ignorar resto         |
| Componente hijo no permitido | Renderizar como texto plano         |
| ID duplicado                 | Renderizar el primero               |
| Profundidad > 4              | Aplanar al nivel 4                  |

---

## 9. Guías de UX para móviles

### 9.1 Principios

```
1. EL CHAT INPUT ES SAGRADO
   Solo desaparece cuando un Form está activo.
   Se restaura inmediatamente después del envío o cancelación.

2. NUNCA SORPRENDER
   El agente siempre prepara al usuario antes de mostrar un Form.
   El usuario elige cuándo activarlo.

3. UNA COSA A LA VEZ
   En móvil, mode="stepped" es la mejor experiencia para 3+ campos.

4. CANCELAR SIEMPRE ES POSIBLE
   ✕ arriba, swipe down, o botón "Volver al chat".

5. NADA SE PIERDE
   Los datos enviados quedan como resumen inline en el historial.
```

### 9.2 Flujo de Form en chat

```
ESTADO 1 — PREPARACIÓN
┌─────────────────────────┐
│                         │
│  [Historial de chat]    │
│                         │
│  Agente: Necesito tus   │
│  datos para continuar.  │
│                         │
│  ┌───────────────────┐  │
│  │ Completar datos   │  │ ← Botón de activación
│  └───────────────────┘  │
│                         │
│ ┌─────────────────────┐ │
│ │ Escribe un mensaje  │ │ ← Chat input ACTIVO
│ └─────────────────────┘ │
└─────────────────────────┘

ESTADO 2 — FORMULARIO ACTIVO
┌─────────────────────────┐
│  Datos de envío      ✕  │ ← Cerrar/cancelar
│                         │
│  Nombre                 │
│  ┌───────────────────┐  │
│  │ Pedro García      │  │ ← Campo activo + teclado
│  └───────────────────┘  │
│                         │
│  ┌───────────────────┐  │
│  │     Siguiente →   │  │ ← Navegación stepped
│  └───────────────────┘  │
│                         │
│  ┌───────────────────┐  │
│  │ ← Volver al chat  │  │ ← Escape alternativo
│  └───────────────────┘  │
│                         │
│ ╔═════════════════════╗ │
│ ║     [Teclado]       ║ │
│ ╚═════════════════════╝ │
└─────────────────────────┘

ESTADO 3 — POST-ENVÍO
┌─────────────────────────┐
│                         │
│  [Historial de chat]    │
│                         │
│  ┌───────────────────┐  │
│  │ ✅ Datos enviados │  │ ← Resumen inline
│  │ Pedro García      │  │    (permanente)
│  │ Reforma 123       │  │
│  └───────────────────┘  │
│                         │
│  Agente: ¡Gracias Pedro!│
│                         │
│ ┌─────────────────────┐ │
│ │ Escribe un mensaje  │ │ ← Chat input RESTAURADO
│ └─────────────────────┘ │
└─────────────────────────┘
```

### 9.3 Gestos soportados

| Gesto                    | Acción                    |
| ------------------------ | ------------------------- |
| Tap en botón/opción      | Seleccionar / activar     |
| Swipe down en form modal | Cerrar (con confirmación) |
| Tap en ✕                 | Cerrar / cancelar         |
| Mantener presionado      | Mostrar tooltip           |

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
| Profundidad > 4        | Media     | Aplanar                             |
| PML ilegible           | Alta      | Renderizar como texto plano         |

### 10.2 Errores de recursos

| Error                   | Acción                                |
| ----------------------- | ------------------------------------- |
| Imagen no carga         | Mostrar `alt` + placeholder           |
| Media no carga          | Mensaje + botón reintentar            |
| Data source no responde | Mostrar "Sin resultados" en reference |
| Form timeout            | "Conexión perdida" + reintentar       |

### 10.3 Errores de validación (usuario)

| Error                     | Acción                                 |
| ------------------------- | -------------------------------------- |
| Campo requerido vacío     | Borde rojo + `errorMessage`            |
| Validación falla          | Borde rojo + mensaje                   |
| Form incompleto (stepped) | No avanzar, error en paso actual       |
| after falla (time)        | Borde rojo + "Debe ser posterior a..." |

---

## 11. Versionado y extensibilidad

### 11.1 Versionado

```
PML v1.0 — Abril 2026
  - 15 componentes standalone
  - Form con lógica declarativa
  - Data sources, auto-populate, calculated fields
  - Layout responsivo
  - Components interactions (acceptation + visibility)
  - 3 modos de presentación (inline, stepped, modal)
```

### 11.2 Proceso de extensión

1. Componentes nuevos en versiones minor (v1.1, v1.2...)
2. Cambios de sintaxis solo en major (v2.0)
3. Renderers deben ignorar componentes desconocidos y renderizar como texto

### 11.3 Capacidades del renderer

```
[[SYSTEM]] capability=pml_version
```

Respuesta:

```
[[SYSTEM_RESPONSE]] pml_version=1.1&components=text,markdown,url,button,buttons,image,media,input,toggle,choice,opt,facts,divider,table,confirm,form
```

### 11.4 Candidatos para v1.1

| Componente     | Descripción                     | Prioridad |
| -------------- | ------------------------------- | --------- |
| `calendar`     | Selector de fecha visual (mes)  | Alta      |
| `map`          | Mapa con marcadores             | Alta      |
| `stepper`      | Indicador de pasos (1 de 5)     | Media     |
| `chart`        | Gráfica simple (bar, line, pie) | Media     |
| `notification` | Banner inline                   | Baja      |
| `avatar`       | Imagen de perfil circular       | Baja      |

---

## Apéndice A — Referencia rápida

```
┌────────────────┬───────────┬────────────────────────────────────────┐
│ Componente      │ Tipo      │ Uso principal                          │
├────────────────┼───────────┼────────────────────────────────────────┤
│ text            │ Standalone│ Texto estático con estilo              │
│ markdown        │ Standalone│ Texto con formato Markdown             │
│ url             │ Standalone│ Enlace clickeable                      │
│ button          │ Standalone│ Acción individual                      │
│ buttons         │ Standalone│ Fila de acciones (máx 5)               │
│ image           │ Standalone│ Imagen estática                        │
│ media           │ Standalone│ Video o audio                          │
│ input           │ Ambos     │ Campo de entrada (+ ref, auto, calc)   │
│ toggle          │ Ambos     │ Interruptor sí/no                      │
│ choice          │ Ambos     │ Selección múltiple                     │
│ opt             │ Choice only│ Opción individual de un choice        │
│ facts           │ Ambos     │ Tabla clave-valor                      │
│ divider         │ Ambos     │ Separador horizontal                   │
│ table           │ Standalone│ Tabla de datos                         │
│ confirm         │ Standalone│ Diálogo de confirmación                │
│ form            │ Contenedor│ Formulario con lógica declarativa      │
├────────────────┼───────────┼────────────────────────────────────────┤
│ source          │ Form only │ Definición de data source              │
│ header          │ Form only │ Cabecera del formulario                │
│ body            │ Form only │ Cuerpo del formulario                  │
│ row             │ Form only │ Fila de layout                         │
│ col             │ Form only │ Columna responsiva                     │
│ interactions    │ Form only │ Contenedor de reglas                   │
│ acceptation     │ Form only │ Regla de aceptación                    │
│ visibility      │ Form only │ Regla de visibilidad                   │
│ footer          │ Form only │ Pie del formulario                     │
└────────────────┴───────────┴────────────────────────────────────────┘
```

---

## Apéndice B — EBNF simplificado

```ebnf
message         = { text | markdown | component } ;

component       = self_closing | container | form ;

self_closing    = "[PML:" TYPE attr_list " /]" ;
container       = "[PML:" TYPE attr_list "]"
                  { child } "[/PML:" TYPE "]" ;

form            = "[PML:form" attr_list "]"
                  [ header ]
                  body
                  [ footer ]
                  "[/PML:form]" ;

header          = "[PML:header]" { child } "[/PML:header]" ;

body            = "[PML:body]"
                  { source | field | row | interactions | submit_button }
                  "[/PML:body]" ;

source          = "[PML:source" attr_list " /]" ;

field           = "[PML:input" attr_list " /]"
                | "[PML:toggle" attr_list " /]"
                | choice ;

choice          = "[PML:choice" attr_list "]" { opt } "[/PML:choice]" ;
opt             = "[PML:opt" attr_list " /]" ;

row             = "[PML:row]" { col } "[/PML:row]" ;
col             = "[PML:col" attr_list "]" field "[/PML:col]" ;

interactions    = "[PML:interactions]"
                  { acceptation | visibility }
                  "[/PML:interactions]" ;

acceptation     = "[PML:acceptation" attr_list " /]" ;
visibility      = "[PML:visibility" attr_list " /]" ;

submit_button   = "[PML:button" attr_list " subtype=\"submit\" /]" ;

footer          = "[PML:footer]" { child } "[/PML:footer]" ;

child           = text | markdown | self_closing | container ;

attr_list       = { ATTR_NAME "=" "\"" ATTR_VALUE "\"" } ;

TYPE            = "text" | "markdown" | "url" | "button" | "buttons"
                | "image" | "media" | "input" | "toggle" | "choice" | "opt"
                | "facts" | "divider" | "table" | "confirm" | "form" ;
```

---

_PML v1.1 — PEPPO Platform — Abril 2026_
