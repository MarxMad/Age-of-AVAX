# El juego

RTS en 3D (Three.js) con la dinámica clásica de Age of Empires: recolectar, construir, avanzar de era y destruir el centro urbano enemigo. El catálogo completo (costos, vida, ataque y tiempos) vive en `game/src/sim/defs.ts` y es la única fuente de verdad.

## Simulación determinista

- Tick fijo de **30 Hz** y RNG con semilla, sin DOM ni render (`src/sim`).
- El jugador solo modifica la partida con **comandos** (`train`, `rally`, `build`, `order`…). `seed + log de comandos` reproduce exactamente la partida, y el verificador lo usa para firmar resultados.
- `SIM_VERSION` (hoy **0.24.0**) debe coincidir entre el cliente y el verificador: si no, el verificador rechaza la victoria con "versión de reglas distinta". Súbela cada vez que cambien las reglas y, al publicar, actualiza el servidor en el mismo momento (en la PC: `cd ~/age10 && git pull && sudo systemctl restart age-servidor`). `/api/health` muestra la versión que corre el servidor (`simVersion`).

## Eras

| Era | Nombre | Desbloquea |
|---|---|---|
| I | Antigua | aldeanos, milicia, casas, molino, aserradero, mina, granja, cuartel, mercado, empalizada |
| II | Clásica | arqueros, exploradores, galería de tiro, establo, herrería, torres, muralla |
| III | Medieval | caballeros, catapultas, taller de asedio, castillo |
| IV | Pólvora | mosqueteros, cañones, universidad |
| V | Industrial | fusileros, fábrica |
| VI | Moderna | tanques |

## Civilizaciones

| Civilización | Bonificación | Cómo se obtiene |
|---|---|---|
| **Pioneros de Avalanche** | ninguna (identidad Avalanche) | gratis |
| Nórdicos del Glaciar | +15% tala, aldeanos +10% velocidad | NFT 201 |
| Imperio Subnet | +15% oro y piedra | NFT 202 |
| Guardianes de la C-Chain | edificios +25% vida, defensas +2 ataque | NFT 203 |
| Jinetes de la P-Chain | caballería +15% ataque, +10% velocidad | NFT 204 |
| Arqueros de la X-Chain | +30 alcance a distancia | NFT 205 |

### Pioneros de Avalanche: el rebranding

Misma mecánica de AoE, con nombres del ecosistema Avalanche (`AVALANCHE_NAMES` en `defs.ts`):

| Clásico | Avalanche |
|---|---|
| Aldeano | Validador |
| Milicia | Centinela |
| Arquero | Tirador Snowball |
| Explorador | Explorador Snowflake |
| Caballero | Caballero Avalanche |
| Catapulta | Catapulta Glaciar |
| Mosquetero | Mosquetero C-Chain |
| Cañón | Cañón Subnet |
| Fusilero | Fusilero Warp |
| Tanque | Tanque Avalanche9000 |
| Centro urbano | Red Primaria |
| Casa | Refugio Alpino |
| Molino | Granero del Nodo |
| Aserradero | Aserradero Firewood |
| Campamento minero | Mina de AVAX |
| Granja | Granja de Staking |
| Cuartel | Cuartel del Consenso |
| Galería de tiro | Galería Snowball |
| Establo | Establo Snowflake |
| Herrería | Forja HyperSDK |
| Torre | Torre Validadora |
| Taller de asedio | Taller de Subnets |
| Universidad | Academia Team1 |
| Fábrica | Fábrica Avalanche9000 |
| Castillo | Fortaleza L1 |
| Mercado | Mercado DeFi |
| Empalizada / Muralla | Empalizada de Hielo / Muralla Glaciar |

Para rebrandear otra civilización basta con agregarle `names` en `CIVS`.

### Evolución por era

Como en AoE, cada avance de era cambia el aspecto de los edificios. Al subir de era, el juego anuncia los edificios, unidades y mejoras nuevos.

| Edificio | Cómo evoluciona |
|---|---|
| Centro urbano (Red Primaria) | Conserva la silueta del logo de Avalanche y cambia de fachada y de recinto alrededor de la plaza triangular: **I** arenisca con empalizada y antorchas · **II** mármol con columnata, obeliscos y remate dorado · **III** piedra con muralla almenada y torres de pizarra en las tres puntas · **IV** ladrillo con torreones de cúpula de cobre, reloj y cañones · **V** ladrillo industrial con reja de hierro, tres chimeneas con humo y tanque de agua · **VI** blanco y vidrio con muro de cristal, franja LED, aguja con luz roja y paneles solares |
| Casas, torre, muralla, castillo | Materiales y techos de cada época (madera → piedra → ladrillo → concreto y vidrio) |
| Molino, aserradero, campamento minero, cuartel, galería de tiro, establo, herrería, taller y mercado | Kit de era (`eraKit` en `models.ts`), ver la tabla siguiente. Los de producción también cambian el piso: tierra → empedrado → asfalto → concreto, con bordillos desde la III |

