# Análisis FODA — PML v1.5

> **Propósito.** Decidir si invertir tiempo y recursos en PML v1.5 como especificación pública para la renderización de UI de agentes conversacionales. El análisis es deliberadamente **crítico**: una decisión Go/No-Go robusta exige asumir lo peor de la propia idea y ver si sobrevive.
>
> **Fecha de corte del análisis de mercado.** Mayo 2026. Última gran reconfiguración del sector: convergencia OpenAI Apps SDK + MCP Apps (SEP-1865) + Google A2UI en Nov 2025.

---

## 0. Resumen ejecutivo

**Veredicto.** PML v1.5 **no debe lanzarse como especificación abierta neutral** que compita de frente con Adaptive Cards, A2UI, MCP Apps u OpenAI Apps SDK. La tesis técnica (densidad de tokens, fricción mínima para el LLM) **es real y está validada por literatura académica reciente**, pero la ventana de "estándar abierto LLM-nativo" se cerró en Q4 2025 con la convergencia OpenAI–Anthropic–MCP-UI–Google. Llegar a esa pelea en 2026 con un único contribuidor, sin renderers, sin diseñador visual, sin SDKs y con la spec en español es entrar a un mercado consolidado con una eslinga.

**Hay dos rutas que sí justifican continuar:**

1. **PML como infraestructura interna de PEPPO** — formato propietario del ChatApp de PEPPO, donde la densidad de tokens se traduce en ahorro operativo directo y la disciplina mobile-first sirve al producto sin necesidad de ganar guerras de estándares.
2. **PML como capa de autoría que transpila a Adaptive Cards / A2UI / MCP Apps HTML** — el LLM emite PML barato; un transpiler determinístico produce el formato que el cliente entiende. Aprovecha la única ventaja defendible (densidad) sin pelear contra ecosistemas que ya ganaron la batalla del renderer.

**Lo que no se debe hacer.** Publicar PML v1.5 como spec independiente, escribir un renderer "de referencia" en React y esperar a que la comunidad la adopte. Eso es una trampa de proyecto-personal-de-fin-de-semana: alto costo de mantenimiento, retorno cercano a cero, y un ecosistema que ya tiene tres alternativas mejor financiadas.

---

## 1. Contexto: qué es PML v1.5 y qué problema dice resolver

PML (PEPPO Markup Language) v1.5 es un lenguaje de marcado basado en brackets (`[PML:input id="x" /]`) diseñado para que LLMs emitan componentes UI interactivos dentro de un chat. Cinco versiones en dos meses (v1.1 abril → v1.5 mayo 2026), última adición: componente `form` con submit único multi-campo.

**La tesis nuclear del proyecto:**

> JSON es verboso, los LLMs cometen errores generándolo, y los formatos UI actuales (Adaptive Cards, JSON Flow de WhatsApp, blocks de Slack) gastan demasiados tokens. Un lenguaje bracket-tag homogéneo, sin anidación tipo objeto, ahorra tokens y reduce errores de generación.

**Esta tesis es plausible** y, como veremos en §2, está parcialmente validada por la literatura académica reciente. La pregunta no es si la tesis es correcta, sino si **traducir esa tesis en un estándar abierto, en mayo de 2026, contra los tres incumbentes consolidados, es una buena asignación de recursos.**

---

## 2. Panorama del mercado (mayo 2026)

### 2.1 Los tres paradigmas dominantes

| Paradigma | Representantes | Pros | Contras |
|---|---|---|---|
| **HTML-in-iframe** | OpenAI Apps SDK, MCP Apps (SEP-1865), MCP-UI | Rica interactividad, full HTML/CSS/JS, ya en producción en ChatGPT (800M usuarios) | HTML es aún más verboso que JSON — peor en tokens — y vendor-locked al runtime del host |
| **JSON declarativo** | Google A2UI, Microsoft Adaptive Cards 1.6, Slack Block Kit, WhatsApp Flows | Portátil entre renderers, ecosistemas maduros, builders WYSIWYG | Verboso (form de 5 campos ≈ 600-800 tokens en Adaptive Cards), overhead estructural |
| **Streaming de componentes React** | Vercel AI SDK 3 (Generative UI), CopilotKit + AG-UI | Integración nativa Next.js, tipado fuerte | Acoplado a React, AG-UI fragmenta, AI SDK RSC se reportó pausado en finales de 2025 |

