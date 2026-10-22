# Comentarios de los Blueprints (versión final, UE 5.7)

Textos de los comentarios que hay dentro de cada Blueprint (`Content/Blueprints`). Están ya puestos en los grafos; este archivo es solo de consulta.


## BP_Weapon_Base

- AL COGER: guardo el mando que me sujeta y me suscribo al dispatcher OnTriggerInput del pawn (sin Tick).
- AL SOLTAR: me desuscribo del pawn y aviso a OnWeaponDropped (el hacha lo usa para lanzarse).
- FireWeapon (padre): solo dispara si el arma esta cogida y le queda municion. Resta 1 bala. Las hijas hacen el resto (trazo, flecha, lanzamiento).
- StopFireWeapon: vacia en el padre. Se llama al soltar el gatillo de la mano que sujeta el arma. El arco la sobrescribe para disparar la flecha.
- ReloadWeapon: CurrentAmmo = MaxAmmo
- OnWeaponGrabbed: bIsHeld = true. Las armas solo pueden disparar mientras estan cogidas.
- OnWeaponDropped: bIsHeld = false. El hacha la sobrescribe para decidir si es un lanzamiento o solo la sueltas.
- OffHandTrigger(bPressed, Controller): gatillo de la OTRA mano. Vacia en el padre; pistola y escopeta la usan para amartillar y el arco para tensar.
- Se llama por el dispatcher OnTriggerInput del pawn. Mano que sujeta: gatillo = FireWeapon / soltar = StopFireWeapon. La otra mano: OffHandTrigger (amartillar, tensar).

## BP_Pistol

- RECARGA: el cargador (BP_Magazine) toca MagCollision -> se destruye, ReloadWeapon() y hay que volver a amartillar.
- FireWeapon (override): solo si esta amartillada y queda municion -> Parent: FireWeapon (resta bala) -> ShootTrace -> sonido. Sin balas hay que recargar y amartillar otra vez.
- AMARTILLAR (override OffHandTrigger): con la otra mano en la corredera (SlideCollision, < 15 cm) y apretando su gatillo -> bIsCorrectlyCocked = true.
- ShootTrace: LineTraceByChannel desde MuzzleLocation hacia delante (TraceRange = 5000). Si toca una diana (BP_Target_Base) le llama a OnTargetHit. El rayo se dibuja 0.6 s (naranja = recorrido, rojo = impacto) para ver por donde va el disparo.

## BP_Shotgun

- ESCOPETA (hija de BP_Pistol): hereda la recarga (MagCollision) y el amartillado (SlideCollision) de la pistola. Solo sobrescribe ShootTrace.
- ESCOPETA (hija de BP_Pistol): misma recarga y amartillado; solo cambia ShootTrace: PelletCount perdigones con dispersion SpreadAngle. Cada disparo gasta 1 cartucho.

## BP_Bow

- ARCO (hijo de BP_Weapon_Base): toda la logica esta en los overrides OffHandTrigger (tensar con la otra mano) y StopFireWeapon (soltar el gatillo = disparar).
- TENSAR: la otra mano aprieta el gatillo junto a la cuerda (StringCollision) -> aparece una BP_Arrow acoplada al arco y guardo la mano que tira.
- SOLTAR EL GATILLO de la mano que tira -> StopFireWeapon (dispara la flecha).
- StopFireWeapon (override): FireWeapon() gasta 1 flecha; tension = distancia mano-arco / 50, limitada entre 0.2 y 1.0; se desacopla la flecha y LaunchArrow(tension). Limpio SpawnedArrow y PullingHand.

## BP_Axe

- Guardo la posicion inicial en la mesa y arranco un temporizador (cada 0.03 s, sin Tick) que mide a que velocidad se mueve el hacha.
- INCRUSTACION: si el hacha lanzada choca con una diana, apago las fisicas (se queda clavada) y llamo a OnTargetHit(). Si choca con otra cosa, rebota.
- ReturnHome: si nadie la tiene cogida, el hacha vuelve a su sitio de la mesa.
- LANZAR o SOLTAR (override OnWeaponDropped): uso la velocidad medida por TrackVelocity justo antes de soltar. Si supera ThrowMinSpeed (150 cm/s) es un lanzamiento: gasta 1, fisicas on, sale con esa velocidad x ThrowMultiplier y girando hacia delante (SpinSpeed). Si va mas lento solo la dejo caer. En los dos casos vuelve a la mesa a los ReturnDelay segundos.
- TrackVelocity: velocidad = (posicion ahora - posicion anterior) / 0.03 s. Mientras la llevo en la mano es la velocidad de mi mano.

