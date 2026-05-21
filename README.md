# email-template-create

Skill local para **crear un nuevo template de email** con react-email para Niñas PRO. Hace una mini-entrevista (audiencia, copias, propósito, CTA) y entrega un `*Email.tsx` listo, registrado en `src/templates.ts` y con su CSV de muestra.

## Qué hace

1. Carga el brand pack desde la skill [`brand-extract`](../brand-extract/) (colores, fuentes, primitivas).
2. Pregunta al usuario 4 cosas:
   - **¿Para quién es el correo?** (alumnas / postulantes / mentoras / apoderadas)
   - **¿Quién más recibe copia?** (cc / bcc dinámicos o fijos)
   - **¿Por qué es importante? ¿Qué se quiere comunicar?** (texto libre — define copy y urgencia)
   - **¿Cuál es la acción principal (CTA)?** (inscribirse / confirmar / unirse / activar / informativo)
3. Infiere parámetros (nombre, tag, color de la window, componentes a usar, subject, preview).
4. Genera 3 archivos: `emails/<Nombre>Email.tsx`, entrada en `src/templates.ts`, `csv/<nombre>.csv`.
5. Verifica que el schema Zod, el `sample` y el CSV estén alineados.

## Cuándo se usa

- "crear un correo nuevo / template / plantilla"
- "necesito un email para X"
- "agregar una plantilla de recordatorio de pago / invitación / lo que sea"

## Cómo invocarla

```
/email-template-create
```

O en lenguaje natural: "crea un nuevo template", "necesito un correo para X".

Si el usuario describe el correo en una sola frase (audiencia + propósito + CTA), la skill **no re-pregunta** lo que ya está claro — solo completa lo que falta.

## Estructura

```
.agents/email-template-create/
├── README.md     ← este archivo (qué es, cómo se usa)
└── SKILL.md      ← instrucciones detalladas para Claude
```

## Relación con otras skills

- [`brand-extract`](../brand-extract/) → fuente de verdad de tokens. Esta skill la invoca como Paso 0.

## Requisitos

- Working directory en la raíz del proyecto.
- Existen: `emails/components/`, `emails/theme.ts`, `src/templates.ts`, `csv/`.
- React Email + Zod ya instalados (`package.json`).

## Salida

Tres archivos sincronizados:

| Archivo | Contenido |
|---|---|
| `emails/<Nombre>Email.tsx` | Componente react-email con `Layout` + `Window` + primitivas + CTA |
| `src/templates.ts` (editado) | Import + schema Zod + entry en `TEMPLATES` con `subject`, `description`, `sample` |
| `csv/<nombre>.csv` | Header `email,cc,bcc,<props>` + 1-2 filas de ejemplo |

Después: `npm run dev` → `http://localhost:3000` → el template aparece automáticamente en la galería.

## Anti-patrones (rechazar)

- Crear un template sin pasar antes por [`brand-extract`](../brand-extract/).
- Hex literales en JSX, Tailwind dentro de `emails/`, fuentes no cargadas en `Layout`.
- Olvidar registrar en `src/templates.ts` (queda invisible para la UI/API).
- Subject como string fijo en vez de función `(p) => string`.
- CSV con headers que no coinciden 1:1 con los keys del schema.
# skill-template-email-create
