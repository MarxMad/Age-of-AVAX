# Economía y funcionamiento on-chain

**Juega gratis, conecta para ganar.** Cualquiera juega sin wallet. Al conectar una wallet, las victorias pagan GLORY y el jugador nunca paga gas.

## Qué va a la cadena y qué no

La partida **no** se juega en la blockchain: sería lenta y cara. Se juega en el navegador con una simulación **determinista**: `semilla + lista de comandos` reproduce la partida exacta en cualquier máquina. A la cadena solo va lo que importa:

| Dato | Dónde vive |
|---|---|
| Cada clic, cada unidad, cada tick | Navegador del jugador (y, en 1vs1, el log del servidor de salas) |
| `logHash`: huella del replay completo | On-chain, en `MatchRegistry` |
| `stateHash`: huella del estado final | On-chain |
| Ganador, perdedor, semilla y duración | On-chain |
| Récord de victorias y derrotas | On-chain |
| GLORY pagado | On-chain (`RewardsPool` → wallet) |
| Recursos comprados y vendidos en el Mercado | On-chain (`MatchVault` + AMM) |

**Cómo se registra un resultado:**

1. Termina la partida y el cliente manda el replay (semilla + comandos) al **verificador**.
2. El verificador la **re-simula** con el mismo código del juego. Si el ganador y el hash coinciden, firma el resultado (EIP-712).
3. El **relayer** (el mismo servidor) envía `MatchRegistry.report(resultado, firma)` y paga el gas.
4. Se abre una **ventana de disputa**: cualquiera puede re-simular el replay (su hash está en la cadena) y disputarlo.
5. Al cerrar la ventana, el relayer llama `claim` y el `RewardsPool` paga al ganador.

Es un modelo **optimista**: se confía en el resultado firmado, pero cualquiera puede verificarlo y disputarlo.

## Recorrido del jugador

```
Invitado ──► juega gratis (semilla del servidor)
   │            └─ gana ─► la victoria se guarda 24 h en su navegador
   ▼
Conecta wallet (Core / MetaMask, o wallet de sesión)
   ├─ recibe 0,2 AGE de gas patrocinado (una vez al día como máximo)
   ├─ cobra sus victorias de invitado pendientes
   └─ desde aquí, cada victoria: "Cobrar GLORY" ─► el servidor registra y cobra solo
   ▼
Dueño de activos: civilizaciones NFT, ejército y planos, Mercado con swaps reales, 1vs1 con recompensa
```

- **Sin gas para cobrar:** el servidor envía `report` y `claim` y paga el AGE, que se acuña con NativeMinter en nuestra L1 y no tiene costo real.
- **Gas patrocinado:** para lo que sí firma el jugador (abrir sesión, comprar en el Mercado o la Bóveda), cada wallet nueva recibe 0,2 AGE. Con una base fee de 1 gwei alcanza para cientos de transacciones.

## GLORY: suministro fijo de 100 millones

`GloryToken` es `ERC20Capped(100 M)`. Todo se acuña en el despliegue y después **nadie conserva `MINTER_ROLE`**, así que no se puede imprimir más. Lo verifica el propio script (`require(totalSupply == MAX_SUPPLY)`).

| Destino | GLORY | % | Cómo se libera |
|---|---|---|---|
| `RewardsPool` (recompensas por jugar) | 40.000.000 | 40% | Presupuesto diario que se reduce a la mitad cada año |
| Liquidez | 15.000.000 | 15% | 95.000 sembrados en los 4 pools; el resto es reserva de liquidez |
| Ecosistema / Team1 | 15.000.000 | 15% | Tesoro (`TREASURY`): torneos, chapters, hackathons |
| Equipo | 15.000.000 | 15% | `TeamVesting`: 1 año de cliff y liberación lineal en 3 años |
| Comunidad | 10.000.000 | 10% | Airdrops y quests; en testnet también paga el faucet |
| Reserva | 5.000.000 | 5% | Multisig |

### Emisión de recompensas

- **Presupuesto global:** 54.794 GLORY/día el primer año (≈20 M), la mitad el segundo, y así. La suma converge a los 40 M del fondo.
- **Recompensa base:** 10 GLORY por victoria contra la IA y 50 por victoria 1vs1.
- **Tope por jugador:** 200 GLORY al día.

### Sumideros (GLORY que sale de circulación)

| Sumidero | Efecto |
|---|---|
| Tienda oficial (Bóveda) | **50% se quema** y 50% va al tesoro |
| NFT de ejército y planos | Se queman al llevarlos a una partida |
| Marketplace | 5% de regalías al tesoro |
| Mercado del juego | El GLORY pagado por recursos se queda en los pools |

## Protección contra farmers y bots

La defensa es en capas: la cadena pone **límites duros** y el servidor **filtra** lo que llega.

**En la cadena (no se pueden saltar):**

| Capa | Contrato | Qué frena |
|---|---|---|
| Presupuesto global diario con halving | `RewardsPool` | Miles de cuentas falsas: la emisión total por día está acotada |
| Tope de 200 GLORY por jugador al día | `RewardsPool` | Un jugador (o bot) jugando todo el día |
| Rendimientos decrecientes (100% · 50% · 25% · 0) | `RewardsPool` | Grindear decenas de partidas |
| Una recompensa al día por pareja de rivales | `MatchRegistry` | Dos cuentas propias jugando entre sí |
| Prohibido jugar contra uno mismo | `MatchRegistry` | Autopartidas |
| Tope global diario de recursos acuñados (50.000 por recurso) | `MatchVault` | Drenar el GLORY de los pools vía liquidaciones |
| Sobrante máximo de 500 por recurso y 5 liquidaciones al día | `MatchVault` | Acumular recursos para venderlos |

**En el servidor (verificador):**

| Capa | Qué frena |
|---|---|
| Semillas de un solo uso emitidas por el servidor (máx. 30 por IP y hora) | Reciclar replays ganadores o precalcular semillas favorables |
| **Tiempo real:** la victoria no se acepta antes de que pase la duración real de la partida | Bots que simulan una partida de 20 minutos en 2 segundos |
| Duración mínima: 5 min contra la IA, 4 min en 1vs1 | Victorias exprés |
| Re-simulación completa del replay | Replays alterados o inventados |
| Victorias de invitado válidas solo 24 h | Acumular victorias para cobrarlas en masa |
| 1vs1: wallets probadas con firma; el servidor guarda el log | Jugar a nombre de otra dirección |
| Límites por IP en gas patrocinado, faucet, salas y RPC | Abuso de los recursos gratuitos |

Todas las capas se probaron: hay 18 pruebas de contratos (`contracts/test/Rewards.t.sol`, `Economy.t.sol` y `Game.t.sol`) y los scripts `game/scripts/e2e.ts` y `pvp-e2e.ts`.

## Parámetros ajustables (admin)

| Parámetro | Función | Valor inicial |
|---|---|---|
| Presupuesto diario y tope por jugador | `RewardsPool.setParams` | 54.794 / 200 |
| Ventana de disputa y recompensas PvP / IA | `MatchRegistry.setParams` | 120 s en Fuji / 50 / 10 |
| Recompensas por pareja al día | `MatchRegistry.setMaxPairRewardsPerDay` | 1 |
| Sobrante máximo y liquidaciones al día | `MatchVault.setParams` | 500 / 5 |
| Tope global diario de recursos | `MatchVault.setDailyMintCap` | 50.000 |

> En mainnet, los roles de admin deben pasar a un multisig y las claves del verificador a un KMS.
