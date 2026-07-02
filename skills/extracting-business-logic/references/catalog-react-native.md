# Detection Catalog — React Native / Expo

Define cómo la Fase 1 (Discovery) detecta y cuenta cada sección en una app React
Native o Expo. Seleccionado cuando `package.json` tiene `react-native` o `expo` y
NO hay `angular.json`. El schema de salida es universal; la detección es por stack.

## Rules for using this catalog
- Los conteos por Glob son denominadores duros; los Grep son señales (activan
  sección, conteo aproximado).
- Activá una sección opcional solo si su patrón matchea ≥1.
- Los nombres de instancias salen del filename O del nombre de la interfaz/tipo/
  screen/slice cuando se detecta por contenido.
- En RN las librerías VARÍAN mucho entre proyectos. Este catálogo lista señales de
  las opciones comunes: en el discovery, detectá CUÁLES usa el proyecto (mirando
  `package.json` + imports) y aplicá solo esas. No asumas una sola.

## Detección del sub-stack (hacelo primero, condiciona el resto)
- **Navegación:** `expo-router` en package.json + carpeta `app/` con archivos de
  ruta → Expo Router (file-based). Si no, `@react-navigation/*` → React Navigation.
- **Estado:** `@reduxjs/toolkit` (slices) · `zustand` · `jotai` · `recoil` ·
  Context API (`createContext`). Detectá cuál domina.
- **Data-fetching:** `@tanstack/react-query` / `swr` / `axios` / `fetch`.
- **Validación:** `zod` · `yup` · resolvers de `react-hook-form` / `formik`.

## Entity detection note (React Native)
No hay decoradores `@Entity`. Las entidades de dominio viven como **tipos/interfaces
TypeScript** o **schemas** (zod/yup). Denominador candidato:
`**/*.types.ts`, `**/types/**/*.ts`, `**/models/**/*.ts`, `**/*.model.ts`, más
schemas `z.object(` / `yup.object(`. NO toda interfaz es entidad: props de
componentes, DTOs de respuesta y tipos de navegación (`RootStackParamList`) son
candidatos a FILTRAR en la Fase 2. Reportá el conteo crudo y filtrá con criterio.

## Business-rule sources note (React Native)
Las reglas de un frontend RN viven sobre todo en:
- **custom hooks** (`use*.ts`) — orquestación, gating condicional, estado derivado.
- **slices/stores** (Redux Toolkit `createSlice`, Zustand `create`) — reducers con
  lógica de dominio (no el set de estado plano).
- **thunks async** (`createAsyncThunk`, action creators con `redux-thunk`) — la
  orquestación y los gates condicionales de operaciones async suelen vivir acá.
- **validación de forms** — schemas zod/yup con reglas de dominio, O, si el proyecto
  NO usa una lib de validación, la validación manual en hooks/componentes (buscá
  chequeos condicionales que bloqueen submit / marquen error).
- **services / api layer** — lógica condicional antes/después de llamar al backend.
Pueden citarse componentes/screens en el `source` cuando la regla vive ahí. Igual
que siempre: solo constraints reales de dominio, no plumbing ni set de estado incondicional.

## Core sections (siempre activas)
| Sección        | Método | Patrón |
|----------------|--------|--------|
| overview       | —      | sintetizada por el modelo, nunca se cuenta |
| entities       | Glob   | `**/*.types.ts`, `**/types/**/*.ts`, `**/models/**/*.ts`, `**/*.model.ts` (candidatas — ver nota) |
| entities       | Grep   | `z.object(`, `yup.object(` (schemas como entidades/validación) |
| business-rules | Glob   | `**/hooks/**/*.ts`, `**/use*.ts`, `**/*.slice.ts`, `**/*.store.ts`, `**/*.schema.ts` |
| business-rules | Grep   | `createSlice(`, `createAsyncThunk(`, `create(` (zustand), `useMutation(`, resolvers de validación / chequeos manuales de form |
| user-flows     | Grep   | React Navigation: `createNativeStackNavigator`, `createBottomTabNavigator`, `<Stack.Screen`, `@react-navigation` |
| user-flows     | Glob   | `**/screens/**/*.tsx`, `**/*.screen.tsx`; Expo Router: `app/**/*.tsx` (rutas por archivo) |

## Optional sections (activar solo si matchea ≥1)
| Sección         | Método | Patrón |
|-----------------|--------|--------|
| roles           | Grep   | chequeos de rol/permiso en hooks/guards, `role`/`permission` en auth context, `casl` |
| integrations    | Grep   | `axios`/`fetch`, `@tanstack/react-query`/`swr`, SDKs en package.json (firebase, `@supabase`, stripe, sentry, amplitude), push (`expo-notifications`, `@react-native-firebase/messaging`), sesión/persistencia (`expo-secure-store`, `@react-native-async-storage`), device (`expo-camera`, `expo-image-picker`, `expo-av`, webview), módulos nativos |
| realtime        | Grep   | `socket.io-client`, `new WebSocket`, `supabase.channel(`, Firebase (`onSnapshot(`, `.on('value'`, RTDB/Firestore listeners), SSE/`EventSource` |
| state-machines  | Glob   | `**/*.machine.ts`, `xstate` en package.json |
| domain-events   | Grep   | `EventEmitter`, `DeviceEventEmitter`, `**/*.event.ts` |

## Secciones que rara vez aplican en RN
`scheduled-jobs` y `queues` casi nunca aplican en una app móvil. Activarlas SOLO si
hay evidencia real (tareas en background: `expo-task-manager`, `expo-background-fetch`,
`react-native-background-*`). Ante la duda, no las actives.