### 2.2 Hallazgo académico clave

El paper **UIFORMER (arXiv 2512.13438)** documenta que **la representación de UI consume entre el 80% y 99% del costo total de tokens de un agente**. Este es el dato más importante de todo este análisis: **valida** la tesis de PML, pero también explica por qué Google, OpenAI y Anthropic están corriendo a resolverlo. La presa es grande y los grandes la están cazando.

### 2.3 Tabla comparativa enfocada PML vs los relevantes

| Tecnología | Lanzamiento | Sintaxis | Densidad de tokens | Forms multi-campo | Ecosistema | Apertura | Vendor lock |
|---|---|---|---|---|---|---|---|
| **PML v1.5** | May 2026 | Bracket DSL | **★★★★★** (~3-5× más denso que JSON) | Sí (`form` MVP) | **0** renderers, **0** SDKs, **0** designers | Spec libre | Ninguno |
| **OpenAI Apps SDK + MCP Apps** | Oct/Nov 2025 | MCP + HTML iframe | ★★ (HTML verboso) | Sí (HTML forms) | Pilots: Canva, Spotify, Zillow, Figma, Booking. Submissions abiertas dic 2025 | Spec abierta (sobre MCP) | Runtime de ChatGPT |
| **Google A2UI** | Nov 2025 (v0.9 en 2026) | JSON declarativo | ★★★ | Sí, nativo multi-campo | Producción en Opal, Gemini Enterprise, Flutter GenUI SDK | Spec abierta | Ninguno (intencional) |
| **Microsoft Adaptive Cards 1.6** | 2017 (v1.6 actual) | JSON | ★★ (verboso, overhead estructural ~50 tokens) | Sí (Input.* + validación) | 10+ SDKs oficiales, Designer maduro, integrado en Teams/Outlook/Copilot Studio | MIT, pero ecosistema MS | Práctico en MS |
| **WhatsApp Flows** | 2023 | JSON con screens + routing | ★★★ | **Sí, best-in-class** multi-pantalla | 2B+ usuarios WA, Flow Builder | Spec Meta | 100% Meta |
| **Slack Block Kit** | 2019 | JSON blocks | ★★ | Sí (modales) | Block Kit Builder, decenas de miles de apps | Spec Slack | 100% Slack |
| **CopilotKit + AG-UI** | 2024 (v1.50 dic 2025) | JSON event stream tipado + hooks React | ★★★ | Sí | Apache 2.0, adoptado por Oracle Agent Spec | Abierto | Acoplado React |
| **Microsoft POML** | Ago 2025 | XML-like tags | N/A (apunta a prompt, no UI) | No aplica | MIT | Abierto | — |

**Lectura:** **PML lidera densidad de tokens** y queda **último en todo lo demás**. Esa es la apuesta entera del proyecto en una sola frase.

---

## 3. FORTALEZAS (internas, reales y verificables)

### F1. Densidad de tokens medible y validada
Una `[PML:input id="email" type="text" required="true" /]` usa ~14 tokens. El equivalente Adaptive Cards (`{"type":"Input.Text","id":"email","isRequired":true}` envuelto en `body` y root) usa ~40-50 tokens contando overhead. **Factor 3-4× consistente**, multiplicado por cada turno del agente, multiplicado por cada usuario, multiplicado por meses. El paper UIFORMER pone número al ahorro: la UI absorbe 80-99% del gasto de tokens del agente. **Esta fortaleza es la única razón seria para que PML exista.**

