# Tarea: Sistema de Apuntado estilo Resident Evil, Animaciones de Arma y Raycast/Láser (100% Blueprints)

Necesito implementar y corregir en **Blueprints** (sin crear clases en C++) la mecánica completa de equipar/guardar arma, apuntado sobre el hombro estilo *Resident Evil*, inclinación vertical del torso al apuntar, sincronización de mirilla UI y disparo por raycast/láser en mi proyecto **OPERACIONEXODO** (UE 5.7.4).


## 1. Requisitos exactos de la mecánica (100% Blueprints)

### A. Sacar y guardar el arma con la tecla `1` (con animación)
- Configura la acción de equipar/guardar arma con la **tecla `1`** usando las variables booleanas `bHasWeapon` (arma equipada) y `bIsEquipping` (bloqueo temporal mientras ocurre la animación).
- **Al presionar `1` cuando el arma está guardada (`bHasWeapon = false`):**
  1. Reproduce la animación / *Anim Montage* de **sacar el arma** encontrada en `Content/Personaje/Animations` (mezclada mediante un Slot de tren superior con `Layered blend per bone` desde el hueso de columna correspondiente, como `spine_01`, para que no congele las piernas si el jugador camina).
  2. Adjunta o muestra el arma en el socket de la mano del personaje y establece `bHasWeapon = true`.
- **Al presionar `1` cuando el arma está equipada (`bHasWeapon = true`):**
  1. Si el jugador estaba apuntando (`bIsAiming == true`), cancela inmediatamente el apuntado y oculta la mirilla antes de guardar el arma.
  2. Reproduce la animación / *Anim Montage* de **guardar el arma** de `Content/Personaje/Animations`.
  3. Oculta o adjunta el arma al socket de la espalda/funda y establece `bHasWeapon = false`.

---

### B. Apuntado con Click Derecho mantenido (`bIsAiming`)
- Configura una `InputAction` para apuntar con **Click Derecho del mouse** (`Started`/`Triggered` al mantener pulsado, `Completed`/`Canceled` al soltar).
- **Condición obligatoria:** El personaje **solo** puede entrar en modo apuntado (`bIsAiming = true`) si `bHasWeapon == true` y no está en medio de la animación de guardar/sacar el arma.
- **Al mantener pulsado Click Derecho (`bIsAiming = true`):**
  1. **Mirilla Canvas (`WBP_Crosshair`):** Muestra en pantalla el widget `Content/Personaje/Armas/Mira/WBP_Crosshair` centrado en el viewport (y ocúltalo al soltar Click Derecho).
  2. **Cámara estilo Resident Evil (Zoom sobre el hombro):** Interpola suavemente (`Timeline` o `FInterpTo`/`VInterpTo`) el `SpringArm` (`CameraBoom`) acercando la cámara al hombro derecho del personaje (reduciendo `TargetArmLength` y ajustando `SocketOffset` en `Y` y `Z` para que el personaje quede encuadrado a la izquierda y la mirilla `WBP_Crosshair` quede despejada al centro).
  3. **Rotación horizontal del personaje:** Activa `Use Controller Rotation Yaw = true` y desactiva `Orient Rotation to Movement = false` en el `CharacterMovementComponent` para que el personaje gire siempre hacia donde apunta la cámara y pueda desplazarse lateralmente (*strafe*).
  4. **Restricciones de movimiento mientras apunta:**
     - Permite **caminar** (reduce `MaxWalkSpeed` a una velocidad lenta de apuntado, aprox. `200 - 250`).
     - **Bloquea correr (Sprint) y saltar (Jump):** Mientras `bIsAiming == true`, el personaje **no** puede correr ni saltar (si venía corriendo al presionar Click Derecho, cancela el sprint inmediatamente; si presiona Espacio o Shift mientras apunta, ignóralos).

---

### C. Inclinación vertical de la mitad superior del cuerpo al mirar arriba/abajo
- En el `AnimBP` del personaje, configura el apuntado vertical para que **la mitad superior del cuerpo (torso, brazos, cabeza y arma) mire hacia arriba o hacia abajo** siguiendo el ángulo vertical (`Pitch`) de la cámara cuando `bIsAiming == true` (o mientras tiene el arma apuntando):
  - Calcula el `AimPitch` a partir de la diferencia entre `GetBaseAimRotation` (o `GetControlRotation`) y `GetActorRotation` (`Delta (Rotator)` -> `Pitch`, limitado con `Clamp Angle` entre `-75` y `75` grados).
  - Aplícalo usando el *Aim Offset* de `Content/Personaje/Animations` (si existe) o mediante nodos `Transform (Modify) Bone` en los huesos de la columna (`spine_01`, `spine_02`, `spine_03`) combinados con `Layered blend per bone`, de modo que las piernas sigan caminando normal en el suelo mientras la mitad superior se inclina suavemente hacia arriba o abajo según a dónde apunte la mirilla.

---

### D. Disparo con Click Izquierdo y corrección del Raycast / Láser hacia la mirilla
- **Condición de disparo:** El **Click Izquierdo** solo puede disparar cuando `bHasWeapon == true` y `bIsAiming == true`. Si el jugador suelta el Click Derecho (`bIsAiming == false`), no debe poder disparar.
- **Corrección del ángulo del Raycast / Láser (problema actual):**
  - Actualmente el raycast/láser sale desviado hacia un costado del arma en lugar de apuntar hacia la mirilla.
  - Corrígelo calculando la trayectoria en **dos pasos**:
    1. **Traza desde la Cámara hacia el centro de `WBP_Crosshair`:** Lanza un `LineTraceByChannel` desde la ubicación de la cámara (`FollowCamera`) en dirección de su `ForwardVector * RangoMaximo` (el centro exacto de la pantalla donde está `WBP_Crosshair`). Toma el `Impact Point` si golpea un objeto, o el `Trace End` si no golpea nada, y guárdalo como `TargetPoint`.
    2. **Raycast / Láser desde el cañón del arma hacia `TargetPoint`:** Lanza el `LineTraceByChannel` real del disparo (y orienta el efecto/beam de `Content/rayo_Laser`) partiendo desde el **Socket de la boca del cañón del arma** hasta `TargetPoint` (usando `Find Look at Rotation` desde el socket del arma hacia `TargetPoint`). Así el láser siempre saldrá de la punta del arma y coincidirá exactamente con el centro de `WBP_Crosshair`.

---

### E. Al soltar Click Derecho (`bIsAiming = false`)
- Oculta `WBP_Crosshair`.
- Interpola suavemente el `SpringArm` de vuelta a su `TargetArmLength` y `SocketOffset` originales.
- Restaura el `AimPitch` del torso a `0` suavemente.
- Restaura la rotación libre de exploración (`Use Controller Rotation Yaw = false`, `Orient Rotation to Movement = true`) y la velocidad normal (`MaxWalkSpeed`).
- Vuelve a habilitar las acciones de **caminar, correr y saltar**, y bloquea el disparo hasta que se vuelva a mantener Click Derecho.

---

## 2. Entregable esperado

1. Aplica o actualiza la lógica en los Blueprints y el `AnimBP` correspondientes (o detalla paso a paso cada nodo y conexión exacta si algún grafo de Blueprint requiere edición manual en UE 5.7.4), usando los nombres exactos de animaciones, huesos, sockets y widgets encontrados en mi proyecto.
2. Verifica que todos los Blueprints modificados compilen sin errores ni advertencias críticas.