| Era | Muros | Techos | Elemento icónico |
|---|---|---|---|
| I · Antigua | madera y yeso | color del equipo | — |
| II · Clásica | mármol blanco | color del equipo | dos columnas clásicas al frente y remate dorado en el techo |
| III · Medieval | piedra | pizarra azul | torreón de piedra con almenas |
| IV · Pólvora | ladrillo rojo | cobre verde | cúpula de cobre con linterna y faroles |
| V · Industrial | ladrillo oscuro | zinc | chimenea industrial de ladrillo con humo |
| VI · Moderna | concreto claro | blanco | paneles solares sobre el techo y antena con luz roja |
| VII · Futurista | cerámica blanca, grafito y pizarra | vidrio oscuro | aristas de neón, franja de luz y anillo holográfico; la Red Primaria estilo Oblivion (`makeFutureTown`) |

Desde la era III, el color del equipo deja los techos y pasa a un borde alrededor del edificio y a un estandarte, como en AoE. La comparación está en `docs/img/eras.jpg` (`?photo=stable&era=N&zoom=7.5`).

Las capturas del README salen del modo `?photo=<edificio|unidad>&era=N`.

### Identidad visual

- El **centro urbano** tiene la silueta del logo de Avalanche: el triángulo grande con el corte diagonal y el triángulo pequeño, sobre una plaza triangular roja.
- La **universidad / Academia Team1** tiene la forma de la marca "t1" de Team1: la t blanca y el 1 rojo, sobre una plataforma morada de LatAm.
- **Siluetas distintas desde arriba:** el aserradero tiene techo de tablones y troncos; el campamento minero, rocas con túnel y una torre con polea; el cuartel, techo de pabellón con torre y patio de arena; el taller de asedio, techo curvo de lona a rayas y una grúa. Para compararlos: `?showcase=1&focus=compare`.
- El **cursor** es un triángulo rojo de Avalanche con la marca blanca (`public/cursor.svg`, con `cursor.png` de respaldo).
- Las **casas** usan los colores de los chapters de Team1 (Global, Africa, Brazil, India, Korea, LatAm, Thailand, Turkiye, United States y Vietnam). Una franja del color del equipo identifica al dueño.

## Punto de reunión y actividad asignada

- **Punto de reunión:** con un edificio que entrena seleccionado, haz clic derecho en el mapa, o pulsa el botón de bandera (B) y luego clic. Se ve una bandera con una línea punteada.
  - **Sobre un recurso:** los aldeanos lo recolectan.
  - **Sobre una obra propia:** la construyen o reparan.
  - **Sobre un enemigo:** los militares lo atacan.
  - **En el suelo:** las unidades van ahí.
- **Actividad antes de nacer:** como en AoE, hay un solo botón de aldeano. Pon el punto de reunión sobre un árbol, una mina, una cantera, bayas o una granja, y cada aldeano nuevo nace con esa actividad. Se reparten entre los recursos iguales de alrededor (no todos en el mismo árbol). Si el recurso marcado se agota, toman otro igual cerca. El panel del edificio indica la actividad ("Aldeanos nuevos: madera").

## Edificios: qué fabrican y qué mejoran

Nombres de la civilización Avalanche (entre paréntesis, el nombre genérico). La era es desde cuándo se puede construir o
entrenar. Los costos de cada mejora están en el botón del juego (también se listan aquí).

### Economía

| Edificio | Era | Qué entrena | Qué recibe | Mejoras que investiga |
|----------|-----|-------------|------------|-----------------------|
| **Red Primaria** (centro urbano) | I | Validador (aldeano) · Sabio Snowman (monje, era II) | Comida, madera, oro y piedra | **Avanzar de era** (I→VII) · **Carretilla** (II): aldeanos cargan +5 y caminan 10 % más rápido · **Almacenes** (III): +50 % de capacidad de almacén · **Mampostería** (III): todos tus edificios +20 % de vida · **Logística** (VI): aldeanos cargan +10 y caminan 15 % más rápido |
| **Refugio Alpino** (casa) | I | — | — | — (da +5 de población) |
| **Granero del Nodo** (molino) | I | — | Comida | **Arado** (II): granjas +25 % · **Collera** (III): granjas +20 % · **Rotación de cultivos** (V): granjas +25 % |
| **Granja de Staking** (granja) | I | — | — | — (fuente renovable de comida; se resiembra con 60 de madera) |
| **Aserradero Firewood** (aserradero) | I | — | Madera | **Hacha doble** (I): tala +20 % · **Sierra de arco** (III): tala +15 % |
| **Mina de AVAX** (campamento minero) | I | — | Oro y piedra | **Pico minero** (II): oro y piedra +20 % · **Minería profunda** (IV): oro y piedra +15 % |
| **Mercado DeFi** (mercado) | I | Caravana DeFi (carreta comerciante, era II) | — | Compra y vende comida, madera y piedra por oro (los precios suben al comprar y bajan al vender) |
| **Puerto Teleporter** (muelle) | I | Pesquero Coreth (I) · Transbordador ICM (I) · Carguero Interchain (II) · Galera Avalanche (II) | Pescado (comida) | — |
| **Academia Team1** (universidad) | IV | — | — | **Balística** (IV): +40 de alcance a distancia · **Química** (V): +3 de ataque a distancia · **Guiado de precisión** (VI): asedio y blindados +20 % de ataque y +40 de alcance |
| **Templo del Consenso** (maravilla) | IV | — | — | — (si resiste 5 minutos terminada, ganas) |