### F2. Diseño LLM-first genuino
La sintaxis es regular, predecible, sin estructuras anidadas tipo objeto, sin comas que olvidar, sin escapes de comillas dentro de JSON, sin pares clave-valor que el modelo confunda. Los modelos actuales generan JSON cada vez mejor (function calling, structured outputs, constrained sampling) pero **PML elimina por construcción categorías enteras de fallos**, no las mitiga. Diferencia conceptual real.

### F3. Modelo de zonas y fases mobile-first muy bien pensado
La separación `shared_history` / `task_narrative` / `task_controls` con tres fases (`first_rendering` / `live_capture` / `final_rendering`) es **mejor diseño UX que lo que ofrecen Block Kit o Adaptive Cards de forma nativa**, que retrofitearon mobile. Reconoce explícitamente el problema del chat input "sagrado", la exclusividad de modales, y la promoción inmutable a historial compartido. Esto es trabajo de diseño honesto.

### F4. Soporte multi-usuario incorporado
Las zonas privadas al disparador (`task_*` efímeras) y el snapshot inmutable a `shared_history` anticipan conversaciones grupales sin acoplar audiencia al PML mismo (queda en el envelope del runner). **Casi ningún competidor maneja esto.** WhatsApp Flows asume 1:1, Adaptive Cards no tiene noción de audiencia, A2UI tampoco.

### F5. Consistencia interna y rigor de la spec
- EBNF completo en Apéndice B.
- Reglas léxicas estrictas (case, comillas, IDs, escapado).
- URL encoding documentado exhaustivamente con tabla de casos.
- Reglas de composición con tabla explícita de anidación permitida.
- Migración v1.3→v1.4→v1.5 documentada con tabla y comportamiento de compatibilidad hacia atrás.
- Manejo de errores tabulado por severidad (§11).

Como artefacto técnico, **es una spec de calidad profesional**. Mejor escrita que muchas specs corporativas que vi a lo largo de los años.

### F6. Compatibilidad hacia atrás disciplinada
v1.5 es puramente aditivo. Un parser v1.4 lee v1.5 sin romperse (renderiza `form` como texto plano). El bracket multi-campo `[[INTERACTION]]` ya se había definido en v1.4 §7.3 como contrato preparatorio. **Esto refleja madurez de diseño**: planificar la extensión antes de necesitarla.

### F7. Separación de concerns UI-Logic vs Agente
El principio `UILogic-first` (lógica UI vive en el frontend, no en el agente) es correcto y reduce el prompt del agente. Adaptive Cards también lo hace; OpenAI Apps SDK lo rompe (la app HTML lleva su propia lógica).

### F8. Costo de generación cero
Spec en Markdown, autor único, sin dependencias, sin infra. **El costo hundido es bajo**, lo que limita el daño si se decide pausar.

---

## 4. OPORTUNIDADES (externas, accionables)

### O1. Costo de tokens validado como dolor real
El paper UIFORMER y el discurso público de OpenAI sobre eficiencia de agentes confirman que **el sector siente el dolor que PML aborda**. Hay narrativa de mercado que PML puede ocupar — "el formato más barato de UI para agentes". Es un slogan defendible.

### O2. Hueco específico: capa de autoría sobre los renderers existentes
**Nadie está construyendo un DSL denso que compile a Adaptive Cards / A2UI / MCP Apps HTML.** El LLM emite PML (barato), un transpiler lo expande al formato que el host entiende (HTML, JSON). Esta es **la posición más defendible que tiene PML**: no compite con los renderers, los alimenta. Convierte el peor enemigo (A2UI) en target de compilación.

### O3. Vacío en LATAM y mercados hispanohablantes
WhatsApp domina LATAM. Las herramientas en inglés siguen siendo la norma. Si PEPPO se posiciona como plataforma de agentes para comercio conversacional en español, **PML como formato interno acelera el time-to-market** del producto.

