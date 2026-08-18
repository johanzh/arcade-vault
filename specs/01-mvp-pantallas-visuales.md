# SPEC 01 — MVP visual de Arcade Vault (pantallas sin lógica de juego)

> **Status:** Approved
> **Depends on:** —
> **Date:** 2026-08-18
> **Objective:** Implementar en Next.js/Tailwind las 5 pantallas visuales de `references/templates/` (Biblioteca, Detalle, Reproductor, Salón de la Fama, Auth) como rutas reales del App Router, sin implementar ningún juego real.

---

## Por qué existe este spec

El proyecto es hoy el scaffold sin modificar de `create-next-app`. `references/templates/` contiene un prototipo funcional en React+Babel standalone (sin build, CSS a mano) que define la identidad visual retro/neón/CRT de Arcade Vault. Este spec traslada esa identidad al stack real del proyecto (Next.js 16 App Router, TypeScript, Tailwind v4) usando rutas reales en vez del router por hash del prototipo, y sin la simulación de "partida" que el prototipo usa como relleno.

---

## Scope

**In:**

- 5 pantallas como rutas reales del App Router:
  - `/` — Biblioteca (grid de juegos, buscador, filtro por categoría).
  - `/juegos/[id]` — Detalle del juego (info, tabla de mejores puntuaciones, botón jugar).
  - `/juegos/[id]/jugar` — Reproductor (HUD, pantalla tipo CRT, modal de fin de juego) como mockup estático interactivo, sin lógica de juego real.
  - `/salon-de-la-fama` — Salón de la Fama (podio + tabla por juego, con tabs).
  - `/auth` — Iniciar sesión / Crear cuenta (formulario con tabs, sin backend).
- Layout compartido (`app/layout.tsx`) con `Nav` (con menú móvil hamburguesa) y footer, igual que `app.jsx`/`nav.jsx` del prototipo.
- Diseño visual (colores neón, tipografías pixel/mono, efectos CRT/scanlines/glow/flicker) reutilizando las clases CSS ya portadas en `app/globals.css` (commit `5c657fe`) desde `styles.css` del template — ver nota en Decisiones.
- Datos mock de juegos, jugadores y puntuaciones (`GAMES`, `CATS`, `PLAYERS`, `seededScores`) portados a TypeScript, reutilizando el mismo contenido y algoritmo pseudoaleatorio determinista de `data.jsx`.
- Interactividad puramente visual dentro de cada pantalla (buscador y chips de categoría en Biblioteca; tabs en Salón de la Fama y Auth; pausa/fin/reinicio y modal en Reproductor; menú móvil en Nav).
- Fuentes Google (`Press Start 2P`, `Courier Prime`, `JetBrains Mono`) cargadas con `next/font/google`.
- Estado de "ruta activa" en `Nav` calculado a partir de la ruta real (`usePathname`), replicando el resaltado de enlaces del prototipo.

**Out of scope (for future specs):**

- Cualquier juego jugable real (Bloque Buster, Caída, Serpentina, etc.). La pantalla `/juegos/[id]/jugar` es un mockup, no un motor de juego.
- Autenticación real (backend, validación de credenciales, hashing, cookies de sesión, OAuth con Google/GitHub). Los botones sociales del prototipo se muestran solo como decoración no funcional.
- Persistencia de sesión de usuario entre pantallas o recargas. Cada pantalla se renderiza siempre en su estado de invitado; enviar el formulario de `/auth` no cambia el estado de `Nav` ni del Salón de la Fama.
- Persistencia de puntuaciones jugadas (no hay `localStorage` de `av_scores` ni `av_user`; el botón "Guardar puntuación" en el Reproductor solo cambia el estado visual local de esa página).
- Contador de créditos funcional (el "CRÉDITOS · 03" del Nav es texto estático decorativo).
- Backend/API routes, base de datos, o cualquier persistencia server-side.
- Internacionalización (todo el contenido queda en español, igual que el prototipo).

---

## Data model

```ts
// lib/games.ts
export type Game = {
  id: string;
  title: string;
  short: string;
  long: string;
  cat: "ARCADE" | "PUZZLE" | "SHOOTER" | "VERSUS";
  cover: string; // clase/id usado para elegir el fondo decorativo de portada
  color: "cyan" | "magenta" | "green" | "yellow";
  best: number;
  plays: string;
};

export const GAMES: Game[]; // los mismos 8 juegos de data.jsx
export const CATS: string[]; // ["TODOS", "ARCADE", "PUZZLE", "SHOOTER", "VERSUS"]
export const PLAYERS: string[]; // 18 nombres de jugador ficticios

export type ScoreRow = { rank: number; name: string; score: number; date: string };
export function seededScores(seed: number, count?: number): ScoreRow[];
```

`seededScores` conserva el mismo generador pseudoaleatorio lineal (`s = (s * 9301 + 49297) % 233280`) para que las tablas de puntuaciones sean deterministas por `id`/categoría, igual que en el prototipo.

Este spec no introduce persistencia (no hay `localStorage`, cookies ni base de datos): todo el "estado" es o bien datos mock estáticos, o bien estado de React local a cada página (`useState`), que se pierde al navegar o recargar.

---

## Implementation plan