### Militares

| Edificio | Era | Qué entrena | Mejoras que investiga |
|----------|-----|-------------|-----------------------|
| **Cuartel del Consenso** (cuartel) | I | Centinela (milicia, I) · Mosquetero C-Chain (IV) · Fusilero Warp (V) · Mecha Validador (robot, VII) | — |
| **Galería Snowball** (galería de tiro) | II | Tirador Snowball (arquero) | — |
| **Establo Snowflake** (establo) | II | Explorador Snowflake (II) · Caballero Avalanche (III) | — |
| **Forja HyperSDK** (herrería) | II | — | **Forja** (II): +2 de ataque cuerpo a cuerpo · **Armadura de placas** (III): +2 de armadura · **Altos hornos** (IV): +2 de ataque cuerpo a cuerpo · **Blindaje** (V): +2 de armadura |
| **Taller de Subnets** (taller de asedio) | III | Catapulta Glaciar (III) · Cañón Subnet (IV) | — |
| **Fortaleza L1** (castillo) | III | Caballero Avalanche · Catapulta Glaciar | — (dispara sola y da +20 de población) |
| **Fábrica Avalanche9000** (fábrica) | V | Cañón Subnet (IV) · Tanque Avalanche9000 (VI) · Halcón Interchain (helicóptero, VII) · Tanque de Plasma Etna (VII) | **Cadena de montaje** (V): todos tus edificios entrenan 25 % más rápido |
| **Torre Validadora** (torre) | II | — | — (dispara sola a los enemigos cercanos) |
| **Empalizada de Hielo**, **Muralla Glaciar** y **Puerta Glaciar** | I / II / I | — | — (se trazan arrastrando; la puerta deja pasar solo a tus unidades) |

En el juego, cada mejora se ve como una insignia: el dibujo a color dice **qué** mejora (trigo, madera, oro y piedra,
espada, escudo, mira…) y la etiqueta verde de la esquina dice **cuánto** (+25 %, +2, +40).

## Dificultad de la IA

Se elige en el lobby, sobre el botón Jugar, y el juego la recuerda para la próxima partida.

| Dificultad | Recursos de la IA | Avance de era | Ejército | Primer ataque | GLORY |
|------------|-------------------|---------------|----------|---------------|-------|
| **Fácil** | ×0.55 | 40 % más lento | ×0.6, entrena cada 14 s | 5:00 | No (práctica) |
| **Normal** | ×1 | como siempre | ×1, cada 10 s | 3:00 | Sí |
| **Difícil** | ×1.5 | 15 % más rápido | ×1.4, cada 8 s | 2:30 | Sí |
| **Experto** | ×2.1 | 30 % más rápido | ×1.9, cada 6 s | 2:00 | Sí |

El primer ataque es cuando la oleada sale de su base; con el mapa de 108 × 108 tarda cerca de un minuto en llegar a la
tuya (en Normal, hacia el minuto 3:50). Las oleadas van contra **tu edificio más cercano**: antes iban contra lo más
cercano tuyo, y si era un aldeano o el explorador a más de 1.100 px la orden se cancelaba sola y la oleada no salía.
La dificultad viaja en el replay y el verificador la re-simula igual. Fácil no cobra GLORY: si cobrara, se podría farmear contra una IA débil.

## Casas

El Refugio Alpino cabe en una casilla, así que las casas **se construyen seguidas**, pegadas una a otra, sin encimarse ni colgar sobre el agua. El aserradero y el campamento minero también ocupan una sola casilla y miden como una casa.

## Granjas