### O4. MCP gana, MCP Apps tiene fricción
MCP base es estándar de facto para tools/resources. MCP Apps añade HTML-in-iframe pero la verbosidad de HTML es brutal. **Si PML se ofrece como "el lenguaje de autoría LLM para producir MCP Apps"**, se acopla al ganador sin competirle.

### O5. Cansancio de DSL nuevos = oportunidad de bracket familiar
Los desarrolladores tienen fatiga de YAML/JSON/TOML. La sintaxis bracket es familiar (XML, JSX) sin la complejidad de XML real. **Apuesta UX**: el desarrollador que ve `[PML:input ...]` lo entiende en 5 segundos.

### O6. Comercio conversacional crece en RCS y WhatsApp
RCS Business Messaging creció +500% YoY en revenue durante 2025. WhatsApp Flows está saturado de developers no-técnicos. **Hay demanda de herramientas que generen interactividad sin tocar JSON de Meta**.

### O7. Valor educativo y de portafolio
Aun si PML no se adopta, **la spec es un artefacto excelente para portafolio del autor**: demuestra capacidad de diseño de lenguajes, rigor formal (EBNF), pensamiento UX mobile, y disciplina de versionado. Valor profesional aunque el producto no vuele.

---

## 5. DEBILIDADES (internas, severas)

### D1. Bus factor = 1
Un solo contribuyente (`pedro-aaron`), 5 commits totales, todos del mismo autor. Sin co-autores, sin revisores externos, sin tests de aceptación. **Cualquier estándar serio necesita al menos 2-3 voces independientes en el diseño** para no calcificar errores. Adaptive Cards tiene un working group, MCP tiene Anthropic + OpenAI + cientos de contributors, A2UI tiene a Google.

### D2. Ecosistema = 0
- **0 renderers** funcionales (ni web, ni Android, ni iOS, ni desktop).
- **0 parsers** publicados (TypeScript, Python, Go, Kotlin, Swift…).
- **0 designer/builder visual**.
- **0 linter / validator**.
- **0 ejemplos en producción** verificables.
- **0 SDKs** para emisión desde el lado del agente.

Adaptive Cards trae 10+ SDKs oficiales + Designer + Builder + galería. WhatsApp tiene Flow Builder. Slack tiene Block Kit Builder. **PML compite con un PDF.**

### D3. Spec exclusivamente en español
La comunidad global de IA opera en inglés. **Una spec que no tiene README en inglés es invisible** para 95% de los potenciales adoptantes, contributors y bloggers que podrían amplificarla. Esto es un veto silencioso al alcance internacional.

### D4. Naming débil
- **PEPPO** es un acrónimo opaco que no comunica nada al primer contacto.
- **PML** colisiona con varios nombres existentes (Polish Markup Language, Process Modeling Language, varios productos enterprise). Búsqueda SEO está perdida desde el día uno.
- Sin marca, sin logo, sin sitio web, sin demo en vivo. La spec es el único contacto.

### D5. Iteración demasiado rápida para algo que pide adopción
**5 versiones en 2 meses** (v1.1 abril → v1.5 mayo 2026). Para un agente o renderer externo, esto es destabilizante: cada mes hay que decidir si trackear o quedarse atrás. Specs maduras (Adaptive Cards, MCP) liberan minor releases con cadencia trimestral o semestral, no semanal.

### D6. Sintaxis "id reservado" del form es smell de diseño
La regla "los hijos del form no pueden tener `id="id"` ni `id="value"`" (§6.5) **revela una fricción que un diseño más limpio habría evitado**. Un prefijo `field-*` o un namespace separado en el bracket multi-campo hubiera eliminado la colisión sin imponer una palabra reservada arbitraria. Detalle menor, pero síntoma.

### D7. Sin streaming explícito
Los LLMs actuales generan token a token y los renderers modernos pintan progresivamente. **La spec no aborda cómo renderizar un componente PML parcialmente emitido**. ¿Esperamos al cierre del bracket? ¿Hay placeholder? Adaptive Cards y MCP Apps tienen el mismo problema, pero PML, que se vende como "LLM-friendly", debería tener una respuesta.