1. Crear `lib/games.ts` con `GAMES`, `CATS`, `PLAYERS` y `seededScores`, tipado en TypeScript, portado desde `references/templates/data.jsx`.
2. Definir tokens visuales en `app/globals.css` (colores neón, fondo con ruido/grid, variables `--cyan`, `--magenta`, `--yellow`, `--green`, `--gold`, `--line`, `--ink*`) y cargar las 3 fuentes vía `next/font/google` en `app/layout.tsx`.
3. Construir `components/nav.tsx` (Client Component) con logo, enlaces `Biblioteca`/`Salón de la Fama`, botón de auth, contador de créditos estático y menú móvil con overlay, usando `usePathname` para resaltar la ruta activa. Integrarlo junto con el footer en `app/layout.tsx`.
4. Construir `components/game-card.tsx` (con el efecto de tilt al mover el mouse) y la página `app/page.tsx` (Biblioteca): hero, buscador, chips de categoría y grid de tarjetas filtradas.
5. Construir `app/juegos/[id]/page.tsx` (Detalle): portada, tags, descripción, stats, botones de acción (enlazando a `/juegos/[id]/jugar`) y tabla lateral de mejores puntuaciones vía `seededScores`. `id` inexistente → `notFound()`.
6. Construir `app/juegos/[id]/jugar/page.tsx` (Reproductor, Client Component): HUD con valores fijos de ejemplo, pantalla tipo CRT con arte placeholder estático, botones Pausa/Fin/Salir funcionales sobre estado local, y modal de fin de juego con campo de iniciales + botón "Guardar puntuación" que solo cambia el estado visual (toast "guardado"), sin persistir nada.
7. Construir `app/salon-de-la-fama/page.tsx` (Salón, Client Component): tabs por juego, podio (top 3) y tabla completa vía `seededScores`, con el bloque "tu mejor marca" omitido (no hay sesión que mostrar).
8. Construir `app/auth/page.tsx` (Auth, Client Component): tabs Iniciar sesión / Crear cuenta, campos de formulario, botones sociales decorativos y botón "Jugar como invitado" que navega a `/`; el submit no persiste ni redirige con sesión activa.
9. Repasar responsive y estados hover/focus de las 5 pantallas en mobile y desktop; ejecutar `npm run lint` y corregir avisos.

---

## Acceptance criteria

- [ ] Las 5 rutas (`/`, `/juegos/[id]`, `/juegos/[id]/jugar`, `/salon-de-la-fama`, `/auth`) cargan sin errores en consola.
- [ ] `Nav` aparece en las 5 rutas y resalta el enlace activo correspondiente.
- [ ] En `/`, escribir en el buscador o pulsar una categoría filtra el grid de juegos sin recargar la página.
- [ ] En `/juegos/[id]`, el botón "Jugar ahora" navega a `/juegos/[id]/jugar`.
- [ ] Visitar `/juegos/id-inexistente` muestra la página 404 de Next.js.
- [ ] En `/juegos/[id]/jugar`, el botón "Pausa" alterna a "Reanudar" y muestra el overlay de pausa; el botón "Fin" abre el modal de fin de juego.
- [ ] En el modal de fin de juego, pulsar "Guardar puntuación" cambia la UI a un mensaje de confirmación, y recargar la página no conserva ese estado.
- [ ] En `/salon-de-la-fama`, cambiar de tab cambia el juego mostrado en el podio y la tabla.
- [ ] En `/auth`, cambiar entre las tabs "Iniciar sesión" y "Crear cuenta" muestra/oculta el campo de correo electrónico; enviar el formulario no cambia el contenido de `Nav`.
- [ ] `npm run lint` pasa sin errores.
- [ ] `npm run build` completa sin errores de TypeScript.

---

## Decisions

- **Sí:** rutas reales del App Router en vez del router por hash del prototipo. Da URLs compartibles y usa el enrutado nativo de Next.js en vez de reinventar uno.
- **No:** mantener el hash-router del prototipo. Contradice las convenciones del framework que ya provee el proyecto.
- **Revertida (durante implementación):** se había decidido reimplementar el diseño con utilidades de Tailwind v4 en vez de portar `styles.css` tal cual. Al iniciar la implementación se encontró que el commit `5c657fe "global styles"` (previo a este spec) ya había portado `styles.css` casi textualmente a `app/globals.css` (clases `.av-nav`, `.btn`, `.chip`, `.card`, `.crt`, `.modal`, etc.) y ya cargaba las 3 fuentes en `app/layout.tsx`. Se decidió adoptar ese trabajo ya existente en vez de descartarlo y reescribirlo con utilidades, para no tirar trabajo ya hecho. Las 5 pantallas se construyen reutilizando esas clases custom.
- **Sí:** el Reproductor es un mockup interactivo (los botones cambian estado visual) pero sin puntuación real ni persistencia, ya que el spec explícitamente no incluye ningún juego.
- **No:** simular la partida con puntuación subiendo sola y "enemigos" animados como en el prototipo — se descartó por decisión explícita del usuario, para no insinuar un juego que no existe.
- **Sí:** sesión de usuario siempre en estado invitado, sin Context ni `localStorage`. Simplifica la implementación (cada página es independiente) a cambio de no poder demostrar visualmente el estado "logueado" del Nav/Salón — aceptado como parte del alcance de este MVP.
- **No:** Context de auth en memoria para reflejar el login durante la visita. Se descartó por decisión explícita del usuario a favor de simplicidad.

---

## What is **not** in this spec

- Cualquier juego jugable (los 8 juegos listados en `GAMES` son solo tarjetas/portadas, ninguno se implementa).
- Autenticación real, backend, o persistencia de sesión/puntuaciones.
- Contador de créditos funcional.

Cada uno de estos, si se implementa, va en su propio spec.