- **Tamaño:** el modelo llena exactamente su terreno de 3×3 casillas (120 px), igual que la vista previa de construcción.
- **Granjas pegadas:** se construyen una junto a otra, como las casas, y forman una parcela continua. Shift + clic coloca varias seguidas.
- **Un aldeano por granja:** el primero que empieza a trabajarla se queda con ella (aunque vaya a descargar al molino). Si mandas a varios a la misma granja, los demás buscan la granja libre más cercana; si no hay ninguna, quedan inactivos y el panel de eventos avisa.
- **Tres cultivos:** trigo, maíz o calabazas. Cada granja toma uno al azar y, cada vez que se resiembra, puede cambiar a otro. Es solo visual: los tres dan la misma comida.
- **Surcos:** el cultivo va por surcos (una malla por surco, o dos en el maíz y las calabazas) y se vacía surco a surco al cosecharse. Así cuesta unas 10–20 llamadas de dibujo por granja, no una por planta.
- **Cambian con la era (sin tapar el cultivo):**
  - **Eras I–II:** cerca baja de madera y espantapájaros.
  - **Eras III–IV:** muro bajo de piedra y pacas de heno.
  - **Eras V–VI:** cerca metálica, canal de riego al centro y tanque de agua.
  - **Era VII:** bordes de luz cian y un dron de riego.

Cuando una granja se agota, se vuelve a sembrar sola cobrando su costo, **60 de madera** (como la cola de resiembra del molino en AoE). El aldeano sigue trabajándola y el panel de eventos avisa "Granja resembrada". Si no hay madera suficiente, la granja desaparece y el aviso dice cuánta falta.

## Movimiento

- **Obstáculos pequeños:** árboles, minas y bayas bloquean solo su casilla (antes, 3×3). Los aldeanos caminan entre los árboles y no rodean un perímetro enorme.
- **Edificios propios:** se cruzan, salvo murallas, empalizadas, torres y castillos. Los del rival bloquean.
- **Rutas suavizadas:** la unidad camina en línea recta mientras el terreno lo permite, en vez de ir en zigzag de casilla en casilla.
- **Cada aldeano llega a su propio lugar:** se acerca al recurso, a la obra o al almacén por el lado que le queda, con un desvío propio, en vez de pelear por el centro.
- **Nadie se traba:** quien camina aparta a los que están quietos y sigue. Los que trabajan ceden poco y los ociosos se apartan del todo. Un aldeano que ya trabaja tiene 12 px de tolerancia: un empujón no lo hace volver a caminar.
- **Al construir un edificio económico,** quien lo termina se pone a trabajar ahí:
  - granja: la cultiva;
  - molino: bayas o caza cerca;
  - aserradero: el árbol más cercano;
  - campamento minero: la mina o cantera con menos aldeanos.
- **Cambio de recurso:** si un aldeano lleva madera y lo mandas al oro, va directo a la mina y suelta la madera al empezar (como en *Age of Empires*). Antes caminaba primero a descargar al almacén y parecía ignorar la orden.
- **Prueba:** `test/movement.test.ts` hace trabajar a 20 aldeanos durante 3 minutos y verifica que ninguno pase más de 2 s sin avanzar ni trabajar. Con el código anterior, alguno quedaba trabado 55 s.

## Edad Futurista (VII)

- **Avance:** 2000 comida, 1600 oro y 800 piedra en la Red Primaria, desde la Edad Moderna. La IA llega a los 24 minutos.
- **Unidades:** robot de combate (Cuartel), helicóptero de ataque y tanque de plasma (Fábrica). Disparan láser (`ranged: 'laser'`): un haz instantáneo cian (tuyo) o magenta (rival) con destellos; el del tanque es más grueso y enciende el blanco.
- **Aeronaves** (`air: true`): vuelan en línea recta sobre agua, bosques y murallas, sin pathfinding. Solo se separan entre ellas, no empujan a la infantería de abajo. Cualquiera las puede atacar.
- **Aspecto** (`src/render/models.ts`): los edificios parten del modelo de la Edad Moderna y `futurize` los rehace (cerámica, grafito, pisos de pizarra, aristas de neón con `EdgesGeometry`, franja de luz y anillo holográfico). El centro urbano es `makeFutureTown`: estructura blanca estilo *Oblivion*, más grande, con vidrio negro en las caras del logo, LED en todas sus aristas, postes de luz y torre con helipuerto. Las personas llevan casco con visor cian y peto blanco.

## Murallas y puertas

