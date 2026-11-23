# Avances — OPERACIONEXODO

> Bitácora de avances, decisiones y pendientes del proyecto.
> La lectura técnica inicial está en [`Bitacora_Lectura_Proyecto.md`](./Bitacora_Lectura_Proyecto.md).

---

## Índice

- [Cómo usar este documento](#cómo-usar-este-documento)
- [Plantilla de entrada](#plantilla-de-entrada)
- [Registro de avances](#registro-de-avances)
- [Pendientes](#pendientes)

---

## Cómo usar este documento

Cada vez que se avance en el proyecto se añade una entrada al final de la sección
**Registro de avances**, con este formato:

```
### AAAA-MM-DD — Título corto

**Objetivo:**
...

**Cambios realizados:**
- ...

**Archivos / assets tocados:**
- ...

**Resultado / verificación:**
...

**Pendiente / siguiente paso:**
...
```

Reglas:

1. Las entradas van **en orden cronológico** (la más reciente al final).
2. No se borran entradas anteriores: si algo se revierte, se documenta en una nueva entrada.
3. Los assets se nombran con su ruta completa desde `Content/` para evitar ambigüedad.
4. Si algo queda roto o a medias, se marca explícitamente con `⚠️`.

---

## Plantilla de entrada

```markdown
### AAAA-MM-DD — Título corto

**Objetivo:**
...

**Cambios realizados:**
- ...

**Archivos / assets tocados:**
- `Content/...`

**Resultado / verificación:**
- [ ] Compila / abre sin errores
- [ ] Probado en PIE
- [ ] Commit realizado

**Pendiente / siguiente paso:**
...
```

---

## Registro de avances

### 2026-11-22 — Lectura inicial del proyecto y creación de la bitácora

**Objetivo:**
Leer y documentar el estado actual del proyecto `OPERACIONEXODO` antes de empezar a modificarlo.

**Cambios realizados:**
- Se recorrió la estructura completa del repositorio (config, content, plugins, binarios, logs).
- Se inventariaron las 18 carpetas de `Content/` (347 archivos, ~374 MB).
- Se reconstruyeron referencias entre assets a partir de los binarios `.uasset` / `.umap`.
- Se identificaron sistemas de gameplay, mapas, IA, input, materiales y deuda técnica.
- Se creó esta carpeta `Bitacora/` con los dos documentos.

**Archivos / assets tocados:**
- `Bitacora/Bitacora_Lectura_Proyecto.md` (nuevo)
- `Bitacora/Avances.md` (nuevo)

**Resultado / verificación:**
- [x] Documentación generada
- [ ] Revisión del contenido por el equipo
- [ ] Commit realizado

**Pendiente / siguiente paso:**
- Validar en el editor los sistemas marcados como "inferidos" (disparo, daño, ataque del zombi grande).
- Reparar la referencia rota `/Game/IA/BTT_Atacar`.

---

## Pendientes

Lista viva de tareas detectadas durante la lectura (ampliar/cerrar a medida que se avance):

- [ ] Abrir el editor y validar los 3 sistemas críticos: disparo, daño, ataque del zombi grande.
- [ ] Reparar la referencia `BTT_Atacar` del `BT_Zombie_Grande`.
- [ ] Asignar tecla a `IA_Equipar` o eliminar la acción.
- [ ] Centralizar la salud en un componente reutilizable.
- [ ] Reorganizar `Content/Animations/` y `Content/IA_Zombie_Grande/`.
- [ ] Limpiar redirectores (`Content/IA/BP_IA`, mapas `SCENA`, `NewMap`).
- [ ] Renombrar el proyecto en `Config/DefaultGame.ini`.
- [ ] Completar `README.md`.
- [ ] Evaluar si `Content/Characters/` (125 MB) se usa.
- [ ] Añadir audio básico.
