# Contratos

Nueve contratos en Solidity 0.8.28 (Foundry + OpenZeppelin 5), en `contracts/src`.

```
                 ┌──────────────┐  paga recompensas    ┌───────────────┐
                 │ MatchRegistry│──── RewardsPool ───▶│  GloryToken   │◀───────────┐
                 └──────▲───────┘                     └──────▲────────┘            │
     firma EIP-712      │                                    │ paga / cobra        │ pools x·y=k
     del verificador    │                                    │                     │
┌──────────────┐        │        ┌─────────────┐      ┌──────┴────────┐      ┌─────┴──────────┐
│  Verificador │────────┴───────▶│ MatchVault  │─────▶│ResourceMarket │◀────▶│ ResourceToken×4│
│ (servidor)   │  firma settle   └──────┬──────┘      └───────────────┘      │ FOOD WOOD GOLD │
└──────────────┘                        │ quema NFT                          │ STONE          │
                                 ┌──────▼──────┐      ┌───────────────┐      └────────────────┘
                                 │  GameItems  │◀────▶│  Marketplace  │
                                 │  ERC-1155   │      │  P2P + 5%     │
                                 └─────────────┘      └───────────────┘
```

## Direcciones

Se regeneran en cada despliegue y quedan escritas en `game/src/chain/deployment.json`, que es lo que lee el juego. El despliegue de Fuji (chain ID 777011) se guarda además en `deployment.fuji.json`.

---|---|
| GloryToken | `0xDbDa30ad00b67702BDCEd019C3b46D35DB9e94ce` |
| GameItems | `0x4359190563c0F38C84037a28288E01A2b1011Ec8` |
| Marketplace | `0x5F909f607A29156Dd9Fc43382d09420bba7799A6` |
| MatchRegistry | `0xb99441DA4A918604d4ee7D1CB20B362Bd3c12705` |
| ResourceMarket (AMM) | `0x20BC04ad10B6300F542e694f8c3aB44DB8Caac65` |
| MatchVault | `0x9c4cD519100100ec3B3c7bff3Df7f52b575F5558` |
| FOOD | `0x1678B18a370C65004c8e4e03b6bf4bE76EaDf4F1` |
| WOOD | `0x277f722E66e81585272CE53413c487be198BcB85` |
| GOLD | `0xf1db872E6454D553686b088c1Ea3889cF2FE3ABe` |
| STONE | `0x224A8113006f2c986217A832e364D676C19C7c21` |

---

## GloryToken — `GLORY` (ERC-20 + Permit + Capped)

Moneda de la economía, con **suministro fijo de 100 M** (`ERC20Capped`). No es el oro que se mina dentro de la partida.

- Todo el suministro se acuña en el despliegue y se reparte (ver [ECONOMIA.md](ECONOMIA.md)). Después se revoca `MINTER_ROLE` y nadie puede acuñar más.
- `burn(amount)`: lo usa la tienda (50% de cada compra) y cualquiera que quiera quemar.

## RewardsPool — de aquí salen todas las recompensas (40 M GLORY)

| Función | Qué hace |
|---|---|
| `pay(player, base)` | Solo `PAYER_ROLE` (`MatchRegistry`). Paga la recompensa aplicando los límites |
| `quote(player, base)` | Cuánto pagaría hoy, sin modificar estado |
| `dailyBudget(day)` | Presupuesto global del día: 54.794 GLORY el primer año, la mitad cada 365 días |
| `factorBps(n)` | Rendimientos decrecientes: recompensas 1–3 del día 100%, 4–6 50%, 7–10 25%, después 0 |

Tope por jugador: 200 GLORY al día. Si el presupuesto del día se agota, la victoria se registra igual pero paga 0.

## TeamVesting — 15 M GLORY del equipo

`VestingWalletCliff` de OpenZeppelin: 1 año de cliff y liberación lineal en 3 años hacia el beneficiario `TEAM`.

## ResourceToken ×4 — `FOOD`, `WOOD`, `GOLD`, `STONE` (ERC-20)

Recursos del juego tokenizados.

- `MINTER_ROLE`: `MatchVault`, que acuña al liquidar una partida. El admin pierde el rol después de sembrar los pools.
- `burn(amount)`: la bóveda quema lo que el jugador compra para meterlo a la partida.

## GameItems — activos del juego (ERC-1155 + ERC-2981)

Skins, ejército, planos de edificios y civilizaciones.

| Rol | Quién | Para qué |
|---|---|---|
| `CURATOR_ROLE` | admin | `defineItem`: nombre, categoría, rareza, suministro máximo y precio en tienda |
| `MINTER_ROLE` | — | acuñación directa (promos) |
| `BURNER_ROLE` | `MatchVault` | quemar ejército/planos al abrir una partida |

- `buyFromShop(id, amount)`: tienda oficial pagada en GLORY. **La mitad se quema** y la mitad va al tesoro.
- **Regalías:** 5% (`_setDefaultRoyalty(treasury, 500)`) en ventas secundarias.

Catálogo inicial (`Deploy.s.sol`):