### D8. Sin accesibilidad ni i18n
- Cero mención de ARIA, screen readers, navegación por teclado, contraste.
- Cero soporte de data binding o templating — todos los labels viajan dentro del PML, no se puede separar contenido de estructura.
- Esto **descalifica a PML para sectores regulados** (gobierno, salud, educación, banca grande) donde a11y es ley.

### D9. Sin modelo de seguridad
URL upload, location, contact, file. Cero mención de CSP, sanitización de inputs reenviados como `value=""` (riesgo de XSS si el renderer no filtra), validación de URLs (riesgo de SSRF en `image url=""`), tamaños máximos por defecto, ni mitigaciones contra prompt injection vía `[[INTERACTION value="..."]]`. **Una spec UI sin sección de seguridad no es production-ready.**

### D10. Componentes que faltan respecto a la competencia
Inventario contra lo que el mercado considera mínimo:

| Componente esperado | PML v1.5 | Adaptive Cards | A2UI | WhatsApp Flows |
|---|---|---|---|---|
| Carousel / cards horizontales | **No** | Sí | Sí | Sí |
| Card de producto (e-commerce) | **No** | Sí | Sí | Sí |
| Charts / gráficos | **No** | Sí (Action.ShowCard ext) | Sí | No |
| Map embebido (no solo picker) | **No** | Sí | Sí | Sí |
| Pago / payment intent | **No** | Sí (Action.Payment) | Sí | Sí |
| Carrusel de imágenes | **No** | Sí | Sí | Sí |
| Reactions / quick feedback | **No** | Sí (Action.Submit con data) | Sí | Sí |
| Tabs / pestañas | **No** | Sí (v1.5+) | Sí | No |
| Progress / loading state | **No** | Parcial | Sí | Sí |
| Lista con scroll virtualizado | **No** | Sí | Sí | Sí (NavigationList) |

**Brecha funcional grande.** Construir cada uno es semanas de diseño + renderer + tests.

### D11. Sin métricas reales del ahorro de tokens
PML afirma ser más denso pero **no hay benchmark publicado**. ¿Cuántos tokens ahorra un form de 5 campos vs el equivalente en Adaptive Cards o A2UI? Sin esa cifra concreta, la fortaleza F1 es retórica, no evidencia.

### D12. Ausencia de runtime de referencia abre la puerta a divergencia
Diferentes implementaciones del renderer interpretarán las áreas grises (anidaciones tolerantes, errores de URL encoding, manejo de `error=""` tras submit) de forma incompatible. **Sin un test suite oficial, la spec se balcaniza al primer intento de adopción seria.**

---

## 6. AMENAZAS (externas, hostiles)

### A1. La ventana del estándar abierto se cerró en Nov 2025
**OpenAI Apps SDK (oct 2025), MCP Apps SEP-1865 (nov 2025) y Google A2UI v0.9 (nov 2025)** convergieron en 30 días. Los proveedores con distribución (OpenAI 800M usuarios, Google Gemini, Anthropic Claude) ya escogieron sus formatos. **PML llega después de que la mesa quedó puesta.**

### A2. Google A2UI es básicamente el competidor terminal
A2UI es JSON declarativo, framework-agnóstico, open spec, con renderers nativos en Flutter/Angular/Lit, **producción real en Opal y Gemini Enterprise, respaldado por Google**. La única ventaja defendible de PML contra A2UI es densidad de tokens. **Es una pelea de un round contra Mike Tyson.**

### A3. Los LLMs generan JSON cada vez mejor
La premisa "los LLMs se equivocan generando JSON" pierde fuerza cada trimestre. Structured outputs, constrained sampling, function calling, JSON mode, schema-guided generation: los frontier models de 2026 producen JSON válido el 99.x% de las veces. **La fricción que PML elimina se está erosionando por software ajeno.**

