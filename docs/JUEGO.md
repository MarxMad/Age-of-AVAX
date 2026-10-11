# El juego

RTS en 3D (Three.js) con la dinámica clásica de Age of Empires: recolectar, construir, avanzar de era y destruir el centro urbano enemigo. El catálogo completo (costos, vida, ataque y tiempos) vive en `game/src/sim/defs.ts` y es la única fuente de verdad.

## Simulación determinista

- Tick fijo de **30 Hz** y RNG con semilla, sin DOM ni render (`src/sim`).
- El jugador solo modifica la partida con **comandos** (`train`, `rally`, `build`, `order`…). `seed + log de comandos` reproduce exactamente la partida, y el verificador lo usa para firmar resultados.
- `SIM_VERSION` (hoy **0.13.0**) debe coincidir entre el cliente y el verificador. Súbela cada vez que cambien las reglas y reinicia `pnpm server`.

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

## Granjas

- **Tamaño:** el modelo llena exactamente su terreno de 3×3 casillas (120 px), igual que la vista previa de construcción.
- **Granjas pegadas:** se construyen una junto a otra, como las casas, y forman una parcela continua. Shift + clic coloca varias seguidas.
- **Trigo:** va por surcos (una malla por surco) y se vacía surco a surco al cosecharse. Cuesta 10 llamadas de dibujo por granja, no una por espiga.

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
- **Prueba:** `test/movement.test.ts` hace trabajar a 20 aldeanos durante 3 minutos y verifica que ninguno pase más de 2 s sin avanzar ni trabajar. Con el código anterior, alguno quedaba trabado 55 s.

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

- **Recursos por mapa** (semilla 7):

| Mapa | Árboles | Oro + piedra | Bayas | Ganado (ovejas + vacas) | Ciervos | Jabalíes | Lobos |
|---|---|---|---|---|---|---|---|
| Continental | 388 (antes 163) | 21 (antes 9) | 19 (antes 6) | 23 | 14 | 5 | 7 |
| Desierto | 219 | 28 | 21 | 24 | 16 | 9 | 6 |
| Islas | 167 | 18 | 13 | 21 | 6 | 7 | 0 |
| Nieve | 365 | 21 | 18 | 23 | 11 | 5 | 4 |

- **Cada base:**
  - tres bosques;
  - dos arbustos de bayas;
  - ocho ovejas y dos vacas;
  - dos jabalíes cerca.
- **Resto del mapa:** minas de oro y canteras repartidas a más de 520 px de las bases, para que valga la pena expandirse.

| Animal | Comida | Comportamiento |
|---|---|---|
| Oveja | 100 | ganado: pasta cerca y se recolecta rápido |
| Vaca | 180 | ganado lento y robusto |
| Ciervo | 140 | salvaje, huye pastando; se caza al instante |
| Jabalí | 320 | **se defiende**: 70 de vida y 3 de ataque. El aldeano que lo caza lo ataca con su lanza (doble daño) hasta abatirlo y luego lo recolecta. Mejor entre dos. |
| Lobo | — | **depredador**: 45 de vida y 4 de ataque. Ataca a cualquier unidad a menos de 130 px y la persigue. Aparece en manadas, a más de 800 px de las bases (más en la nieve). Los militares ociosos lo atacan solos. |

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

## Modos de desarrollo

`?autoplay`, `?demo` (aldeanos trabajando), `?demo=tc` (punto de reunión y cola llena de 10), `?demo=crowd&t=40` (20 aldeanos en madera y oro, adelantado 40 s), `?demo=readme&era=3&zoom=19` (la base de las capturas del README), `?demo=naval&map=costa` (muelle y flota), `?demo=v`, `?demo=fx&zoom=9&cx=&cy=` (edificios dañados, combate y cambio de era al segundo y medio), `?showcase=N` (todos los modelos en la era N, con `&focus=town|houses|team1|castle|units|eco|mil|compare|fauna|farms|people|wonder|gates`).