## BP_Arrow

- La flecha lanzada toca una diana -> OnTargetHit() y la flecha se destruye.
- LaunchArrow(Tension): activa el ProjectileMovement con velocidad ArrowSpeed * Tension. Se autodestruye a los 6 s.

## BP_Magazine

- CARGADOR: solo fisicas + GrabComponent. La recarga la hace la pistola o la escopeta cuando el cargador entra en su MagCollision.

## BP_Target_Base

- OnTargetHit: la llaman las armas. Grito por el dispatcher OnTargetDestroyed(Puntos, Tiempo) y me destruyo. No busco a ningun manager. bWasHit evita contar dos veces (perdigones).
- Sin Tick: el ProjectileMovement (gravedad 0) mueve la entidad en linea recta hacia su eje frontal a MoveSpeed. Si nadie le dispara se autodestruye a los 8 s.

## BP_Target_Enemy

- ENEMIGO (hijo de BP_Target_Base): sin logica propia, solo cambia valores por defecto: PointValue = 100, TimePenalty = 0, MoveSpeed = 350 y la malla CuteCreature.

## BP_Target_Civilian

- CIVIL (hijo de BP_Target_Base): sin logica propia. PointValue = -200, TimePenalty = -5 s, MoveSpeed = 450 y la malla TinyHero. Si le disparas pierdes puntos y tiempo.

## BP_Target_Spawner

- SPAWNER: no hace nada por si solo. El GameManager le llama a SpawnEntity cada segundo de ronda; bSpawnsCivilians decide si saca civiles o enemigos.
- SpawnEntity (lo llama el Manager): crea un civil o un enemigo segun bSpawnsCivilians, en la flecha del spawner con un desplazamiento lateral aleatorio, y avisa con OnEntitySpawned(diana).

## BP_GameManager_Screen

- BEGINPLAY: busco los spawners y me suscribo a su OnEntitySpawned. Guardo las armas para rellenar arco y hacha. Empiezo en pausa (10 s) y arranco el reloj maestro (timer de 1 s, sin Tick).
- UpdateScreenTexts: el propio Manager actualiza el Widget de la pantalla (SCORE x / umbral, TIME, WAVE, HIGH SCORE y estado).
- FinishGame: paro el reloj y actualizo el High Score (solo en memoria, no se guarda al cerrar el juego).
- ModifyScoreAndTime: suma los puntos de la diana y los segundos de penalizacion (civil = -5 s). Solo cuenta durante la ronda.
- HandleTargetDestroyed(Points, Time): llega por el dispatcher OnTargetDestroyed de cada diana y pasa los valores a ModifyScoreAndTime.
- Cuando un spawner crea una diana, me suscribo a SU dispatcher OnTargetDestroyed.
- StartRound: 45 s de ronda. Aparece 1 cargador por cada arma recargable (pistola y escopeta) y el arco y el hacha (las que no son pistola) recuperan sus 30 usos.
- EndRound: si llega al umbral y era la ronda 10 -> HAS GANADO. Si llega y no es la 10 -> ronda + 1, umbral + 500 y otra pausa de 10 s. Si no llega -> GAME OVER.
- MasterClockTick (cada 1 s): resta un segundo. En ronda pide una entidad a un spawner al azar. Al llegar a 0: si era pausa empieza la ronda; si era ronda, EndRound.

## BP_ShootingGameMode

- GAME MODE (hijo de BP_XRGameMode): solo pone BP_RangePawn como Default Pawn. No tiene logica.

## BP_RangePawn

- PAWN VR: al ser poseído añade los Input Mapping Context de ambas manos. Los gatillos (IA_Shoot_Left / IA_Shoot_Right, al pulsar y al soltar) llaman a HandleTrigger. El Pawn no sabe qué arma lleva: sólo emite el evento de input.
- INPUT -> EVENT DISPATCHER: emite OnTriggerInput(mano izquierda?, pulsado?, mando de esa mano). Las armas cogidas se suscriben a este dispatcher; el Pawn nunca llama directamente a un arma.

## WBP_BowlingUI

- El widget no tiene logica propia: el BP_GameManager_Screen le llama a SetScore, SetTime, SetRound y SetStatus cuando cambia algo.
- SetStatus: texto de estado de la pantalla (DISPARA, PAUSA, GAME OVER o HAS GANADO).
- SetTime: pasa los segundos que quedan a formato m:ss y los pone en TimeText.
- SetScore: SCORE x / umbral y HIGH SCORE.
- SetRound: escribe 'WAVE: RONDA n' en RoundText.