### A4. Vendor channels obligan a sus formatos
Si PEPPO quiere desplegar en WhatsApp, debe emitir JSON Flow. En Teams, Adaptive Cards. En ChatGPT, Apps SDK. En iMessage Business, su formato Apple. **PML termina siendo, en el mejor caso, un formato intermedio que se transpila a otra cosa** en cada canal — exactamente lo que ya proponemos en O2 como pivot.

### A5. MCP gana la guerra de tools, MCP Apps gana la de UI
Si MCP Apps (SEP-1865) consolida HTML-in-iframe como estándar abierto bendecido por Anthropic + OpenAI, **el espacio "UI markup abierto" ya tiene incumbente con dos gigantes detrás**. PML como spec independiente compite contra eso sin distribución.

### A6. Fatiga de "yet another DSL"
La comunidad developer está saturada de DSLs: MJML, JSX, MDX, Pug, HAML, Slim, Marko, YAML-based DSLs, etc. Cada nuevo DSL paga un costo de educación enorme. **PML no presenta un diferencial 10× que justifique el costo cognitivo de aprenderlo** sobre alternativas conocidas.

### A7. Falta de fondeo y runway
Las specs abiertas que prosperan tienen patrocinio (Adaptive Cards-Microsoft, A2UI-Google, MCP-Anthropic, AG-UI-CopilotKit) **o** comunidad self-sustaining (CommonMark). PML no tiene ninguno. Sin runway, en 12 meses el proyecto será un repo archivado más.

### A8. Riesgo regulatorio
GDPR, CCPA, LGPD, HIPAA, PCI-DSS aplican a forms que capturan datos personales o de salud. **Sin tooling, certificaciones ni guías de cumplimiento, los integradores serios la descartan.** Adaptive Cards tiene guías de Microsoft para Teams en sectores regulados; PML no tiene nada.

---

## 7. Matriz de cruce FODA → estrategias

| | Oportunidades | Amenazas |
|---|---|---|
| **Fortalezas** | **Estrategias FO (ofensivas)**<br>• Publicar benchmark token-density PML vs A2UI/Adaptive Cards (F1+O1) → claim defendible.<br>• Posicionar PML como **capa de autoría que transpila a MCP Apps / A2UI / Adaptive Cards** (F1+F2+O2+O4).<br>• Lanzar PML como formato interno PEPPO en español para LATAM (F3+F4+O3+O6). | **Estrategias FA (defensivas)**<br>• Atarse a MCP/MCP Apps en lugar de competirles (F1+A5): "PML emite MCP Apps".<br>• Convertir F3 (zonas mobile) en feature exclusiva no replicable trivialmente por A2UI/Adaptive Cards (F3+A2).<br>• Aprovechar el bajo costo hundido (F8) para pivotar rápido en cuanto haya señales (F8+A1). |
| **Debilidades** | **Estrategias DO (adaptativas)**<br>• Traducir spec a inglés antes que nada (D3+O1) — costo bajo, desbloqueo grande.<br>• Construir UN renderer de referencia (Android nativo, por sinergia con PEPPO/Peppo) en lugar de prometer N (D2+O3).<br>• Cerrar el componente set faltante (D10) priorizando solo lo que PEPPO necesite, no lo que el mercado pida. | **Estrategias DA (de supervivencia)**<br>• **No** lanzar como spec abierta neutral (D1+D2+A1+A2+A5). Esta es la conclusión central.<br>• Si se publica, hacerlo etiquetado explícitamente "formato interno PEPPO" para evitar la expectativa de estándar (D1+A6).<br>• Congelar la spec en v1.5 hasta que haya un renderer funcional; deja de iterar specs sin código (D5+A7). |

---

## 8. Escenarios estratégicos

### Escenario A — "Spec abierta competitiva" (NO RECOMENDADO)
PML se publica como estándar abierto, se construye un renderer de referencia, se promociona en HackerNews / dev.to / blog corporativos.

