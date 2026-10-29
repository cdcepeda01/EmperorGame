# Espada y vida

## Probar

Abre `/Game/Levels/L_Blockout_Nivel01` y pulsa Play. Hay un enemigo GASP Ragdoll delante del PlayerStart. Acércate y ataca con clic izquierdo: su etiqueta muestra la vida restante y cae con ragdoll al llegar a cero. Cada nueva sesión restaura el blanco.

- Vida inicial: 100.
- Daño de espada: 25 por objetivo y ventana de ataque.
- Cuatro impactos matan al enemigo y lo dejan caido con fisica.
- El trace ignora al portador, su espada y los actores ya alcanzados en la misma ventana.

## Assets

- `/Game/Component/AC_Combat`: `WeaponDamage` y `WeaponTraceHalfSize` configurables. `PerformWeaponTrace` utiliza los sockets `TopTrace` y `BottomTrace` de la espada y aplica el daño de Unreal. `ResetWeaponHitActors` reinicia los objetivos ignorados; `EndWeaponAttack` cierra el trace y restaura la entrada al acabar, interrumpirse o comenzar el blend out del montage.
- `/Game/UEFNAnimations/NotifyStates/State_WeaponTrace`: activa el trace en NotifyBegin y lo desactiva en NotifyEnd. Busca AC_Combat por clase, sin depender de una etiqueta de componente.
- `/Game/Component/AC_Health`: `MaxHealth`, `Health`, `bIsDead`, `OnHealthChanged` y `OnDeath`. Ignora daño cero/negativo y daño posterior a la muerte; limita la vida al intervalo 0..MaxHealth. Al morir deshabilita la recepción de daño y la entrada del actor.
- `/Game/Component/BP_TrainingDummy`: blanco con colisión, vida y texto. Se destruye al morir.
- `SandboxCharacter_CMC` y `SandboxCharacter_Mover`: tienen `HealthComponent` y reenvían Event AnyDamage a `HandleOwnerDamage`. El jugador del nivel actual es CMC.

Para un futuro enemigo, añade AC_Health y conecta Event AnyDamage con `HealthComponent.HandleOwnerDamage`, pasando el propio actor, Damage, DamageType, InstigatedBy y DamageCauser. Su colisión debe bloquear Visibility, el canal que usa la espada. Escucha OnDeath para su animación o lógica de muerte. El blanco es una referencia de esta conexión; todavía no tiene IA.

Los valores pueden ajustarse en el componente de cada Blueprint. El componente AC_Combat existente en SandboxCharacter_CMC también tiene sus valores establecidos explícitamente en 25 y (10,10,10).

## Problemas corregidos

1. NotifyBegin enviaba false y nunca activaba el trace.
2. NotifyEnd llamaba a CallSetInputState en lugar de apagar el trace.
3. El resultado del BoxTrace no se utilizaba para aplicar daño.
4. El ataque apagaba el tick necesario para actualizar el trace.
5. No existían vida, recepción de daño ni control de impactos repetidos.

## Verificación

Cinco Blueprints compilaron con warnings tratados como errores. Se ejecutó en PIE de un jugador `/Game/Developers/CombatTests/BP_SwordDamageTest`, con nueve comprobaciones aprobadas:

1. Existen los dos sockets de la espada.
2. Dos traces en el mismo golpe dejan al blanco en 75 de vida.
3. Un trace inactivo no causa daño.
4. La siguiente ventana deja 50 de vida.
5. La tercera ventana deja 25 de vida.
6. El cuarto golpe destruye el blanco.
7. ApplyDamage al jugador reduce su vida a 75.
8. El daño negativo no cura ni reduce vida.
9. El daño letal al jugador deja vida 0 y bIsDead=true.

El registro de ejecución no mostró Accessed None ni Blueprint Runtime Error. El actor de prueba temporal fue retirado del nivel. Para repetir las pruebas, coloca BP_SwordDamageTest en una copia de prueba del nivel y pulsa Play; busca `COMBAT_TEST` en Output Log. Esta prueba aplica daño letal al jugador, por lo que debe retirarse antes de jugar normalmente.

Las copias de los cuatro assets originales modificados están en `Saved/CombatFixBackup`. No se cambiaron las animaciones de ataque ni se añadieron IA enemiga o HUD de vida.

## Verificacion del ataque real

Tambien se corrigio GetInputRot: al atacar sin movimiento conserva la rotacion actual del personaje, en lugar de girarlo hacia yaw 0. Con movimiento utiliza la ultima entrada mundial de movimiento del Pawn.

La prueba manual con clic izquierdo dejo al blanco en 75/100 tras un solo golpe y no produjo errores de ejecucion. Los elementos sin AC_Health siguen bloqueando el trace y no se agregan a la lista de objetivos ignorados.


## Enemigo GASP Ragdoll

BP_EnemyGASP_Ragdoll hereda de SandboxCharacter_Mover_Ragdoll. Conserva el inicio y tick originales de GASP, recibe dano con AC_Health y llama a TriggerRagdoll con InjuryState Limp al morir. Oculta el texto de vida, conserva el cuerpo y anula NPC_GetupCheck para impedir que se levante. El AC_Combat heredado queda sin tick en este enemigo pasivo.

El cilindro fue reemplazado en L_Blockout_Nivel01; BP_TrainingDummy permanece como fixture de las pruebas anteriores. No se modificaron los Blueprints originales de GASP en este cambio.

BP_GASPRagdollTest comprueba 75 de vida tras un golpe, 25 y vivo tras tres, cero y muerto tras cuatro y el perfil Ragdoll con pelvis simulando fisica seis segundos despues. Las cuatro comprobaciones pasaron. El primer impacto de espada tambien se verifico manualmente. El actor temporal de prueba fue retirado.
