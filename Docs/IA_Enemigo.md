# IA del enemigo GASP

`/Game/AI/BP_EnemyAIController` controla `BP_EnemyGASP_Ragdoll`, que conserva su herencia de `SandboxCharacter_Mover_Ragdoll`. Usa las solicitudes de navegación de AIController y el NavMover que GASP ya consume en `Get_MoveInput`.

## Comportamiento

- Patrulla puntos alcanzables dentro de 650 cm de su posición inicial. Reintenta las consultas fallidas cada dos segundos.
- Consulta visión cada 0,2 segundos: jugador local, distancia máxima de 1800 cm, cono horizontal de 120 grados y `LineOfSightTo` para comprobar obstáculos.
- Persigue al jugador visible; el margen de llegada del path following es de 90 cm. Al estar a 180 cm o menos intenta atacar.
- Al perder visión, camina hacia la última posición observada. Tras tres segundos sin verlo, abandona la persecución y vuelve a patrullar.
- Al recibir daño válido detiene inmediatamente la navegación. Mientras reproduce el montage de impacto no inicia nuevas rutas; después reanuda su comportamiento.
- Al morir cancela movimiento y el temporizador de IA. Conserva el ragdoll y la anulación de `NPC_GetupCheck` que impide levantarse.

Los parámetros `RoamRadius`, `SightRadius`, `SightDotThreshold` (coseno del semiángulo), `LoseSightDelay` y `ChaseAcceptanceRadius` se ajustan en Class Defaults del controlador. `State`, `CanSeePlayer` y `TargetPawn` permiten observar el comportamiento durante PIE.

## Nivel

`L_Blockout_Nivel01` contiene `Enemy_NavMeshBounds`, centrado en (8500, 4500, 200), con límites (4500, 500, -800) a (12500, 8500, 1200). Su `RecastNavMesh-Default` usa generación Dynamic para disponer de navegación en PIE. Para ampliar el escenario hay que ampliar este volumen o añadir cobertura de navegación.

El enemigo colocado en el nivel tiene el controlador asignado explícitamente y Auto Possess AI = Placed in World or Spawned; los mismos valores están establecidos en su Blueprint.

## Comprobaciones

Verificado en PIE: desplazamiento real con Mover durante persecución; patrulla y elección de destinos; abandono del objetivo al alejar al jugador; ausencia de detección con el jugador detrás y adquisición con él delante. El registro no mostró Blueprint Runtime Error ni Accessed None.

También se ejecutó `BP_EnemyHitReactionTest` con la IA habilitada: seis comprobaciones aprobadas, incluyendo golpes consecutivos, reinicio y finalización del montage, rechazo del daño negativo y muerte con pelvis simulando física. El controlador terminó en estado Dead. El actor temporal de prueba se retiró antes de guardar.

Los Blueprints del controlador y enemigo compilaron con warnings tratados como errores. Copia del enemigo anterior a los cambios de daño de esta sesión: `Saved/EnemyAIBackup/BP_EnemyGASP_Ragdoll.uasset`.

Los tres enlaces de YouTube proporcionados no pudieron recuperarse. No se ha verificado una reproducción exacta de los pasos de esos videos.

## Ataques y muerte del jugador

El enemigo equipa `BP_Sword` en el socket Weapon del SkeletalMesh de GASP. Alterna `Anim_PS_Combo_A1_RM_Montage` y `Anim_PS_Combo_A2_RM_Montage`, reproducidos con Play Montage (Mover Actor). La dirección se calcula hacia el jugador al iniciar cada golpe. Un Teleport Instant Movement Effect de Mover, en la misma posición y con la nueva orientación, sincroniza la rotación con la simulación; cambiar únicamente SetActorRotation hacía que Mover restaurara la orientación anterior.

Los notifies originales `State_WeaponTrace` abren y cierran la ventana mediante el AC_Combat heredado. Su tick sigue desactivado porque presupone un Character CMC. `UpdateEnemyCombat`, después del tick de GASP, consume esa ventana y hace un BoxTrace de la hoja, entre TopTrace y BottomTrace, con semiextensión (10,10,10). Consulta objetos Pawn para detectar al jugador y comprueba Visibility para impedir impactos a través de obstáculos. Solo daña al objetivo del ataque; ignora al enemigo, su espada y los objetivos alcanzados durante el mismo golpe. No aplica daño por mera proximidad.

`AttackRange` = 180 cm, `EnemyWeaponDamage` = 25 y `AttackCooldown` = 0,9 segundos después de terminar o interrumpir el montage. Se ajustan en el Blueprint del enemigo. Durante el ataque la IA pausa el path following. Un golpe recibido cancela el ataque y su trace antes de iniciar la reacción; la muerte cancela también la navegación. El controlador no adquiere como objetivo a un jugador muerto.

`SandboxCharacter_CMC.HandleCombatDamage` usa el AC_Health existente, con 100 de vida inicial. Al llegar a cero ejecuta una sola vez `EnterDeathRagdoll`: detiene los montages, cierra el daño de la espada, desactiva el combate y la entrada y llama a `Ragdoll_Start` de GASP. Al morir, el movimiento queda en None, la cápsula pierde colisión y el cuerpo simula física desde pelvis. Mientras vive, la cápsula conserva la configuración anterior: bloquea Pawn e ignora Visibility, Camera y PhysicsBody.

## Revisión de movimiento del jugador

Tras el reporte de movimiento entrecortado se comparó el jugador con `Saved/EnemyAttackBackup/SandboxCharacter_CMC.uasset`. Los valores del Pawn, CharacterMovement y SkeletalMesh coinciden, así como 13 grafos de entrada, desplazamiento, rotación, velocidades, aceleración y datos de animación. La diferencia encontrada en la cápsula era Visibility = Block; se restauró su BodyInstance anterior completo (Visibility = Ignore) y se trasladó la adaptación de detección al trace del enemigo. No se modificaron los grafos ni parámetros de locomoción del jugador.

Las 11 comprobaciones de `BP_EnemyCombatTest` volvieron a pasar con esta configuración, incluido daño desde cuatro direcciones y ragdoll del jugador. Esto verifica la conservación del combate; la sensación de fluidez reportada requiere confirmación durante juego interactivo. La copia temporal usada para comparar los assets y el actor de pruebas fueron retirados. Copias anteriores a esta revisión: `Saved/PlayerMovementReviewBackup`.

`BP_EnemyCombatTest` pasó 11 comprobaciones en PIE: cancelación del daño tras interrupción, rechazo de daño negativo, orientación y daño único de 25 desde cuatro direcciones y muerte del jugador con pelvis simulando física. También pasaron nuevamente las seis comprobaciones de `BP_EnemyHitReactionTest`. Los cuatro Blueprints modificados o nuevos compilaron con warnings tratados como errores; no hubo Blueprint Runtime Error ni Accessed None. Los actores temporales fueron retirados del nivel. Las copias de los assets anteriores a esta etapa están en `Saved/EnemyAttackBackup`.
