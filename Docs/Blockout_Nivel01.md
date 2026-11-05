# Ampliacion del blockout

Nivel: `/Game/Levels/L_Blockout_Nivel01`.

Se uso la referencia suministrada para conservar un recorrido alargado y sinuoso, rodearlo de masas de roca, introducir zonas altas y bajas y ofrecer dos desvios que vuelven al camino principal. Es un blockout jugable; los materiales y rocas son de prototipo.

## Recorrido

La guia ocre marca el camino principal. Las piezas verdes indican apoyos de parkour y las plataformas azuladas distinguen los desvios elevados. Las guias son BP_RouteGuide: desactivan colision en Construction Script y BeginPlay para que no produzcan pequenos bloqueos al caminar.

| Zona | Centro aproximado XY (cm) | Altura del suelo |
| --- | --- | --- |
| Entrada | 9350, 4350 | 0 cm |
| Terraza inicial | 8050, 3900 | 120 cm |
| Patio central | 5250, 2700 | 180 cm |
| Mirador del primer desvio | 6400, 1900 | 600 cm |
| Corredor central | 1700, 2750 | 0 cm |
| Puente del segundo desvio | -769, 3000 | 750 cm |
| Patio oeste | -3200, 1600 | 150 cm |
| Terraza final | -7000, 1300 | 300 cm |
| Mirador de llegada | -8500, 1000 | 450 cm |

El primer desvio sube desde la terraza inicial mediante cuatro escalones de 120 cm y baja por una rampa hacia el patio central. El segundo desvio usa escalones de 150 cm para subir al puente y descender hacia el patio oeste. Bajo el puente permanece un paso para continuar por el suelo.

Hay 23 piezas LevelBlock_Traversable, reutilizando diez del nivel, y nueve rampas. Los bloques conservan las cuatro splines de borde y el perfil TraversalObjectPreset de GASP. Se agregaron 93 rocas de bloque y un portal de llegada; se ensancharon las masas de los muros conservando su cara interior. Las piezas estan organizadas en la carpeta Blockout_Expansion del Outliner.

## Enemigos y navegacion

Hay ocho enemigos GASP distribuidos entre las terrazas inicial, central, oeste y final, el corredor, el mirador de parkour, el puente alto y la llegada. Se redistribuyeron los tres anteriores y se agregaron cinco. Conservan espada, ataques, dano, reaccion y muerte con ragdoll.

Los enemigos usan rampas para navegar. Los tramos que requieren escalar estan pensados para el jugador; no se agregaron animaciones de escalada ni enlaces de navegacion especiales a la IA. El guardia del puente puede patrullar su superficie navegable.

Enemy_NavMeshBounds cubre desde (-10000,-5000,-800) hasta (11000,7000,2800). Recast conserva generacion Dynamic. Puede verse con P en el viewport. En las instancias de los enemigos se uso OnlyTickMontagesAndRefreshBonesWhenPlayingMontages para reducir evaluaciones de animacion fuera de vista y conservar la actualizacion necesaria al atacar.

No se modificaron los grafos ni parametros de locomocion del jugador. Solo se recoloco el PlayerStart dentro del nivel para dejar una entrada libre ante la primera rampa.

## Verificacion

BP_BlockoutValidation paso 50 comprobaciones en PIE: ocho posiciones sobre NavMesh, 34 conexiones consecutivas del recorrido, conexion completa de inicio a meta, seleccion y ejecucion de tres escaladas reales con GASP y paso fisico libre bajo el puente. No aparecieron Blueprint Runtime Error ni Accessed None. El actor de prueba se retiro antes de guardar el nivel.

Para repetir las pruebas, colocar BP_BlockoutValidation temporalmente en una copia del nivel y buscar BLOCKOUT_TEST en Output Log. Durante esa prueba se detiene la IA y se teletransporta al jugador para comprobar las escaladas; no debe quedar colocado al jugar normalmente.

Copias del mapa y actores anteriores a la ampliacion, inventario inicial, manifiesto final y captura de vista general: `Saved/BlockoutExpansionBackup`. La malla y los materiales nuevos estan en `/Game/Levels/BlockoutExpansion`.
