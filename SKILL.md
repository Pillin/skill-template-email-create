---
name: email-template-create
description: Crea un nuevo template de email con react-email para Niñas PRO. Conduce una mini-entrevista (audiencia, copias, propósito, CTA), aplica el brand pack de [[brand-extract]] y entrega un `*Email.tsx` listo + registro en `src/templates.ts` + CSV de muestra. Úsala cuando el usuario diga "crear un correo / template / plantilla nueva".
---

# email-template-create — Niñas PRO

Genera un template de email **react-email** completo y registrado, a partir de 4–5 preguntas guiadas. Es la skill complementaria de [[brand-extract]] (que entrega los tokens de marca).

## Stack obligatorio

- `@react-email/components` (Layout, Window, Button, etc. ya están envueltos en `emails/components/`).
- `colors` + `fontStack`/`monoStack` desde `emails/theme.ts`. **Nunca** hex hardcodeado ni Tailwind dentro de `emails/`.
- `zod` para el schema en `src/templates.ts`.
- Tipos `Props` exportados + valores default en la firma para que `template-sample` funcione.

## Paso 0 · Cargar el brand pack

Antes de preguntar nada, invocar / leer la skill [[brand-extract]] y mantener su output en contexto.
Si los tokens cambiaron desde la última vez, **re-leélos** (no asumir cache mental).

## Paso 1 · Entrevista guiada

Usar `AskUserQuestion` con estas 4 preguntas (en este orden). Si el usuario ya dio alguna respuesta en su prompt inicial, **saltar esa pregunta** — no preguntar lo que ya sabes.

**P1 — ¿Para quién es el correo?**
Header chip: `Audiencia`
Opciones sugeridas (single-select):
- "Alumnas inscritas" — destinatario principal son las niñas/alumnas ya matriculadas
- "Postulantes / leads" — gente interesada pero aún no inscrita
- "Mentoras / staff" — equipo interno
- "Apoderadas/es" — familias / tutores legales

**P2 — ¿Quién más va a recibir copia?**
Header chip: `Copias`
Multi-select:
- "Nadie más" — solo el destinatario
- "Mentora del curso" — agregar como `cc` dinámico
- "Coordinación Niñas PRO" — `cc` fijo (hola@ninaspro.cl)
- "Apoderada/o" — `cc` con campo extra en CSV

**P3 — ¿Por qué es importante este correo? ¿Qué se quiere comunicar?**
Texto libre (no usar opciones — esto define copy y CTA).
Pedir al usuario: propósito, urgencia, fecha clave, acción esperada del receptor.
Si la respuesta inicial del usuario ya cubre esto, no re-preguntar.

**P4 — ¿Cuál es la acción principal (CTA)?**
Header chip: `CTA`
Opciones:
- "Inscribirse / postular" — link a postulación
- "Confirmar asistencia" — link RSVP
- "Unirse a sesión" — link Meet / dirección presencial → usar `AccessBlock`
- "Activar cuenta / ingresar a plataforma" — link a `app.ninaspro.cl`
- "Sólo informativo" — sin CTA, cierre con email de contacto

## Paso 2 · Inferir parámetros

Con las 4 respuestas, derivar:

| Decisión | Regla |
|---|---|
| Nombre del template | PascalCase del propósito → `RecordatorioPagoEmail`, `InvitacionApoderadasEmail`, etc. |
| `tag` del HeroBlock | `[ NN ] <Categoría>` — siguiente número libre mirando los templates existentes |
| `titleBarColor` del Window | `amarillo` por defecto · `rosaFuerte` si es anuncio · `turquesa` si es invitación · `cream` si es comunicado neutro |
| `filename` del Window | `/<seccion>/<slug>.ext` — extensión semántica (`.txt`, `.exe`, `.ics`, `.md`) |
| Componentes a usar | Inscripción/anuncio → `InfoRow` table · Sesión/charla → `AccessBlock` · Bienvenida/informativo → solo `HeroBlock` + `Body` |
| Props del componente | Siempre `nombre: string`. Agregar fechas/horas/links según P3+P4 |
| Subject | Corto, 1 emoji al inicio + dato dinámico clave |
| Preview | < 90 chars, complementa el subject (no lo repite) |

## Paso 3 · Generar archivos

**A) `emails/<Nombre>Email.tsx`** — copiar el esqueleto base (abajo) y rellenar.

