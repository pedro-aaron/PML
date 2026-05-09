# Propuesta: Unificar `[[INTERACTION]]` y `[[CONFIRM]]` en un solo keyword

## Problema actual

En la especificación v1.1, existen dos keywords distintos para capturar interacciones del usuario:

- `[[INTERACTION]]` — para acciones normales (Button, Input, Toggle, Choice, URL, etc.)
- `[[CONFIRM]]` — para diálogos de confirmación (Confirm component)

**Ambos tienen el mismo propósito**: capturar una acción del usuario y enviarla al agente. La distinción actual introduce complejidad innecesaria.

### Ejemplos actuales

**Interacción normal:**
```
[[INTERACTION]] id=confirmar value=confirmed
```

**Confirmación:**
```
[[CONFIRM]] id=eliminar-cuenta value=confirmed
[[CONFIRM]] id=eliminar-cuenta value=cancelled
```

## Propuesta

Unificar ambos en un solo keyword `[[INTERACTION]]` con un atributo opcional `type` para distinguir el contexto:

```
[[INTERACTION]] id=confirmar value=confirmed
[[INTERACTION]] id=eliminar-cuenta type=confirmation value=confirmed
[[INTERACTION]] id=eliminar-cuenta type=confirmation value=cancelled
```

## Beneficios

1. **Simplicidad**: Un único keyword en lugar de dos
2. **Consistencia**: Todas las interacciones del usuario usan el mismo patrón
3. **Extensibilidad**: Fácil agregar nuevos tipos en el futuro sin introducir nuevos keywords
4. **Menor carga cognitiva**: El renderizador y los agentes solo necesitan parsear un patrón
5. **Token efficiency**: Menos keywords diferentes = más eficiente en tokens

## Detalles técnicos

### Sintaxis propuesta

```
[[INTERACTION]] id=<identificador> [type=<tipo>] value=<valor>
```

### Valores de `type` (opcional, default: `normal`)

| Valor | Descripción | Respuestas posibles |
|-------|-------------|-------------------|
| `normal` (default) | Acción estándar | 1 (el value del componente) |
| `confirmation` | Diálogo de confirmación | 2 (confirmed o cancelled) |

### Ejemplos completos

#### Antes (actual)
```
[PML:button id="guardar" label="Guardar" value="save" style="primary" /]
→ [[INTERACTION]] id=guardar value=save

[PML:confirm id="eliminar" title="¿Eliminar?" confirmLabel="Sí" cancelLabel="No" /]
→ [[CONFIRM]] id=eliminar value=confirmed
→ [[CONFIRM]] id=eliminar value=cancelled
```

#### Después (propuesta)
```
[PML:button id="guardar" label="Guardar" value="save" style="primary" /]
→ [[INTERACTION]] id=guardar value=save

[PML:confirm id="eliminar" title="¿Eliminar?" confirmLabel="Sí" cancelLabel="No" /]
→ [[INTERACTION]] id=eliminar type=confirmation value=confirmed
→ [[INTERACTION]] id=eliminar type=confirmation value=cancelled
```

## Compatibilidad

### Con agentes
- Los agentes pueden parsear `type` de forma opcional
- Si `type` no está presente, asumir `type=normal`
- Fácil mantener compatibilidad hacia atrás

### Con renderers
- Solo cambio en cómo se generan los keywords
- La lógica de manejo permanece igual

## Impacto en la especificación

### Secciones a actualizar

1. **Sección 2.5** — Encoding de interacciones
   - Actualizar ejemplos para usar `type=`

2. **Sección 4.4** (Button)
   - Cambiar ejemplos de `[[INTERACTION]]`

3. **Sección 4.8** (Input)
   - Cambiar ejemplos de `[[INTERACTION]]`

4. **Sección 4.10** (Choice)
   - Cambiar ejemplos de `[[INTERACTION]]`

5. **Sección 4.14** (Confirm)
   - **Cambiar de `[[CONFIRM]]` a `[[INTERACTION]]` con `type=confirmation`**

6. **Sección 6** (Modelo de interacción)
   - Consolidar en un único patrón

## Alternativas consideradas

### 1. Mantener ambos keywords
- ❌ Complejidad innecesaria
- ❌ Difícil de mantener

### 2. Usar un atributo `subtype` en lugar de `type`
- ✓ Posible, pero `type` es más estándar en especificaciones
- ✓ `subtype` ya se usa en elementos PML (e.g., button subtype="submit"), así que hay precedente
- 🤔 Decisión de diseño: preferencia por consistencia

## Preguntas abiertas

1. ¿Debería `type=confirmation` requerir siempre `id` único?
2. ¿Hay otros tipos de interacción futuros que justifiquen este cambio?
3. ¿El renderer debería validar que `value` sea `confirmed|cancelled` solo si `type=confirmation`?

## Conclusión

Esta propuesta **simplifica la especificación** sin perder capacidad expresiva, mejora la **consistencia** del lenguaje y facilita futuras extensiones.

---

**Propuesto por:** usuario pedro-aaron (basado en feedback)  
**Fecha:** 2026-05-09  
**Versión afectada:** v1.2 (propuesta)