**Costo esperado.** 12-18 meses persona para alcanzar paridad mínima con A2UI: spec en inglés, parser TS + Python, renderer Web + Android, designer básico, suite de tests, sitio web, docs, ejemplos, marketing.

**Probabilidad de éxito** (=ser adoptado por al menos 100 proyectos independientes en 24 meses): **<3%**.

**Razones del fracaso esperado.** A1, A2, A5, A7. La mesa ya está puesta. Sin distribución ni patrocinio, no hay forma de competir con A2UI/MCP Apps/Adaptive Cards.

**Veredicto.** No.

### Escenario B — "Capa de autoría que transpila" (RECOMENDADO con condiciones)
PML se posiciona como **lenguaje compacto de autoría**: el LLM emite PML; un transpiler determinístico (Python/TS) produce A2UI JSON, Adaptive Cards JSON, MCP Apps HTML, o WhatsApp Flow JSON según el canal de destino.

**Costo esperado.** 3-4 meses persona para MVP: spec congelada en v1.5, transpiler PML → A2UI + Adaptive Cards (los dos más portables), benchmark publicado de token density, demo en vivo.

**Probabilidad de éxito** (= adoptado por equipos que sienten dolor de tokens): **15-25%**. Más alto si el benchmark UIFORMER se cita en el material de marketing.

**Razones por las que esto puede funcionar.** Aprovecha F1, F2, O2, A5 a tu favor (en lugar de pelearles). Posiciona PML como complemento, no sustituto. Las decisiones del mercado quedan validadas como targets, no como competidores.

**Veredicto.** Sí, si y solo si hay compromiso de 3-4 meses con métricas duras.

### Escenario C — "Formato interno PEPPO" (RECOMENDADO sin condiciones)
PML se queda como formato propietario del ChatApp de PEPPO. El backend de PEPPO emite PML, los clientes Android/iOS/Web de PEPPO lo renderizan. La spec se publica como documentación interna abierta (para developers que integran PEPPO) pero **no se promueve como estándar**.

**Costo esperado.** Ya hundido en la spec. Solo añade renderer en cada cliente PEPPO (esto se haría de todos modos con cualquier formato).

**Probabilidad de éxito** (= reducir costo de tokens de PEPPO en producción): **>80%**, asumiendo que PEPPO mismo llegue a producción.

**Razones por las que esto funciona.** Convierte F1 en ahorro operativo directo y medible (línea en P&L). Las debilidades D1-D5 dejan de importar (no compite con nadie). El éxito de PML queda atado al éxito de PEPPO, que es la apuesta real del autor de todos modos.

**Veredicto.** Sí, por default.

### Combinación recomendada: C ahora, B opcional después
- **Inmediato (0-3 meses).** Escenario C. Congelar PML v1.5, construir el renderer en el ChatApp PEPPO, generar el primer dato real de ahorro de tokens en producción.
- **Después de 6 meses con datos reales (escenario C corriendo).** Re-evaluar Escenario B con evidencia. Si el ahorro medido es 30%+ contra una baseline JSON, hay un caso para sacar el transpiler como producto open-source separado.
- **Nunca.** Escenario A puro.

---

## 9. Criterios de decisión y umbrales

**Continuar invirtiendo en PML SOLO si todos estos se cumplen:**

1. **PEPPO mismo está vivo y tiene roadmap de 6+ meses.** Si PEPPO no llega a producción, PML no tiene sentido.
2. **Se acepta que PML es infraestructura interna, no estándar abierto.** Si la motivación es "que la comunidad lo adopte", abandonar.
3. **Hay capacidad de construir el primer renderer (Android o Web) en ≤8 semanas.** Si no, la spec es papel mojado.
4. **Hay benchmark numérico de ahorro de tokens vs A2UI / Adaptive Cards antes del primer renderer.** Sin métrica, no hay defensa.

**Abandonar PML SI:**