```tsx
import { Section, Text } from "@react-email/components";
import { Layout } from "./components/organisms/Layout";
import { Window } from "./components/organisms/Window";
import { Button } from "./components/atoms/Button";
import { HeroBlock } from "./components/molecules/HeroBlock";
import { Body } from "./components/atoms/Text";
import { colors } from "./theme";

export type <Nombre>Props = {
  nombre: string;
  // ...props específicas
};

export const <Nombre>Email: React.FC<<Nombre>Props> = ({
  nombre = "Ana",
  // defaults para preview
}) => {
  return (
    <Layout preview={`<preview text>`}>
      <Window filename="/<seccion>/<slug>.ext" titleBarColor={colors.<color>}>
        <HeroBlock tag="[ NN ] <Categoría>" title={`Hola, ${nombre} ✨`} intro="..." />

        {/* Bloque(s) según P3+P4 — InfoRow / AccessBlock / Section custom */}

        <Section style={{ textAlign: "center", margin: "8px 0 8px" }}>
          <Button href={enlace}>CTA →</Button>
        </Section>

        <Text style={{ fontSize: 13, color: colors.gris, margin: "20px 0 0", textAlign: "center" }}>
          Si tienes dudas, responde este correo o escríbenos a hola@ninaspro.cl
        </Text>
      </Window>
    </Layout>
  );
};

export default <Nombre>Email;
```

**B) Registrar en `src/templates.ts`:**

1. Importar `<Nombre>Email` y `<Nombre>Props` arriba.
2. Crear `<nombre>Schema = z.object({ ...baseEnvelope, ...props })` con validación Zod.
3. Agregar entry en `TEMPLATES`:
   ```ts
   <nombre>: {
     component: <Nombre>Email,
     schema: <nombre>Schema,
     subject: (p) => `<emoji> ${p.<campoClave>}`,
     description: "<una línea>",
     sample: { email: "ana@example.com", nombre: "Ana", /* ... */ },
   },
   ```

**C) CSV de muestra en `csv/<nombre>.csv`:**
Header con `email,cc,bcc,<cada prop del schema>` + 1–2 filas de ejemplo.

## Paso 4 · Verificación final

Antes de entregar, **leer** los archivos generados y confirmar:

- [ ] No hay hex literales en el `.tsx` nuevo (todo via `colors.xxx`).
- [ ] Schema Zod **incluye `...baseEnvelope`** y los `.url()` / `.email()` cuando corresponda.
- [ ] `sample` tiene **todos** los campos del schema (si falta uno, el preview rompe).
- [ ] `subject` es función `(p) => string`, no string fijo.
- [ ] El nombre del template (key en `TEMPLATES`) es kebab-case-sin-guiones (`recordatoriopago`) o camelCase (`recordatorioPago`) — seguir el patrón de los existentes (`bienvenida`, `anuncio`, `recordatorio`, `charla`).
- [ ] CSV header coincide 1:1 con los keys del schema.

Reportar al usuario:
1. Ruta del nuevo `*Email.tsx`.
2. Resumen del schema (props requeridas vs opcionales).
3. Comando para previsualizar: `npm run dev` → `http://localhost:3000` → tab del nuevo template.

## Reglas duras

- **Una pregunta a la vez**: usar `AskUserQuestion` con las 4 preguntas en un solo bloque cuando se pueda agrupar, pero respetar el flujo (P3 es texto libre, va aparte si hace falta).
- **No re-preguntar** lo que el usuario ya dijo en su mensaje inicial.
- **No crear componentes nuevos** dentro de `emails/components/` salvo que el usuario lo pida explícitamente — esta skill compone con lo existente.
- **No tocar `Footer`** — es global y se inyecta desde `Layout`.
- **No usar Tailwind** dentro de `emails/`. Estilos inline siempre.
- **No saltarse el registro en `src/templates.ts`** — sin eso el template no aparece en la galería ni en la API.

## Output al usuario

Al terminar, entregar:

1. Confirmación de los 3 archivos creados/editados (`emails/<Nombre>Email.tsx`, `src/templates.ts`, `csv/<nombre>.csv`).
2. Snippet del JSX final del template para revisión rápida.
3. Próximos pasos: `npm run dev` y verificar el preview.

Ver `README.md` para el resumen de uso de esta skill.
