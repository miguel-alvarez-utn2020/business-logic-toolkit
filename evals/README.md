# Eval suite — business-logic-toolkit

Tests de **comportamiento** de los skills/agentes del toolkit. No es un eval de
modelo: mide si los prompts producen el resultado correcto de forma **consistente**
(el modelo es estocástico → cada escenario se corre N veces y se mide pass rate).

## Por qué evals y no solo pruebas manuales
Las preguntas y decisiones las genera el modelo → hay variabilidad entre corridas.
"Anduvo una vez" no alcanza, sobre todo porque el target es gente con poca
experiencia. El eval da: (a) confianza medible, (b) regresión cada vez que tocamos
un prompt.

## El problema de la interactividad (y cómo lo resolvemos)
El intake de `/fix` usa `AskUserQuestion` (necesita un humano). Para poder evaluar
sin humano, los evals automatizados son **read-only de razonamiento**:

- Se le pasa al agente el set COMPLETO de respuestas del QA por adelantado (el
  "guion" del reporte) y se le pide ejecutar el RAZONAMIENTO del intake (localizar,
  aplicar el gate de reproducción, decidir, notar convenciones) y EMITIR la decisión
  + el Fix-Brief o el intake "no-reproducible" — **sin** abrir preguntas y **sin
  aplicar** cambios al repo.
- Esto vuelve cada eval read-only → corre en paralelo sin colisiones ni necesidad de
  sembrar/revertir bugs. La localización se testea igual: "dado el síntoma, ¿a qué
  archivo/función apunta y qué propone?" funciona sobre el código limpio.
- **Fuera del suite automatizado** (se chequean a mano): (a) la UX de la entrevista
  en vivo —calidad de las preguntas—; (b) que `fix-dev` APLIQUE el diff
  correctamente —muta archivos, ya validado en pruebas manuales—.

## Estructura
```
evals/
├── README.md
├── fix/                 # escenarios de /fix (intake + fix-dev)
├── business-logic/      # extracción de lógica de negocio
└── conventions/         # extracción de convenciones
```

## Formato de cada escenario
Cada archivo `.md` define:
- **id / qué prueba:** la dimensión bajo test.
- **setup:** estado del repo objetivo (bug a sembrar, si aplica) + app objetivo.
- **input:** comando + guion de respuestas del QA (para los de lógica).
- **expected:** criterios de pass, separados en:
  - **objetivos** (verificables sin juicio: archivo terminó en X, git diff = Y, NO
    se aplicó cambio, tocó solo el archivo Z).
  - **de juicio** (LLM-judge o revisión: mapeó la BR correcta, siguió la convención
    NGXS, las preguntas quedaron observables).
- **runs:** cuántas veces correr (default 3) y umbral de pass (ej. 3/3 para los
  críticos como el gate de reproducción; 2/3 para los de juicio).

## Cómo se corre
- **Semi-manual:** se ejecuta cada escenario en contexto limpio y se califica contra
  los criterios. Pocos runs, barato.
- **Workflow (opt-in):** fan-out de escenarios × N corridas + agentes-juez que
  califican. Más cobertura, más tokens. Requiere pedirlo explícitamente.

## Limpieza
Los evals automatizados son read-only → no hay nada que limpiar. Si algún chequeo
manual siembra un bug, revertí con `git checkout -- <archivo>` al terminar.