- **Bloqueo real:** empalizadas, murallas, torres y castillos bloquean la casilla para todos (`HARD_BUILDINGS`). Antes, si no había camino (por ejemplo, un recinto cerrado), la unidad caminaba en línea recta hacia el destino y atravesaba el muro.
- **Destino inalcanzable:** `NavGrid.find(..., nearest)` devuelve la ruta a la casilla alcanzable más cercana al destino. La ruta parcial se conserva y se recalcula cada 3 s, por si se abre paso (una puerta o un muro destruido).
- **Bolsillos del bosque:** si una unidad queda atrapada en menos de 40 casillas (por ejemplo, empujada entre árboles), puede cruzar recursos para salir, nunca muros (`NavGrid.res`).
- **Puerta (`gate`):** se coloca sobre un tramo propio, que reemplaza, o en un hueco libre.
  - En el mapa de navegación es una casilla blanda del dueño: sus unidades la cruzan y las enemigas no.
  - El renderer abre las hojas cuando hay una unidad propia cerca.
  - Es de troncos junto a empalizadas y de piedra junto a murallas (bit 8 de la máscara de muro).
- **Asedio de murallas:** una unidad militar con orden de ataque cuya ruta queda parcial ataca el muro, la puerta o la torre enemiga más cercana (`blockerNear`). Al destruirlo, retoma su orden original (`resume`) y entra por la brecha.
- **Pruebas:** `test/walls.test.ts`:
  - recinto cerrado de empalizada y de muralla;
  - las unidades propias pasan por la puerta;
  - las enemigas no la cruzan;
  - la puerta solo se coloca sobre un tramo propio;
  - un ejército abre una brecha.

## Mapa y fauna

- **Tamaño:** 108 × 108 casillas de 40 px (4.320 px por lado), un 50 % más por lado que el mapa original de 72 × 72.
  Los centros urbanos están en la diagonal, a 18 casillas de su esquina, unos 4.000 px uno del otro.
- **Simetría:** el terreno es simétrico respecto del centro y todo lo que rodea a cada base (bosques, vetas, bayas,
  ganado y jabalíes) se coloca en espejo, casilla por casilla. Lo que se reparte al azar por el mapa (vetas sueltas,
  bosques, manadas y lobos) queda a más de 650–1.000 px de ambas bases, para que ninguna tenga ventaja.
- **Cantidades:** bosques, vetas, bayas, fauna, peces y la decoración del terreno se escalan con el área (×2,25), así que
  la densidad es la misma que antes.

### Oro y piedra en vetas

Como en *Age of Empires II*, el oro y la piedra vienen en **vetas de varias piezas pegadas**, una por casilla y en dos
filas, así cada pieza toca una casilla libre y la pueden trabajar varios aldeanos a la vez. Al agotarse una pieza, el
aldeano pasa sola a la de al lado (y el campamento minero manda a sus aldeanos a la pieza con menos gente).

| | Por pieza | Junto a cada base | Total por base |
|---|---|---|---|
| Oro | 400 | 2 vetas: 5 y 4 piezas | **3.600** (antes 1.600: dos minas de 800) |
| Piedra | 350 | 1 veta de 4 piezas | **1.400** (antes 600: una cantera) |

- **Centro del mapa:** una veta de oro de 2 × 2 justo en el centro y dos de piedra a los lados, a la misma distancia de
  ambos jugadores.
- **Resto del mapa:** vetas sueltas de 3 o 4 piezas a más de 900 px de las bases, para que valga la pena expandirse.
- **Desierto:** una veta extra de oro y otra de piedra en cada base. **Montañas y Desierto** tienen además vetas extra
  repartidas.
- Las piezas de oro son roca ocre con pepitas doradas; las de piedra, bloques claros. Ambas se van vaciando al recolectar.

- **Recursos por mapa** (semilla 7; oro y piedra en piezas):

| Mapa | Árboles | Oro · piedra | Bayas | Ganado (ovejas + vacas) | Ciervos | Jabalíes | Lobos | Peces |
|---|---|---|---|---|---|---|---|---|
| Tierra firme | 654 | 65 · 63 | 28 | 30 | 31 | 6 | 8 | 84 |
| Ríos | 750 | 69 · 60 | 36 | 24 | 27 | 11 | 13 | 30 |
| Islas | 243 | 54 · 53 | 17 | 20 | 15 | 7 | 5 | 98 |
| Costa | 634 | 64 · 58 | 29 | 31 | 24 | 8 | 17 | 86 |
| Desierto | 357 | 77 · 79 | 25 | 27 | 27 | 12 | 10 | 70 |
| Montañas | 672 | 88 · 89 | 39 | 28 | 28 | 11 | 12 | 0 |
| Nieve | 603 | 61 · 59 | 24 | 23 | 26 | 10 | 10 | 0 |

- **Cada base:**
  - tres bosques;
  - dos arbustos de bayas;
  - ocho ovejas y dos vacas;
  - dos jabalíes cerca;
  - dos vetas de oro y una de piedra.

