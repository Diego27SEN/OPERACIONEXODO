# Bitácora de lectura del proyecto — OPERACIONEXODO

> Documento de reconocimiento técnico del proyecto de Unreal Engine 5.7 `OPERACIONEXODO`.
> Fecha de lectura: 22 de noviembre de 2026.
> Alcance: estructura de carpetas, inventario de assets, configuración, mapas, sistemas de gameplay, materiales y estado del control de versiones.

---

## 1. Resumen ejecutivo

`OPERACIONEXODO` es un prototipo de videojuego de disparos en tercera persona (TPS) con temática de zombis y supervivencia, construido sobre la plantilla *Third Person BP Game Template* de Unreal Engine.

El proyecto **no tiene código C++ propio**: toda la lógica de gameplay está implementada con **Blueprints**. Existe una carpeta `Intermediate\Source` con el stub del módulo `OPERACIONEXODO` (generado por la plantilla), pero no hay carpeta `Source\` real en la raíz ni módulos de juego compilados aparte del plugin `UnrealOpenCode`.

Los pilares que se pueden identificar en el contenido son:

- Un **personaje jugador SWAT** en tercera persona con cámara orbital, apuntado (aim), arma equipable y disparo por *line trace*.
- Un **HUD de mira (crosshair)**.
- **Zombis** de varios tipos (común, furioso, ácido, jefe) con IA basada en *Pawn Sensing*, Behavior Trees y Blackboards.
- **Trampas de láser** y **triggers de daño/curación** implementados como Blueprints.
- Escenarios de **cañón/arena** y **laboratorios secretos**, con World Partition.

---

## 2. Ficha técnica

| Dato | Valor |
|---|---|
| Nombre del proyecto | `OPERACIONEXODO` |
| Archivo de proyecto | `OPERACIONEXODO.uproject` |
| Versión de motor | Unreal Engine **5.7** (`EngineAssociation: "5.7"`) |
| Nombre interno (`DefaultGame.ini`) | `Third Person BP Game Template` |
| GameName (`DefaultEngine.ini`) | `OPERACIONEXODO` |
| Módulo de juego | `OPERACIONEXODO` (solo stub, sin código de gameplay) |
| Lenguaje de gameplay | Blueprints (100 %) |
| Mapa de inicio | `/Game/ThirdPerson/Lvl_ThirdPerson` |
| GameMode por defecto | `/Game/ThirdPerson/Blueprints/BP_ThirdPersonGameMode` |
| RHI objetivo | DX12, Shader Model 6 |
| Render | Lumen, Ray Tracing, Substrate, Virtual Shadow Maps, Mesh Distance Fields |
| Audio | 48 000 Hz, 1024 frames, 32 canales máx. |
| Control de versiones | Git + Git LFS (`.gitattributes` para `.uasset` / `.umap`) |

### Plugins declarados en el `.uproject`

| Plugin | Estado | Notas |
|---|---|---|
| `ModelingToolsEditorMode` | Habilitado (solo Editor) | Herramientas de modelado dentro del editor. |
| `GameplayStateTree` | Habilitado | State Tree para gameplay (no se observó uso en assets). |

Además, en `Plugins\UnrealOpenCode\` vive un plugin de terceros (`UnrealOpenCode`) que expone un puente MCP para que un agente/IDE pueda inspeccionar el editor. **No es contenido del juego**, es una herramienta de desarrollo.

### Redirecciones heredadas de la plantilla

En `DefaultEngine.ini`:

```ini
+ActiveGameNameRedirects=(OldGameName="TP_ThirdPersonBP",NewGameName="/Script/OPERACIONEXODO")
+ActiveGameNameRedirects=(OldGameName="/Script/TP_ThirdPersonBP",NewGameName="/Script/OPERACIONEXODO")
```

Esto confirma que el proyecto nació de la plantilla `TP_ThirdPersonBP` y fue renombrado.

---

## 3. Estructura general del repositorio

```
OPERACIONEXODO/
├── OPERACIONEXODO.uproject
├── README.md                  (solo "# OPERACIONEXODO")
├── Config/
│   ├── DefaultEngine.ini
│   ├── DefaultGame.ini
│   ├── DefaultInput.ini
│   └── ...
├── Content/                   (todo el contenido del juego)
├── Plugins/UnrealOpenCode/    (herramienta de desarrollo, no gameplay)
├── Binaries/Win64/            (stub compilado del módulo del proyecto)
├── Intermediate/              (caché del editor; incluye el stub de Source)
├── Saved/                     (autosaves, logs, cache)
├── .gitignore
└── .gitattributes
```

> Nota: el `.gitignore` es el de una plantilla genérica (incluye reglas de Xcode/Fortran/Visual Studio 2015) y **no** ignora `Content/`, por lo que los assets binarios se versionan (vía LFS).

---

## 4. Inventario de `Content/`

### 4.1 Conteo global

| Métrica | Valor |
|---|---|
| Archivos totales en `Content/` (excluyendo externos) | 347 |
| `.uasset` | 336 |
| `.umap` | 11 |
| Peso total aproximado | 373,8 MB |
| Actores externos (`Content/__ExternalActors__`) | 104 archivos (~1,1 MB) |
| Objetos externos (`Content/__ExternalObjects__`) | 6 archivos |

### 4.2 Peso y cantidad por carpeta principal

| Carpeta | Archivos | Assets/Maps | Tamaño |
|---|---:|---:|---:|
| `Personaje` | 99 | 99 | 210,5 MB |
| `Characters` | 128 | 128 | 125,3 MB |
| `Models` | 6 | 6 | 16,0 MB |
| `Texturas` | 9 | 9 | 10,9 MB |
| `Objetos` | 5 | 5 | 3,5 MB |
| `LevelPrototyping` | 29 | 29 | 2,7 MB |
| `Animations` | 5 | 5 | 2,5 MB |
| `Scenes` | 21 | 21 | 0,7 MB |
| `ThirdPerson` | 8 | 8 | 0,5 MB |
| `IA` | 11 | 11 | 0,4 MB |
| `rayo_Laser` | 6 | 6 | 0,4 MB |
| `Input` | 13 | 13 | 0,3 MB |
| `Materials` | 5 | 5 | 0,1 MB |
| `Joints` | 1 | 1 | ~0 MB |
| `IA_Zombie_Grande` | 1 | 1 | ~0 MB |
| `Collections` | 0 | 0 | — |
| `Developers` | 0 | 0 | — |

### 4.3 Tipos de asset detectados (según cadena de clase en el binario)

| Tipo | Cantidad aproximada |
|---|---:|
| `AnimSequence` | 133 |
| `SkeletalMesh` | 99 |
| `StaticMesh` | 29 |
| `Texture2D` | 23 |
| `MaterialInstanceConstant` | 11 |
| `InputAction` | 8 |
| `MaterialParameterCollection` | 7 |
| `Level` (`.umap`) | 7 (no redirectores) |
| `BehaviorTree` | 7 |
| `WidgetBlueprint` | 4 |
| `Blueprint` | 3 (+ blueprints cuyo marcador no se detectó por el escaneo) |
| `AnimMontage` | 3 |
| `BlackboardData` | 2 |
| `Material` | 2 (+ instancias) |
| `InputMappingContext` | 2 |
| `AnimBlueprint` | 1 |
| `MaterialFunction` | 1 |

> El conteo de tipos es aproximado: se obtuvo buscando marcadores de clase en los binarios, no abriendo el editor. Blueprints y varios assets modernos usan serialización que puede no contener el marcador esperado en los primeros kilobytes.

---

## 5. Inventario detallado por carpeta

### 5.1 `Content/ThirdPerson/` — plantilla y punto de entrada

| Archivo | Tipo | Descripción |
|---|---|---|
| `Blueprints/BP_ThirdPersonCharacter` | Blueprint (Character) | Personaje jugador principal. |
| `Blueprints/BP_ThirdPersonGameMode` | Blueprint (GameModeBase) | GameMode por defecto. |
| `Blueprints/BP_ThirdPersonPlayerController` | Blueprint (PlayerController) | Controlador del jugador; añade Input Mapping Contexts y UI táctil. |
| `Lvl_ThirdPerson.umap` | Redirector → `GRAND_CANYON_PROTOTIPE` | Mapa de arranque (redirigido). |
| `GRAND_CANYON_PROTOTIPE.umap` | Redirector → `SCENA` | Redirección intermedia. |
| `SCENA.umap` | Redirector | Redirección final (cadena de renombrados). |
| `MI_ThirdPersonColWay` | Material Instance | Material de colisión/camino de la plantilla. |
| `_GENERATED/USUARIO/Path_3A54D716` | Asset generado | Ruta/navegación generada por el editor. |

**Cadena de redirecciones de mapa:** `Lvl_ThirdPerson` → `GRAND_CANYON_PROTOTIPE` → `SCENA`. Es decir, el mapa de arranque configurado en `DefaultEngine.ini` es en realidad un redirector; el mapa real cargado está en otra ubicación (ver `Scenes/`). Esto es un riesgo de mantenimiento: si se rompe la cadena, el juego no arranca en el mapa esperado.

### 5.2 `Content/Personaje/` — personaje jugador (SWAT) y arsenal

**Modelo del jugador y materiales**

| Archivo | Descripción |
|---|---|
| `agent_Swat` | Skeletal Mesh del soldado SWAT. |
| `agent_Swat_Anim` | Anim Blueprint / asset de animación del SWAT. |
| `agent_Swat_Skeleton` | Esqueleto del SWAT. |
| `BS_locomotion` | Blend Space de locomoción. |
| `green_swat_Body` + `_Diffuse` + `_Opacity` + `_Specular` | Cuerpo y sus texturas. |
| `green_swat_Top`, `Bottom`, `Shoes`, `Glove`, `Hat`, `Eyewear` (+ `_Diffuse`, `_Opacity`, `_Specular`) | Piezas de vestuario y texturas. |

**Animaciones del personaje (`Personaje/Animations/`)**

- `Character` — Anim Blueprint principal (probablemente el usado por el jugador).
- `Idle`, `Walking`, `Slow_Run`, `Standing_Jump`.

**Armas (`Personaje/Armas/`)**

| Ruta | Descripción |
|---|---|
| `Pistola/Pistola` | Skeletal Mesh de la pistola. |
| `Pistola/PistolLocomotion` | Blend Space de locomoción con pistola. |
| `Pistola/Pistol_Idle` | Idle con pistola. |
| `Pistola/Pistol_Walk`, `Pistol_Walk_Backward` | Caminar adelante/atrás. |
| `Pistola/Pistol_Strafe`, `Pistol_Strafe_2` | Strafe lateral. |
| `Pistola/Pistol_Jump` | Salto con pistola. |
| `Pistola/Anim_SacarArma` | Animación de desenfundar. |
| `Pistola/Anim_GuardarArma` | Animación de guardar. |
| `Pistola/Put_Back_Rifle_Behind_Shoulder` | Animación de rifle (nombre heredado). |
| `Pistola/Toss_Grenade` | Animación de lanzar granada (sin sistema de granadas visible). |
| `Rifle/Rifle_Run`, `Rifle/Jump_Up` | Animaciones sueltas de rifle (sin mesh de rifle). |
| `Mira/mira` | Mesh/asset de la mira. |
| `Mira/WBP_Crosshair` | Widget de mira (HUD). |

> El arsenal está **incompleto/inconsistente**: hay animaciones de rifle y de granada, pero no se encontró una malla de rifle ni un Blueprint de granada. La pistola es el arma realmente implementada en el personaje.

**Zombis (`Personaje/Zombies/`)**

| Subcarpeta | Contenido |
|---|---|
| `Zombie_Comun/` | `BP_IA` (zombi básico), mesh `Ch10_nonPBR` (+ `_Anim`, `_Skeleton`), texturas `Ch10_1001_*` / `Ch10_1002_*` / `Ch10_body*`, animación `Walking_Zombie`. |
| `furious_Zombie/` | `BP_IA_ZOmbie_Run` (zombi corredor), mesh `Zombiegirl_W_Kurniawan` (+ `_Anim`, `_Skeleton`), texturas `zombie_*` y `ZombieGirl_*`, animación `Zombie_Run`. |
| `AcidZombie/` | Mesh `Parasite_L_Starkie` (+ `_Anim`, `_Skeleton`) y sus materiales/texturas (`parasiteZombie_*`). **Sin Blueprint**: solo arte. |
| `Boss/` | Mesh `Skeletonzombie_T_Avelange` (+ `_Anim`, `_Skeleton`) y materiales/texturas (`skeletonZombie_*`, `*_glow`). **Sin Blueprint**: solo arte. |

### 5.3 `Content/Characters/` — maniquíes de la plantilla (128 assets, 125 MB)

Contiene el paquete estándar de maniquíes de UE5:

- `Meshes/`: `SKM_Manny_Simple`, `SKM_Quinn_Simple`, `SK_Mannequin`.
- `Rigs/`: `CR_Mannequin_Body`, `CR_Mannequin_FootIK`, `CR_Mannequin_Procedural`, `PA_Mannequin` (Control Rigs y Physics Asset).
- `Anims/`: animaciones de Pistol (Aim, Jog, Jump, Walk), Rifle (AIM, HitReact, Jog, Jump, Walk), Unarmed (Attack, Jog, Jump, Walk), Death.
- `Materials/` y `Textures/`: materiales y texturas de Manny/Quinn.

> Es el paquete más pesado después de `Personaje`, pero **no se encontró referencia** a `SKM_Manny`/`SKM_Quinn` fuera de las propias animaciones del maniquí. Funciona como librería de animaciones reutilizable; se podría externalizar o eliminar si no se usa.

### 5.4 `Content/Models/Pumpkinhulk/` — zombi gigante

| Archivo | Descripción |
|---|---|
| `Pumpkinhulk_L_Shaw` | Skeletal Mesh del zombi gigante. |
| `Pumpkinhulk_L_Shaw_Skeleton` | Esqueleto. |
| `Pumpkinhulk_L_Shaw_PhysicsAsset` | Physics Asset. |
| `PumpkinHulk_Material` | Material. |
| `pumpkinHulk_diffuse`, `pumpkinHulk_normal` | Texturas. |

Este mesh es el usado por `BP_IA_Zombie_Grande`.

### 5.5 `Content/Animations/` — animaciones del zombi grande (carpeta plana)

| Archivo | Descripción |
|---|---|
| `AnimBP_Zombie_Grande` | Anim Blueprint del zombi gigante. |
| `Idle__Zombie_Grande` | Idle. |
| `Walk_Zombie_Grande` | Caminar. |
| `Attack_Zombie_Grande` | Ataque. |
| `AM_Zombie_Grande_Attack` | **Anim Montage** de ataque. Contiene los notifies `BeginAttackDamage` y `FinishAttack`, lo que indica que el daño se aplica entre esos dos eventos. |

> Problema de organización: estas animaciones están **fuera** de `Content/Personaje/Zombies/`, en una carpeta plana genérica. Deberían moverse a una subcarpeta del zombi grande para mantener coherencia.

### 5.6 `Content/IA/` — inteligencia artificial

| Archivo | Tipo | Descripción |
|---|---|---|
| `BP_IA_Zombie_Grande` | Blueprint | Zombi grande controlado por Behavior Tree. |
| `AIC_Zombie_Grande` | AIController | Controlador de IA del zombi grande. |
| `BT_Zombie_Grande` | Behavior Tree | Árbol de comportamiento principal. |
| `BB_Zombie_Grande` | Blackboard | Claves: `HasSeePlayer?` (bool), `ObjectReferenceEnemy` (Object), `PosicionEnemy` (Vector), `SelfActor`. |
| `BTT_Zombie_Grande_Attack` | BT Task | Tarea de ataque. |
| `BTT_Zombie_Grande_Wander` | BT Task | Tarea de deambular. |
| `AIC_Enemigo` / `BT_Enemigo` / `BB_Enemigo` / `BTT_Wander` | Set legacy | IA anterior, usada por `BP_IA` (zombi común). |
| `BP_IA` | **Redirector** | Apunta a `Content/Personaje/Zombies/Zombie_Comun/BP_IA`. Asset duplicado/huérfano. |

**Comportamiento inferido del Blackboard:** el zombi detecta al jugador (`HasSeePlayer?`), guarda una referencia al enemigo (`ObjectReferenceEnemy`) y su posición (`PosicionEnemy`) para perseguir/atacar.

**Comportamiento inferido del `BT_Zombie_Grande`:** incluye nodos estándar de Move To, Wait, selección de tareas y referencias a `BTT_Zombie_Grande_*`. Se detectó una referencia a **`/Game/IA/BTT_Atacar` que no existe en el proyecto** (asset colgante/roto).

### 5.7 `Content/rayo_Laser/` — trampas y pickups

| Archivo | Tipo | Descripción |
|---|---|---|
| `BP_LaserTrap` | Blueprint | Trampa de láser. Variables `LaserStart`, `LaserEnd`, `Vida`. Usa `LineTraceSingle` y `Subtract` para calcular daño/distancia, y `PrintString`. |
| `BP_LaserTrap2` | Blueprint | Variante de la trampa (referencia al mesh `/Game/Objetos/rayo_Laser`). |
| `BP_DamageTrigger` | Blueprint | Trigger que resta vida al jugador al entrar en contacto. Componente `trampa` con evento `OnComponentBeginOverlap`; usa `Subtract (Double)` y `PrintString`. Referencia a `BP_ThirdPersonCharacter`. |
| `BP_Health_trigger` | Blueprint | Trigger que **suma** vida (usa `Add (Double)`). Misma estructura que el de daño. |
| `Botiquin` | Blueprint | Botiquín / pickup de salud. |
| `Zombie` | Blueprint | Probablemente un actor de zombi simple/colocable en escena. |

> La "vida" del jugador parece gestionarse con una variable **local al Blueprint** (tipo `double`) y mostrarse con `PrintString`, no con un componente de salud ni con el `Health` del Character. Es una implementación de prototipo.

### 5.8 `Content/Objetos/` — mallas y materiales auxiliares

| Archivo | Descripción |
|---|---|
| `big_health` | Mesh/material de salud grande. |
| `laser` | Mesh/material de láser. |
| `rayo_Laser` | Mesh/material del rayo (usado por `BP_LaserTrap2`). |
| `StaticMeshZombie` | Malla estática de zombi (decorativa). |
| `StaticMeshZombieFurioso` | Malla estática de zombi furioso (decorativa). |

### 5.9 `Content/Joints/`

| Archivo | Descripción |
|---|---|
| `TRAMPA` | Asset suelto (¿mesh de trampa?). Carpeta de un solo archivo, sin organización clara. |

### 5.10 `Content/Materials/` — materiales propios

| Archivo | Descripción |
|---|---|
| `MAT_Arena` | Material del suelo/arena. Referencia a `/Game/Texturas/Arena/` → `baseColor`, `Cristal`, `normal`, `orm`. Es un material complejo con funciones y colecciones de parámetros (`MaterialFunction`, `MaterialParameterCollection`). |
| `MAT_Color` | Material de color plano/paramétrico. |
| `MAT_Cristal` | Material de cristal (probablemente translúcido/refractivo). |
| `MAT_Espejo` | Material de espejo. |
| `Rock1` | Material de roca. |

### 5.11 `Content/Texturas/` — texturas

| Ruta | Uso |
|---|---|
| `rock`, `rock_Bump` | Textura y bump de roca. |
| `Arena/baseColor` | Color base del material de arena. |
| `Arena/normal` | Normal map. |
| `Arena/orm` | Oclusión/Roughness/Metallic empaquetados. |
| `Arena/roughness` | Rugosidad. |
| `Arena/height` | Height map (parallax/displacement). |
| `Arena/ambientOcclusion` | AO. |
| `Arena/Cristal` | Textura específica de cristal. |

### 5.12 `Content/LevelPrototyping/` — kit de prototipado

- `Meshes/`: `SM_ChamferCube`, `SM_Cube`, `SM_Cylinder`, `SM_Plane`, `SM_QuarterCylinder`, `SM_QuarterCylinderOuter`, `SM_Ramp`.
- `Materials/`: `M_FlatCol`, `M_PrototypeGrid`, `MF_ProcGrid`, `MI_DefaultColorway`, `MI_PrototypeGrid_Gray`, `MI_PrototypeGrid_Gray_02`, `MI_PrototypeGrid_Gray_Round`, `MI_PrototypeGrid_TopDark`.
- `Textures/`: `T_GridChecker_A`.
- `Interactable/Door/`: `BP_DoorFrame` + `SM_Door`, `SM_DoorFrame_Corner`, `SM_DoorFrame_Edge`.
- `Interactable/JumpPad/`: `BP_JumpPad`, Niagara `NS_JumpPad`, materiales `MI_GlowNT`, `M_GradientGlow`, `M_SimpleGlow`, meshes `SM_CircularBand`, `SM_CircularGlow`.
- `Interactable/Target/`: `BP_WobbleTarget` + `SM_TargetBaseMesh`.

> `BP_DoorFrame` conserva referencias heredadas a `/Game/FirstPerson/...` y `/Game/LevelPrototyping/Door/...` (redirectores de la plantilla). `BP_WobbleTarget` referencia `/Game/DemoTemplate/_Core/MI_Intro_Colorway`, que **no existe** en el proyecto.

### 5.13 `Content/Input/` — Enhanced Input

**Input Mapping Contexts**

| Archivo | Uso |
|---|---|
| `IMC_Default` | Movimiento, salto, correr, disparar, apuntar, equipar. |
| `IMC_MouseLook` | Rotación de cámara con ratón. |

**Input Actions (`Input/Actions/`)**

| Archivo | Nombre lógico |
|---|---|
| `IA_Move` | Movimiento |
| `IA_Look` | Rotación (gamepad) |
| `IA_MouseLook` | Rotación (ratón) |
| `IA_Jump` | Salto |
| `IA_Shoot` | Disparar |
| `IA_Apuntar` | Apuntar |
| `IA_Correr` | Correr |
| `IA_Equipar` | Equipar arma |

**UI táctil (`Input/Touch/`)**

| Archivo | Uso |
|---|---|
| `BPI_TouchInterface` | Blueprint Interface para la interfaz táctil. |
| `UI_Thumbstick` | Joystick virtual. |
| `UI_TouchSimple` | Widget táctil simple (añadido por el PlayerController). |

---

## 6. Mapas y escenas

### 6.1 Listado de `.umap`

| Mapa | Ubicación | Estado |
|---|---|---|
| `Lvl_ThirdPerson` | `ThirdPerson/` | **Redirector** → `GRAND_CANYON_PROTOTIPE`. |
| `GRAND_CANYON_PROTOTIPE` | `ThirdPerson/` | **Redirector** → `SCENA`. |
| `SCENA` | `ThirdPerson/` | Redirector (cadena heredada). |
| `GRAND_CANYON` | `Scenes/` | Mapa real de cañón. |
| `GRAND_CANYON_PROTOTIPE` | `Scenes/` | Mapa real de prototipo de cañón/arena. |
| `LABORATORIOS` | `Scenes/` | Laboratorios. |
| `SECRET_LAB` | `Scenes/` | Laboratorio secreto. |
| `SECRET_LAB_Prototipe` | `Scenes/` | Prototipo del laboratorio secreto. |
| `SECRET_LAB_MAP` | `Scenes/` | Variante de mapa. |
| `SECRET_LAB_MAP1` | `Scenes/` | Variante casi idéntica de `SECRET_LAB_MAP` (difiere ~58 bytes), sin referencias externas. |
| `NewMap` | `Scenes/` | **Redirector** → `SECRET_LAB`. |

### 6.2 Composición de mapas

Todos los mapas usan **World Partition** (existen `__ExternalActors__`, `__ExternalObjects__`, `WorldPartitionMiniMap`, Data Layers). Por eso la escena está repartida en 104 archivos de actores externos.

**`Scenes/GRAND_CANYON_PROTOTIPE`** (el más trabajado)

- Materiales: `MAT_Arena`, `MAT_Cristal`, `MAT_Color`, `MAT_Espejo`.
- Actors: `BP_LaserTrap`, `BP_IA_Zombie_Grande`, `Pistola` (malla del arma), suelos y caminos (`Floor`, `Path`).
- Iluminación estándar (Directional Light, Sky, Exponential Height Fog, Volumetric Clouds).
- Datos de navegación (`NavMesh`, `RecastNavMesh`).

**`Scenes/LABORATORIOS`**

- **15 × `BP_DamageTrigger`**, **7 × `BP_Health_trigger`**, **4 × `BP_LaserTrap`**.
- 2 zombis.
- NavMesh.
- Es el mapa con más lógica de "trampas y curación" colocada.

**`Scenes/SECRET_LAB` / `SECRET_LAB_Prototipe` / `SECRET_LAB_MAP` / `SECRET_LAB_MAP1`**

- Variantes del laboratorio secreto, con iluminación direccional y geometría de prototipado.
- `SECRET_LAB_MAP` referencia 12 rutas generadas (`Scenes/_GENERATED/USUARIO/Path_*`).
- `SECRET_LAB_MAP1` es un duplicado sin referencias: probablemente un backup manual.

**`Scenes/GRAND_CANYON`**

- Versión "final" del cañón (sin los detalles del prototipo).

### 6.3 Rutas generadas (`_GENERATED/USUARIO/Path_*`)

Hay 13 assets `Path_*` entre `Scenes/` y `ThirdPerson/`. Son datos generados por el editor (por ejemplo, caminos de navegación o *splines* generados). No son arte autoría manual y **no deberían versionarse** salvo que el equipo sepa que se regeneran.

---

## 7. Sistemas de gameplay

### 7.1 Personaje jugador (`BP_ThirdPersonCharacter`)

**Componentes**

- `CapsuleComponent` (heredado de Character).
- `CameraBoom` (Spring Arm).
- `FollowCamera` (Cámara).
- `Pistola` (Skeletal Mesh) montada en el socket **`Pistol_Socket`**.
- `CharacterMovement`.

**Variables detectadas**

| Variable | Tipo | Uso |
|---|---|---|
| `IsAiming` | bool | Estado de apuntado. |
| `IsEquipped` | bool | Estado de arma equipada. |

**Lógica detectada**

- Enhanced Input con eventos `Triggered`, `Started`, `Completed`, `Ongoing`.
- Apuntado (`Aim`) con interpolación de cámara/brazo (`Timeline`).
- Disparo con **`LineTraceSingle`** → `BreakHitResult` → `ApplyDamage` / `AddImpulseAtLocation` sobre el actor impactado.
- Uso de `WBP_Crosshair` (widget de mira).
- Soporte de interfaz táctil.
- Referencias a `IA_Shoot` e `IA_Equipar`.

**Flujo de disparo inferido**

1. `IA_Shoot` (Triggered) → line trace desde la cámara/muzzle.
2. `BreakHitResult` → obtener `HitActor`, `TraceStart`, `TraceEnd`.
3. `ApplyDamage` al actor impactado (el zombi lo recibe).
4. `AddImpulseAtLocation` para empujar objetos físicos.

### 7.2 Cámara y apuntado

- Cámara en tercera persona con Spring Arm (`CameraBoom`).
- `IMC_MouseLook` se añade desde el PlayerController para la rotación con ratón.
- Al apuntar (`RightMouseButton`) se activa `IsAiming`, que típicamente acerca la cámara y activa la mira (`WBP_Crosshair`).

### 7.3 Arma

- Solo hay **pistola** implementada como Skeletal Mesh (`Pistola`).
- Socket de montaje: `Pistol_Socket`.
- Animaciones: idle, walk, walk backward, strafe (x2), jump, desenfundar, guardar, lanzar granada, rifle (sueltas).
- No hay sistema de munición, recarga ni cambio de arma visible en los assets (solo `IsEquipped`).

### 7.4 HUD / Mira

- `WBP_Crosshair` (Widget Blueprint). Se muestra/oculta según el estado de apuntado.
- `UI_TouchSimple` y `UI_Thumbstick` para controles táctiles.

### 7.5 Salud y daño

- `BP_DamageTrigger`: al solaparse el jugador, resta vida (`Subtract (Double)`) y muestra el valor con `PrintString`.
- `BP_Health_trigger`: suma vida (`Add (Double)`).
- `Botiquin`: pickup de salud.
- `BP_LaserTrap` / `BP_LaserTrap2`: trampas que restan vida (`Vida`) usando `LineTraceSingle` + `Subtract`.
- La variable de vida parece ser **local al Blueprint** (tipo `double`), no un componente reutilizable. No se encontró un `HealthComponent` ni `ApplyDamage` centralizado en el jugador.

> Consecuencia: el sistema de salud es de prototipo y está **duplicado** en varios Blueprints. Es el primer candidato a refactorizar (un `BP_HealthComponent` o `AttributeSet` de GAS).

### 7.6 IA de zombis

**Zombi común (`BP_IA` en `Personaje/Zombies/Zombie_Comun`)**

- `PawnSensingComponent` con evento `OnSeePawn`.
- Variable `ISeeEnemy` (bool) y `TargetActor`.
- Al ver al jugador: `CreateMoveToProxyObject` + `K2_GetRandomReachablePointInRadius` para perseguir/deambular.
- Animación: `Walking_Zombie` (single node).
- AIController: `AIC_Enemigo`.

**Zombi furioso (`BP_IA_ZOmbie_Run`)**

- `PawnSensingComponent`.
- Eventos `ReceiveActorBeginOverlap`, `ReceiveBeginPlay`, `ReceiveTick`.
- Animación: `Zombie_Run`.
- Sin Behavior Tree propio detectado (lógica en el Blueprint).

**Zombi grande (`BP_IA_Zombie_Grande`)**

- Mesh: `Pumpkinhulk_L_Shaw`.
- `PawnSensingComponent`.
- Ataque por **sphere trace** + `ApplyDamage`.
- Reproduce montage `AM_Zombie_Grande_Attack` (notifies `BeginAttackDamage` / `FinishAttack`).
- Variable `IsAttacking`.
- Controlado por `AIC_Zombie_Grande` + `BT_Zombie_Grande` + `BB_Zombie_Grande`.

**Zombi ácido y Jefe**

- Solo existen **mallas, esqueletos, animaciones y materiales**. **No hay Blueprints** que los usen. Son arte pendiente de integrar.

### 7.7 Interactables del kit de prototipado

- `BP_DoorFrame`: puerta con marco (hereda redirectores de la plantilla).
- `BP_JumpPad`: plataforma de salto con Niagara `NS_JumpPad` y materiales glow.
- `BP_WobbleTarget`: diana oscilante para pruebas de disparo.

---

## 8. Configuración de input (`DefaultInput.ini`)

- Clases de Enhanced Input configuradas (`DefaultPlayerInputClass`, `DefaultInputComponentClass` sobre `EnhancedInput`).
- Tecla de consola: `Tilde`.
- Los mappings reales viven en los assets `IMC_Default` e `IMC_MouseLook`.

### Mapeos observados en los IMC

| Acción | Teclas / Ejes |
|---|---|
| `IA_Move` | `W/A/S/D`, `Gamepad_Left2D` |
| `IA_Look` | `Gamepad_Right2D` |
| `IA_MouseLook` | Movimiento del ratón (X/Y) |
| `IA_Jump` | `SpaceBar`, `Gamepad_FaceButton_Bottom` |
| `IA_Shoot` | `LeftMouseButton` |
| `IA_Apuntar` | `RightMouseButton` |
| `IA_Correr` | `LeftShift` |
| `IA_Equipar` | Sin mapeo claro detectado (posiblemente sin asignar) |

> `IA_Equipar` existe como asset pero no se encontró una tecla asociada en los IMC. Es un hueco funcional: el arma no se puede (des)equipar con una tecla conocida.

---

## 9. Materiales, texturas y render

- **Materiales propios**: `MAT_Arena` (complejo, con texturas de arena), `MAT_Color`, `MAT_Cristal`, `MAT_Espejo`, `Rock1`.
- **Texturas**: set de arena (baseColor, normal, orm, roughness, height, AO, Cristal) y roca (`rock`, `rock_Bump`). Todo en `Content/Texturas/`.
- **Kit de prototipado**: materiales grid/plano y una función procedural (`MF_ProcGrid`).
- **Render**: Lumen + Ray Tracing + Substrate + Virtual Shadow Maps + Mesh Distance Fields. Es una configuración **pesada** (calidad máxima, DX12 SM6). Para un prototipo en equipos modestos conviene bajar Lumen/RT o usar un perfil escalable.
- **Sin audio**: no se encontró ningún `SoundWave` ni `SoundCue`. El juego es **mudo**.

---

## 10. Estado del control de versiones

- Rama actual: `main`.
- Cambios: **7 archivos modificados** (`M`) y **11 rutas sin seguimiento** (`??`).
- Sin seguimiento destacado:
  - `Content/Animations/`
  - `Content/IA/AIC_Zombie_Grande`, `BB_Zombie_Grande`, `BP_IA_Zombie_Grande`, `BTT_Zombie_Grande_Attack`, `BTT_Zombie_Grande_Wander`, `BT_Zombie_Grande`
  - `Content/Models/`
  - Actores externos nuevos de `Scenes/GRAND_CANYON_PROTOTIPE`
- `.gitattributes` fuerza LFS para `.uasset` y `.umap`. Hay que asegurarse de que Git LFS esté instalado y de que los assets nuevos se añadan con `git lfs track` (ya cubierto por el patrón).

> Recomendación: hacer un commit de "estado actual" antes de seguir tocando contenido, para tener un punto de restauración. Los assets nuevos de IA, modelos y animaciones son el trabajo más reciente y no están versionados.

---

## 11. Problemas conocidos y deuda técnica

| # | Problema | Ubicación | Impacto |
|---|---|---|---|
| 1 | Mapa de arranque es un redirector en cadena (`Lvl_ThirdPerson` → `GRAND_CANYON_PROTOTIPE` → `SCENA`) | `ThirdPerson/`, `DefaultEngine.ini` | Riesgo de arranque en mapa incorrecto. |
| 2 | `BT_Zombie_Grande` referencia `/Game/IA/BTT_Atacar` inexistente | `IA/BT_Zombie_Grande` | Nodo roto en el árbol; el zombi grande puede no atacar correctamente. |
| 3 | `BP_WobbleTarget` referencia `/Game/DemoTemplate/_Core/MI_Intro_Colorway` inexistente | `LevelPrototyping/Interactable/Target` | Material faltante (posible warning al cargar). |
| 4 | `BP_DoorFrame` conserva redirectores a `/Game/FirstPerson/...` | `LevelPrototyping/Interactable/Door` | Deuda de la plantilla; puede romperse al limpiar redirectores. |
| 5 | `Content/IA/BP_IA` es un redirector duplicado de `Personaje/Zombies/Zombie_Comun/BP_IA` | `IA/` | Confusión de rutas; candidato a eliminar. |
| 6 | `Content/IA_Zombie_Grande/0.uasset` es un material suelto | `IA_Zombie_Grande/` | Carpeta sin sentido; candidato a mover/eliminar. |
| 7 | Animaciones del zombi grande en carpeta plana `Animations/` | `Content/Animations/` | Desorganización. |
| 8 | `IA_Equipar` sin tecla asignada | `Input/IMC_*` | No se puede equipar/guardar el arma. |
| 9 | Vida implementada como variable local duplicada | `rayo_Laser/*` | Sin sistema de salud centralizado. |
| 10 | Sin audio en todo el proyecto | — | El juego no tiene sonido. |
| 11 | Sin munición, recarga ni cambio de arma | `BP_ThirdPersonCharacter` | Sistema de combate incompleto. |
| 12 | Zombi ácido y Jefe sin Blueprint | `Personaje/Zombies/` | Arte no integrado. |
| 13 | `SECRET_LAB_MAP1` duplicado de `SECRET_LAB_MAP` | `Scenes/` | Archivo redundante. |
| 14 | 13 assets `Path_*` generados versionados | `Scenes/_GENERATED`, `ThirdPerson/_GENERATED` | Ruido en el repo. |
| 15 | `README.md` vacío (solo el título) | raíz | Sin documentación de entrada. |
| 16 | `ProjectName` sigue siendo "Third Person BP Game Template" | `Config/DefaultGame.ini` | Nombre incorrecto en build/empaquetado. |
| 17 | `Content/Characters/` (125 MB) no parece usarse fuera de sí mismo | `Characters/` | Peso innecesario si se confirma. |
| 18 | Sin `.gitignore` específico de Unreal moderno | raíz | Posibles artefactos versionados. |

---

## 12. Resumen de números

| Métrica | Valor |
|---|---:|
| Assets `.uasset` | 336 |
| Mapas `.umap` | 11 (7 reales + 4 redirectores) |
| Blueprints de gameplay identificados | ~15 |
| Behavior Trees | 7 |
| Blackboards | 2 |
| Personajes jugables | 1 (SWAT) |
| Tipos de zombi con lógica | 3 (común, furioso, grande) |
| Tipos de zombi solo arte | 2 (ácido, jefe) |
| Armas jugables | 1 (pistola) |
| Mapas jugables reales | 6 |
| Assets de audio | 0 |
| Tamaño total de `Content/` | ~374 MB |

---

## 13. Metodología de esta lectura

El puente MCP del plugin `UnrealOpenCode` **no respondió** durante la sesión (`UE not connected: Request timeout`), así que la inspección se hizo sin abrir el editor, mediante:

1. Lectura de archivos de texto: `.uproject`, `Config/*.ini`, `.gitignore`, `.gitattributes`, `README.md`.
2. Recorrido del árbol de `Content/` con PowerShell (nombres, tipos, tamaños).
3. Extracción de cadenas ASCII de los binarios `.uasset`/`.umap` para reconstruir:
   - nombres de paquetes y rutas de referencia (`/Game/...`, `/Script/...`),
   - nombres de clases, variables, funciones y nodos de Blueprint (`CallFunc_*`, `BndEvt_*`, `K2Node_*`),
   - nombres de notifies de montages,
   - referencias a assets faltantes.
4. Verificación cruzada de referencias (qué asset referencia a cuál) para detectar huérfanos y enlaces roscos.
5. Lectura de logs del editor en `Saved/Logs/` para confirmar carga del plugin y errores de arranque.

**Limitación**: sin el editor abierto no se puede garantizar el 100 % de la lógica de un Blueprint (los grafos están serializados y las cadenas solo revelan nombres de nodos, no el cableado exacto). Las descripciones de flujo están marcadas como "inferidas".

---

## 14. Próximos pasos sugeridos

1. **Abrir el editor** y validar los 3 sistemas críticos: disparo, daño y ataque del zombi grande.
2. **Reparar** la referencia `BTT_Atacar` del `BT_Zombie_Grande`.
3. **Asignar** una tecla a `IA_Equipar` o eliminar la acción.
4. **Centralizar la salud** en un componente reutilizable.
5. **Reorganizar** `Content/Animations/` y `Content/IA_Zombie_Grande/` dentro de `Personaje/Zombies/`.
6. **Limpiar redirectores** (`Content/IA/BP_IA`, mapas `SCENA`, `NewMap`) con la herramienta *Fix Up Redirectors*.
7. **Renombrar** el proyecto en `DefaultGame.ini` y completar `README.md`.
8. **Commit** del estado actual (incluyendo los assets nuevos de IA/modelos) antes de refactorizar.
9. Evaluar si `Content/Characters/` (125 MB) se usa; si no, externalizarlo.
10. Añadir audio básico (pasos, disparo, gruñidos) para que el prototipo se sienta completo.

---

*Fin del documento. Las actualizaciones de progreso se llevan en `Bitacora/Avances.md`.*
