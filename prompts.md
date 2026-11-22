# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
analiza el módulo de cuentas y acceso, backend y frontend
```

**Qué salió:** leyó backend y frontend del módulo y probó la API con curl; devolvió un análisis en el chat, todavía sin spec.

---

## Prompt 2

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
HAZLO
```

**Qué salió:** escribió docs/spec-viva/LA.md con la spec (formato obligatorio) y un borrador de las tres listas de la parte B, a partir del análisis del prompt 1.

---

## Prompt 3

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
PERO ESTO CUMPLIRIA LO DE ESTE MODULO?
```

**Qué salió:** revisó el archivo contra la checklist del enunciado y listó lo que falta para cumplirlo.

---

## Prompt 4

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
1. VALIDA TU. 2 HAZLO
```

**Qué salió:** el agente comprobó los requisitos contra el código y con peticiones reales a la API, corrigió la spec y reescribió las listas de la parte B. La verificación la hizo el agente, no yo; así consta en la lista 1.

---

## Prompt 5

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
SUBELO
```

**Qué salió:** commit y push de la rama spec-viva-LA a mi fork.