| Animal | Comida | Comportamiento |
|---|---|---|
| Oveja | 100 | ganado: pasta cerca y se recolecta rápido |
| Vaca | 180 | ganado lento y robusto |
| Ciervo | 140 | salvaje, huye pastando; se caza al instante |
| Jabalí | 320 | **se defiende**: 70 de vida y 3 de ataque. El aldeano que lo caza lo ataca con su lanza (doble daño) hasta abatirlo y luego lo recolecta. Mejor entre dos. |
| Lobo | — | **depredador**: 45 de vida y 4 de ataque. Ataca a cualquier unidad a menos de 130 px y la persigue. Aparece en manadas, a más de 1.000 px de las bases (más en la nieve). Los militares ociosos lo atacan solos. |

- **Los aldeanos se defienden** de un animal que los ataca y, al terminar, retoman su tarea.
- **Clic derecho sobre un lobo** ordena atacarlo.
- **Clic derecho sobre un jabalí:** los aldeanos lo cazan y los militares lo atacan.

## Animación

- **Pasos y movimiento:** el ritmo del paso depende de la distancia recorrida, no del tiempo; a pie da unos 2,4 pasos por segundo. Se calcula sobre la posición suavizada que se dibuja: antes salía de la simulación (30 Hz), y a 60 fps brazos y piernas parpadeaban entre balanceo y reposo.
- **Paso y reposo:** al arrancar o detenerse, la amplitud del paso se mezcla suavemente.
- **Golpes de trabajo** (talar, picar, construir): a ritmo humano.
- **Animales:** mueven las patas en diagonal.
- **Combate:**
  - la milicia hace un tajo con carga y recuperación (el cuerpo gira con el golpe);
  - la caballería da estocadas con la lanza;
  - el arquero tensa la cuerda poco a poco y la suelta con cada flecha;
  - mosquetes y rifles retroceden al disparar.
- **Reposo:** las unidades quietas miran a su alrededor de vez en cuando.
- **Salida de escena:** al morir, las personas caen y se hunden en el suelo; los jinetes y el asedio caen de lado; los barcos escoran y se hunden; los edificios se derrumban; los árboles talados caen. El renderer guarda la vista unos segundos después de que la entidad deja la simulación (`Dying`).
- **Equipo por era:** las unidades se reconstruyen al cambiar de era (la era es parte de la clave del modelo). Ver la tabla de *Personajes por era* en el README.

## Efectos

Partículas en `src/render/fx.ts`: tres mallas instanciadas con un búfer circular (astillas y escombros opacos, polvo y humo translúcidos, chispas y fuego aditivos). Cuando un búfer se llena se reutiliza la partícula más vieja, así que la memoria y el costo no crecen. Los efectos continuos (polvo, estela, humo) solo se emiten cerca de la cámara.

| Efecto | Cuándo |
|---|---|
| Astillas, chispas de oro, lascas, tierra, polvo | en el instante de cada golpe de trabajo (talar, minar, picar, cultivar, construir) |
| Polvo al caminar | cada ~0,5 casillas a pie, ~0,3 a caballo o con asedio; color según el bioma |
| Estela de espuma | detrás de los barcos en movimiento; salpicaduras al pescar |
| Chispas y destello | en cada golpe (`hit`) y al llegar cada proyectil |
| Rastro y humo | flechas con rastro; humo al disparar mosquetes y cañones; explosión al impactar el asedio |
| Humo / fuego | edificios por debajo del 60 % / 30 % de vida |
| Columna de Avalanche | al avanzar de era (evento `era`): columna roja con el triángulo, onda expansiva y estallido |
| Portal Teleporter | al salir un barco del muelle (evento `spawn` naval) |
| Aura y curación | anillo del Sabio Snowman; destellos sobre la unidad que recupera vida |
| Nieve | partículas que caen alrededor de la cámara en el mapa Nieve |

## Sabio Snowman y Maravilla

- **Sabio Snowman (monje, `healer`):** era II, 100 de oro, en la Red Primaria. Cada medio segundo cura `HEAL.perSec × 0,5` de vida a cada aliado herido a menos de `HEAL.radius` (120 px), sin contarse a sí mismo y sin barcos ni asedio. Es civil: no ataca ni responde ataques, y una orden de ataque se convierte en moverse hasta el objetivo. La IA no lo entrena.
- **Templo del Consenso (maravilla, `wonder`):** era IV, 1000 de madera, oro y piedra, 5×5 casillas. Al terminarse guarda `wonderAt` y empieza la cuenta atrás de `WONDER_SECS` (300 s). Avisa a ambos jugadores al terminarse y cada minuto. Si sigue en pie al llegar a cero, su dueño gana. La cuenta atrás se ve sobre el templo para los dos jugadores. La IA manda todo su ejército contra una maravilla del jugador.
- **Pruebas:** `test/avalanche.test.ts`.

## Agua y barcos