- Pasan 6 meses sin un renderer funcional.
- El benchmark de tokens muestra ahorro <20% vs A2UI (umbral mínimo para que el costo de mantener el DSL se justifique).
- Aparece un competidor con la misma propuesta y distribución (ej. Google publica un "A2UI compact mode" o Anthropic propone "MCP Apps Tag Syntax").
- El equipo PEPPO crece y los nuevos miembros se quejan de tener que aprender PML (señal de fricción interna).

**Independientemente del escenario, hacer YA:**

1. **Traducir README.md a inglés.** Costo: 1 día. Beneficio: desbloquea cualquier conversación con la comunidad internacional.
2. **Publicar un benchmark de tokens.** Tomar un formulario representativo (registro, checkout) y medir tokens en PML, A2UI, Adaptive Cards, WhatsApp Flow. Pública la tabla.
3. **Añadir sección "Comparación con prior art"** a la spec, citando explícitamente Adaptive Cards, A2UI, MCP Apps. La ausencia actual de esta sección hace ver la spec como ingenua o aislada.
4. **Congelar la cadencia de versiones.** No más v1.6 hasta tener un renderer en producción. La velocidad de spec sin código encubre falta de validación.

---

## 10. Veredicto final

**PML v1.5 es una pieza de diseño técnico bien ejecutada que llega tarde a un mercado que se acaba de consolidar.** La tesis central (densidad de tokens) está validada por literatura académica y es defendible, pero la ventana competitiva para un estándar abierto se cerró seis meses antes de que esta spec existiera.

**Decisión recomendada: PIVOT a uso interno (Escenario C), con la puerta abierta a Escenario B si los datos lo justifican en 6 meses.** Bajo ninguna circunstancia perseguir el Escenario A (spec abierta neutral).

El trabajo invertido en PML **no se desperdicia** bajo el Escenario C: cada principio bien diseñado (zonas mobile, lifecycle, multi-usuario, URL encoding disciplinado, EBNF) se aplica al producto PEPPO directamente. El error sería confundir "tengo una spec bonita" con "tengo un estándar que el mundo va a adoptar". Son dos juegos distintos y el segundo ya tiene tres jugadores con presupuesto.

> **Una métrica que cierra el caso.** Tres convergencias en Q4 2025 (OpenAI Apps SDK, MCP Apps, Google A2UI), un paper académico (UIFORMER) que cuantifica el problema y **cero specs bracket-DSL** entre los ganadores. El mercado eligió: HTML-in-iframe o JSON declarativo, no DSLs nuevos. PML como spec abierta nace en oposición a esa elección sin la fuerza para revertirla.
>
> Como infraestructura interna de PEPPO, en cambio, **PML está en pelea consigo mismo: solo necesita ser mejor que JSON suelto** — y lo es. Esa es la pelea que vale la pena dar.

---

## Apéndice — Resumen de una línea por punto

**Fortalezas:** densidad de tokens (F1), diseño LLM-first (F2), modelo de zonas mobile (F3), multi-usuario (F4), spec rigurosa (F5), backward compat (F6), separación UI/agente (F7), costo hundido bajo (F8).

**Oportunidades:** dolor validado (O1), nicho de autoría sobre renderers (O2), LATAM hispano (O3), montar sobre MCP (O4), fatiga de DSL crea apertura paradójica (O5), boom RCS/WhatsApp (O6), portafolio (O7).

**Debilidades:** bus factor=1 (D1), ecosistema=0 (D2), spec en español (D3), naming débil (D4), versionado hiperactivo (D5), id reservado es smell (D6), sin streaming (D7), sin a11y/i18n (D8), sin modelo seguridad (D9), componentes faltantes (D10), sin métricas (D11), sin test suite (D12).

**Amenazas:** ventana cerrada Nov 2025 (A1), Google A2UI (A2), LLMs cada vez mejores en JSON (A3), vendor channels obligan formatos (A4), MCP Apps ya es estándar abierto bendecido (A5), fatiga de DSL (A6), sin fondeo (A7), riesgo regulatorio (A8).
