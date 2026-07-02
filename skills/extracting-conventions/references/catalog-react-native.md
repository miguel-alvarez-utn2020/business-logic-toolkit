# Detection Catalog — Convenciones React Native / Expo

Define cómo detectar el patrón dominante de cada categoría en una app React Native /
Expo, y las fuentes oficiales a consultar. Seleccionado cuando `package.json` tiene
`react-native` o `expo`. La detección es por stack; el esquema de salida es universal.

## Detección del sub-stack (hacelo primero)
- **Navegación:** `expo-router` + carpeta `app/` con rutas → Expo Router (file-based).
  Si no, `@react-navigation/*` → React Navigation.
- **Estado:** `@reduxjs/toolkit` (slices) · `redux` "clásico" (reducers + `redux-thunk`)
  · `zustand` · `jotai` · Context. OJO: proyectos suelen tener Redux clásico + RTK a
  medio migrar conviviendo — detectá y reportá ambos.
- **Data:** `@tanstack/react-query`/`swr` · `axios`/`fetch`.
- **Validación:** `zod`/`yup` + `react-hook-form`/`formik`, o **manual** (sin lib).
- **Estilos:** `StyleSheet.create` · `nativewind` · `tamagui` · `styled-components`.

## Fuentes oficiales (traer la versión detectada)
- **React:** https://react.dev/reference/react (hooks, componentes)
- **React Native:** https://reactnative.dev/docs/getting-started
- **Expo / Expo Router:** https://docs.expo.dev , https://docs.expo.dev/router/introduction/
- **Redux Toolkit (estilo oficial de Redux):** https://redux-toolkit.js.org/usage/usage-guide , https://redux.js.org/style-guide/
- **React Navigation:** https://reactnavigation.org/docs/getting-started
> Versión-sensible: mirá la versión de cada lib en `package.json` y traé la guía
> acorde. Redux moderno = RTK + slices (el estilo oficial desaconseja el "Redux
> clásico" a mano); marcá esa distinción en el delta.

## Categorías y señales de detección

| Categoría | Qué muestrear / Grep | Qué determina el patrón |
|-----------|----------------------|-------------------------|
| Arquitectura / carpetas | estructura (`app/`, `components/`, `hooks/`, `redux/`, `context/`, `utils/`) | por-feature vs por-tipo; ubicación de `app/` (Expo Router) |
| Navegación | `app/**` (Expo Router) vs `createNativeStackNavigator`/`createBottomTabNavigator` | file-based vs navigators; layouts `_layout.tsx` |
| Estado global | `createSlice(` (RTK) vs `createStore`/`combineReducers`/`redux-thunk` (clásico) vs `create(` (zustand) vs `createContext` | store dominante; RTK vs clásico; convivencia/migración a medias |
| Datos / API | cliente `axios.create(` central vs `fetch` suelto; inyección de token/headers; `react-query`/`swr` | ¿hay cliente central? ¿dónde se inyecta auth? (deuda si se duplica) |
| Persistencia | `@react-native-async-storage`, `expo-secure-store`, `react-native-mmkv` | qué se persiste y con qué (token en secure-store, etc.) |
| Componentes | `**/*.tsx` funcionales, tipos de props, `memo`, default vs named export | anatomía, naming, tipado de props |
| Hooks | `**/hooks/**`, `**/use*.ts` | patrón de custom hooks, qué encapsulan |
| Forms / validación | `react-hook-form`/`formik` + `zod`/`yup`, o validación **manual** en handlers | reactivo con schema vs manual; duplicación de reglas |
| UI / estilos | `StyleSheet.create`, `nativewind`, `tamagui`, theme/tokens, `styles/` | sistema de estilos; ubicaciones conviviendo |
| Integraciones nativas | módulos `expo-*` (camera, notifications, image-picker, av), `firebase`, webview | qué capacidades nativas se usan y cómo se envuelven |
| Tooling | `eslint-config-expo`/`.eslintrc`, `prettier`, `jest-expo`, EAS (`eas.json`), scripts | lint (¿configurado y operante?), testing, build/EAS, commits |

## Notas
- **Patrón dominante, no inventario.** Si hay convivencia (Redux clásico + RTK,
  dos ubicaciones de estilos), reportá el dominante y anotá la otra como excepción/deuda.
- **Versión manda en lo oficial.** No recomiendes APIs que la versión instalada no
  tenga; y si el proyecto usa el estilo viejo (Redux a mano) mientras lo oficial es
  RTK, es `delta = desviación/deuda`: documentalo, no lo corrijas.
- **Inconsistencias reales = oro:** doble inyección de headers de auth, ESLint
  instalado pero sin config operante, validación duplicada — anotalas en su categoría.