| Elemento | Era | Costo | Detalle |
|---|---|---|---|
| Muelle (Puerto Teleporter) | I | 150 madera | Sobre agua libre y tocando tierra firme (`canPlace('dock')`). Punto de entrega de comida, solo para barcos. Entrena los cuatro barcos. |
| Bote pesquero (Pesquero Coreth) | I | 75 madera | 60 de vida, carga 15. Solo recolecta peces y descarga en el muelle. |
| Transporte (Transbordador ICM) | I | 125 madera | 160 de vida, 5 pasajeros. Órdenes `board` (subir) y `unload` (navega a la casilla de agua con tierra al lado más cercana al punto y baja a los pasajeros en tierra). |
| Mercante (Carguero Interchain) | II | 100 madera, 50 oro | Orden `trade`: va y vuelve entre el muelle propio más cercano y el elegido. Cada llegada paga `distancia entre muelles × 0,025` de oro. |
| Galera de ataque (Galera Avalanche) | II | 140 madera, 50 oro | 200 de vida, 9 de ataque a 190 px, ×1,5 contra edificios. |
| Banco de peces | — | — | 225 de comida, ritmo 1,0. |

- **Navegación:** los barcos usan un segundo mapa de navegación (`waterNav`) con agua profunda y poco profunda; los vados son de ambos. Los muelles propios se cruzan, como los edificios propios en tierra.
- **Peces:** en espejo para ambos bandos: un banco cerca de cada base (el agua más próxima a menos de ~900 px) y unos 40 repartidos.
- **Pasajeros:** van "a bordo" (`aboard`). No se dibujan, no actúan y no se pueden atacar, pero cuentan para la población. Si el transporte se hunde, se hunden con él.
- **Pruebas:** `test/naval.test.ts` cubre:
  - colocación y construcción del muelle;
  - pesca;
  - transporte entre islas;
  - comercio;
  - combate naval.

## Sumideros de recursos

Para que los recursos no se estanquen en la partida larga:

1. **Mejoras escalonadas por era.** Cada era abre un nivel más caro:

| Era | Tecnología | Edificio | Efecto |
|---|---|---|---|
| III | Collera | Molino | granjas +20% |
| III | Sierra de arco | Aserradero | tala +15% |
| III | Almacenes | Centro urbano | almacén +50% |
| III | Mampostería | Centro urbano | edificios +20% vida |
| IV | Minería profunda | Mina | oro y piedra +15% |
| IV | Altos hornos | Herrería | cuerpo a cuerpo +2 ataque |
| V | Rotación de cultivos | Molino | granjas +25% |
| V | Blindaje | Herrería | militares +2 armadura |
| V | Cadena de montaje | Fábrica | entrenamiento 25% más rápido |
| VI | Logística | Centro urbano | aldeanos +10 de carga y +15% velocidad |
| VI | Guiado de precisión | Universidad | asedio y blindados +20% ataque, +40 alcance |

2. **Reparar cuesta:** una reparación completa vale el 50% del costo del edificio (`REPAIR_COST`). Si no alcanza, el aldeano se detiene y avisa.
3. **Almacén con tope:** por recurso, `800 + 400 × (era − 1) + 150` por cada punto de entrega propio que lo reciba, ×1,5 con Almacenes. El excedente se **merma un 4% por minuto**. En la barra superior el recurso se pinta ámbar y el tooltip lo explica. Aplica también a la IA.

## Indicaciones visuales

| Situación | Indicación |
|---|---|
| Unidad en entrenamiento (cola de hasta **10** por edificio) | cartel flotante sobre el edificio con el nombre, progreso y `+N` en cola; destello verde al nacer |
| Población llena | cartel rojo "¡Población llena!" sobre el edificio, aviso en el panel de eventos y contador de población parpadeando |
| Faltan recursos | el recurso que falta parpadea en rojo y el aviso dice cuánto falta ("Te falta: 20 de madera") |
| Recurso agotado sin otro cerca | aviso "¡Se agotó … aldeanos inactivos!" y botón de aldeano inactivo resaltado |
| Almacén lleno | recurso en ámbar con explicación |

## Rendimiento

- Las piezas estáticas de cada modelo (edificios, árboles, recursos) se **fusionan en una malla por material** (`mergeStatic`): de ~1.800 a ~590 llamadas de dibujo por cuadro.
- Máximo **60 fps** (en pantallas de 120 Hz el navegador pedía el doble), resolución limitada a 1,5x en Retina y sombras recalculadas cada 2 cuadros.
- **Modo ahorro** (botón "Ahorro", se recuerda por navegador): 30 fps, resolución 1x, mapa de sombras de 1024 y sombras cada 4 cuadros.
- El terreno del minimapa se pinta una vez y se reutiliza; el minimapa se refresca 4 veces por segundo.

