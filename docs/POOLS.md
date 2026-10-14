# Pools de recursos (ResourceMarket)

`ResourceMarket` es un AMM de **producto constante** (`x · y = k`, como Uniswap v2) con **un pool por recurso contra GLORY**:

| Pool | Índice | Token |
|---|---|---|
| FOOD / GLORY | 0 | Age Food |
| WOOD / GLORY | 1 | Age Wood |
| GOLD / GLORY | 2 | Age Gold |
| STONE / GLORY | 3 | Age Stone |

- **Comisión:** 0,3% (`FEE_BPS = 30`). Se queda en el pool y aumenta `k`.
- **Liquidez:** propiedad del protocolo (tesoro). Solo el admin puede `addLiquidity`. No hay tokens LP.
- **Vender recursos nunca acuña GLORY:** solo saca GLORY del pool, así que la emisión de GLORY no depende del mercado.

## Liquidez inicial

| Pool | Recurso | GLORY | Precio inicial | k |
|---|---|---|---|---|
| FOOD | 400.000 | 20.000 | 0,05 GLORY | 8,0·10⁹ |
| WOOD | 400.000 | 20.000 | 0,05 GLORY | 8,0·10⁹ |
| GOLD | 200.000 | 30.000 | 0,15 GLORY | 6,0·10⁹ |
| STONE | 250.000 | 25.000 | 0,10 GLORY | 6,25·10⁹ |

Total sembrado: **95.000 GLORY**. La escasez relativa refleja el juego: el oro es el más caro y la comida y la madera los más baratos.

## Fórmulas

Con `F = 10.000 − 30`:

```
vender  resIn  →  gloryOut = resIn·F·G / (R·10.000 + resIn·F)
comprar resOut →  gloryIn  = G·resOut·10.000 / ((R − resOut)·F) + 1
precio marginal = G / R
```

donde `R` es la reserva del recurso y `G` la reserva de GLORY.

### Cotizaciones con los pools recién sembrados

| Operación | FOOD | WOOD | GOLD | STONE |
|---|---|---|---|---|
| Comprar 100 | 5,016 | 5,016 | 15,053 | 10,034 |
| Vender 100 | 4,984 | 4,984 | 14,948 | 9,966 |
| Comprar 1.000 | 50,28 | 50,28 | 151,21 | 100,70 |

Las cifras están en GLORY. El impacto en precio de 1.000 unidades es de ~0,5%: los pools son profundos para el tamaño de una partida.

## Cómo se usa desde el juego

1. **Comprar en el Mercado del juego** (botones "+100"): `MatchVault.buyResource` paga GLORY al pool, recibe el token, lo **quema** y suma el recurso a la partida. Con bloques de 1 s se confirma casi al instante.
2. **Vender en el Mercado del juego** (botones "−100"): el recurso sale de la partida y la venta queda registrada con `sellResource`. Se paga al liquidar.
3. **Liquidar al terminar:** el verificador re-simula, confirma que esos recursos existían y firma. `settle` acuña lo vendido más lo sobrante (tope de 500 por recurso; los sobrantes solo en una victoria que cobra recompensa, ver [ECONOMIA.md](ECONOMIA.md)) y, con `sellAll`, lo vende en el pool por GLORY.

```
GLORY ──buyResource──▶ pool ──token──▶ MatchVault ──burn──▶ recurso en la partida
recurso en la partida ──sellResource──▶ (registro) ──settle + firma──▶ mint ──▶ pool ──▶ GLORY
```

## Límites de la bóveda (antifarming)

Las liquidaciones son la única vía por la que recursos ganados en una partida se vuelven GLORY de los pools. Por eso tienen límites duros en `MatchVault`:
- 500 de sobrante por recurso y por partida, y solo en victorias que cobran recompensa (en una derrota se pierden);
- 5 liquidaciones por jugador al día;
- un tope **global** de 50.000 unidades por recurso al día entre todas las cuentas;
- el verificador exige que la partida haya durado en tiempo real lo que dice el replay, contado desde la apertura on-chain de la sesión.

## Relación con los sumideros del juego

Dentro de la partida hay sumideros (mejoras por era, reparaciones con costo y almacén con tope y merma, ver [JUEGO.md](JUEGO.md)). Esto evita que los jugadores acumulen recursos sin límite para venderlos en masa al final. Junto con `leftoverCap`, limita la presión vendedora sobre los pools.

## Riesgos y mejoras

- **Arbitraje y bots:** con liquidez del protocolo, un precio desalineado lo corrige cualquiera. Conviene monitorear las reservas.
- **Drenaje de GLORY:** si se vende más de lo que se compra, la reserva de GLORY baja. Hay que vigilar `gloryReserve` y reponer desde el tesoro, o subir la comisión con el admin.
- **Slippage:** `buyExact` y `sell` aceptan `maxGloryIn` / `minGloryOut`. El cliente debería pasar un margen y no `0`.
- **Oráculo:** `price(res)` es el precio spot. No debe usarse como oráculo en otros contratos sin un TWAP.