| ID | Ítem | Categoría | Rareza | Suministro | Precio (GLORY) |
|---|---|---|---|---|---|
| 1 | Legión Esmeralda | Skin | Común | ∞ | 25 |
| 2 | Guardia Púrpura | Skin | Rara | 5.000 | 80 |
| 3 | Imperio Dorado | Skin | Épica | 500 | 250 |
| 4 | Sombra de Avalanche | Skin | Legendaria | 50 | 900 |
| 101 | Pelotón de milicia | Ejército | Común | ∞ | 40 |
| 102 | Escuadra de arqueros | Ejército | Común | ∞ | 60 |
| 103 | Caballeros mercenarios | Ejército | Rara | 5.000 | 120 |
| 104 | Caravana de aldeanos | Ejército | Común | ∞ | 50 |
| 111 | Plano: torre de vigilancia | Plano | Común | ∞ | 60 |
| 112 | Plano: mercado | Plano | Común | ∞ | 80 |
| 113 | Plano: castillo | Plano | Épica | 500 | 400 |
| 201 | Nórdicos del Glaciar | Civilización | Rara | 1.000 | 300 |
| 202 | Imperio Subnet | Civilización | Rara | 1.000 | 300 |
| 203 | Guardianes de la C-Chain | Civilización | Épica | 500 | 450 |
| 204 | Jinetes de la P-Chain | Civilización | Épica | 500 | 450 |
| 205 | Arqueros de la X-Chain | Civilización | Legendaria | 100 | 800 |

La civilización base, **Pioneros de Avalanche** (ID 0), es gratis y no es NFT.

## Marketplace — compraventa P2P

- `list(itemId, amount, pricePerUnit)`: el vendedor deja los ítems en custodia.
- `buy(listingId, amount)`: admite compras parciales. Paga el precio en GLORY y reparte la regalía ERC-2981 automáticamente.
- `cancel(listingId)` y `activeListings(from, to)`.

## MatchRegistry — resultados verificables (modelo optimista)

La partida se juega en el cliente con simulación determinista. Al terminar:

1. El **verificador** (`server/reporter.ts`) re-simula `seed + log de comandos`. Si el hash del estado coincide, firma `MatchResult{matchId, winner, loser, seed, logHash, stateHash, ticks}` con EIP-712 (dominio `AgeMatchRegistry` v1).
2. Cualquiera llama `report(result, sig)`. La firma debe venir de un `REPORTER_ROLE`.
3. Durante la **ventana de disputa** cualquiera puede llamar `dispute`. Un `ARBITER_ROLE` re-simula y llama `resolve(valid)`.
4. Pasada la ventana, cualquiera llama `claim` (lo hace el relayer del servidor) y el **RewardsPool** paga al ganador. Aquí no se acuña nada. Se registran victorias y derrotas.

Protecciones: `SelfMatch` (ganador = perdedor) y, en 1vs1, una sola recompensa al día por pareja de rivales (`maxPairRewardsPerDay`).

| Parámetro (despliegue) | Valor |
|---|---|
| Ventana de disputa | `DISPUTE_WINDOW` (600 s por defecto, 120 s en Fuji) |
| Recompensa base PvP | 50 GLORY |
| Recompensa base contra IA | 10 GLORY |

## MatchVault — puente entre la partida y la economía

| Paso | Función | Qué pasa |
|---|---|---|
| Abrir | `open(seed, map, civ, ids, amounts)` | Exige poseer el NFT de la civilización y **quema** hasta 3 NFT de ejército/planos. `sessionId = keccak256(player, seed)`. |
| Comprar | `buyResource(id, res, amount, maxGloryIn)` | El jugador paga GLORY, el AMM entrega el recurso, la bóveda lo quema y el recurso entra a la partida al instante. |
| Vender | `sellResource(id, res, amount)` | Registra la venta. Se paga al liquidar. |
| Liquidar | `settle(settlement, sig, sellAll)` | Con la firma EIP-712 del verificador (dominio `AgeMatchVault` v1) acuña lo vendido más lo sobrante. Con `sellAll` lo vende todo en el AMM por GLORY en la misma transacción. |

Protecciones:
- lo vendido no puede superar lo registrado (`SoldMismatch`);
- el sobrante tiene tope (`leftoverCap` = 500 por recurso);
- máximo de liquidaciones por jugador al día (`maxSettlementsPerDay` = 5);
- **tope global diario** de recursos acuñados entre todas las cuentas (`dailyMintCap` = 50.000 por recurso, `DailyMintCap`);
- cada sesión se liquida una sola vez.

---

## Compilar, probar y desplegar

```bash
cd contracts
pnpm install                       # OpenZeppelin
forge install foundry-rs/forge-std # si falta lib/forge-std
forge build
forge test                         # 18 pruebas: Game.t.sol, Economy.t.sol y Rewards.t.sol

set -a && . ../chain/.env && set +a
forge script script/Deploy.s.sol --rpc-url $AGE_RPC_URL --private-key $DEPLOYER_PK --broadcast
cast send $REPORTER --value 10ether --rpc-url $AGE_RPC_URL --private-key $DEPLOYER_PK   # gas para el verificador
```

El script escribe las direcciones en `game/src/chain/deployment.json`, que es lo que lee el cliente. Después de cambiar un contrato, regenera las ABI del juego con `python3 contracts/script/gen-abis.py`.

Variables opcionales del despliegue: `TREASURY`, `TEAM`, `COMMUNITY`, `RESERVE` (por defecto, el deployer) y `DISPUTE_WINDOW`.

## Antes de producción

- Pasar los roles de admin a un multisig y separar curador, árbitro y verificador.
- Pedir una auditoría externa de `MatchVault` y `ResourceMarket`, que son los que mueven valor.
- Varios verificadores independientes, o un esquema de pruebas (fraud proofs) en lugar de un solo firmante.
- Ajustar `leftoverCap` y `maxSettlementsPerDay` con datos reales de partidas.