## Controles

| Acción | Control |
|---|---|
| Seleccionar | clic · arrastrar para varios · doble clic para todos los del tipo |
| Ordenar / punto de reunión | clic derecho o clic con dos dedos |
| Cámara | mouse en cualquier borde de la ventana (como en AoE; más rápido cuanto más pegado) · dos dedos en el trackpad · flechas · pellizcar o rueda para el zoom |
| Panel de comandos | Q W E R T / A S D F G / Z X C V B |
| Centro urbano / aldeano inactivo / centrar | H / . / Espacio |
| Grupos | Ctrl + 1..9 guarda · 1..9 recupera |
| Marcador en vivo | F4 muestra u oculta |

## Partidas guardadas

`src/ui/savegame.ts`: la partida contra la IA se guarda en el navegador (`age.save`) cada 4 s, al ocultar la pestaña y al cerrarla: semilla, configuración, órdenes aplicadas (`sim.log`), tick, cámara y sesión on-chain. Al volver se ofrece continuarla. Las órdenes vuelven a la cola en su tick y el bucle adelanta la simulación por tramos de ~25 ms con una barra de avance (30 min de juego ≈ 3 s), descartando los eventos del pasado. El resultado es idéntico bit a bit (prueba en `test/sim.test.ts`). No se guardan 1 vs 1 ni modos de captura; una partida de otra versión de las reglas se descarta con aviso, y la que termina se borra (la victoria se guarda aparte hasta cobrarla). El botón Atrás pregunta antes de salir y cerrar la pestaña pide confirmación.

## Tutorial y fin de partida

- **Tutorial** (`src/ui/tutorial.ts`): seis pasos en la primera partida contra la IA. Cada paso revisa el estado de la simulación (selección, órdenes de recolectar, casas, cola del centro urbano, ejército) y avanza solo; el botón que toca usar parpadea. Se guarda en el navegador (`age.tut`) y se puede repetir con `?tutorial`. No aparece en 1 vs 1 ni en los modos de captura.
- **Fin de partida:** el evento `win` trae el motivo (`town`, `resign` o `wonder`) y la posición del centro urbano caído o de la Maravilla. La cámara viaja ahí 2 s mientras la partida sigue (se ve el derrumbe); el replay, los recursos y las estadísticas se toman en el momento exacto del final, así que el cobro no cambia. En 1 vs 1 el lockstep se detiene de inmediato.
- **Reportes:** el botón de insecto manda el texto del jugador, la partida en curso, los últimos errores y el equipo a `POST /api/report` (6 por hora por IP; se guardan en `server/data/reports`, fuera de git). `GET /api/report/<id>` solo responde desde la propia máquina, nunca por el túnel.
- **Barra de producción** (`src/ui/prodbar.ts`): recorre las colas de tus edificios y agrupa por unidad o mejora (`Validador ×5`, `Centinela ×4` entre dos cuarteles, la era en curso). El avance es el del primero de cada cola que está más cerca de salir; ámbar si espera población. Clic: selecciona y centra un edificio que lo produce, rotando entre ellos.
- **Marcador en vivo** (`src/ui/score.ts`): `sim.score(owner)` = recursos juntados / 10 + valor de lo destruido al rival / 5 + unidades y edificios en pie / 10 + eras y mejoras / 10. Los contadores `gathered` y `destroyed` no entran en el hash, así que no cambian los replays ni la verificación. A la IA se le cuenta también su ingreso pasivo. `F4` lo oculta (se recuerda en el navegador).
- **Clasificación:** `GET /api/leaderboard` suma los eventos `RewardClaimed` de los registros actual y anteriores (`registryOld` en `deployment.json`) y separa IA de 1 vs 1 con `MatchReported`. Se recalcula como mucho una vez por minuto.

## Modos de desarrollo

`?autoplay`, `?demo` (aldeanos trabajando), `?demo=tc` (punto de reunión y cola llena de 10), `?demo=crowd&t=40` (20 aldeanos en madera y oro, adelantado 40 s), `?demo=readme&era=3&zoom=19` (la base de las capturas del README), `?demo=naval&map=costa` (muelle y flota), `?demo=v`, `?demo=fx&zoom=9&cx=&cy=` (edificios dañados, combate y cambio de era al segundo y medio), `?showcase=N` (todos los modelos en la era N, con `&focus=town|houses|team1|castle|units|eco|mil|compare|fauna|farms|people|wonder|gates`), `?tutorial` (repite el tutorial), `?reporte=<id>` (vuelve a jugar la partida de un reporte hasta 20 s antes de enviarlo), `?film=<toma>&rec&clean&fz=` (modo cine del comercial: un cuadro exacto por llamada a `window.__rec.frame()`).
