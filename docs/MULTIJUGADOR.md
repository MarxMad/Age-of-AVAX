# Multijugador 1vs1

Salas con código: un jugador crea la sala y comparte el código de 5 letras (o el enlace `…/?sala=CODIGO`), y el otro se une. La partida empieza al entrar el segundo jugador.

Paso a paso:

1. **Anfitrión:** JUGAR → pestaña **1 vs 1** → **Crear sala**. Cada sala recibe un código nuevo. Si tienes la sesión iniciada, tu wallet firma el código de la sala (así nadie juega a tu nombre).
2. **Invitado:** abre el enlace o escribe el código y pulsa **Unirse**.
3. La partida arranca en las dos pantallas al mismo tiempo.

Detalles:

- Si el anfitrión no termina de firmar en 20 segundos, la sala sigue abierta y juega como invitado (sin recompensa).
- Para probar en un solo navegador, las dos pestañas comparten la wallet: la segunda entra como invitada y la partida no paga recompensa. Para cobrar GLORY hacen falta dos wallets distintas (y dos conexiones distintas).
- Al entrar con un enlace de invitación, el código se borra de la dirección: recargar no vuelve a ofrecer la sala vieja.

## Lockstep determinista

No se transmite el estado del juego, solo **comandos**. Las dos simulaciones son deterministas, así que con los mismos comandos en el mismo tick llegan al mismo estado.

```
turno = 3 ticks (100 ms)
lo que ordeno en el turno T  ──►  se ejecuta en el turno T + 3 (300 ms de margen para la latencia)
                                  en ambos navegadores, primero 'player' y luego 'enemy'
```

1. Cada cliente manda al servidor, una vez por turno, sus comandos (puede ser una lista vacía).
2. El servidor los **filtra** (en 1vs1 no hay comercio on-chain) y reenvía su versión a los dos clientes.
3. Un cliente solo simula el turno T cuando tiene los comandos de ambos para ese turno. Si el rival se retrasa, aparece "Esperando al rival…".
4. Cada 5 s los clientes comparan el hash del estado. Si difieren, la partida se anula por desincronización.

Código: `src/net/protocol.ts` (protocolo compartido), `src/net/lockstep.ts` (cliente) y `server/rooms.ts` (salas).

### Matemática determinista

`Math.sin`, `Math.cos` y `Math.hypot` pueden diferir en el último bit entre Chrome (V8) y Safari (JavaScriptCore), y eso basta para desincronizar. La simulación usa `src/sim/dmath.ts`, que solo emplea `+ − × ÷ √` (exactos en IEEE-754). El error frente a las funciones nativas es menor a 10⁻¹¹.

## Resultado y recompensa

- El servidor guarda el log completo de la sala y al final **re-simula** para certificar al ganador; no confía en lo que diga ningún cliente.
- **Abandonar** (cerrar la pestaña, "Rendirse" o 15 s sin mandar turnos) da la victoria al rival.
- **Recompensa** (1.000 GLORY base) solo si se cumplen todas estas condiciones:
  - las dos wallets están conectadas y probadas con firma;
  - la partida duró al menos 4 minutos;
  - no avanzó más rápido que el tiempo real;
  - es la primera recompensa del día entre esa pareja.
- El resultado se registra on-chain con ganador **y** perdedor, y suma al récord de ambos.

## Probarlo

```bash
cd game
pnpm server                      # servidor en :8787
pnpm dev                         # dos pestañas (la segunda entra como invitada) o dos navegadores en http://localhost:5173
npx tsx scripts/pvp-e2e.ts       # prueba automática: dos clientes, lockstep, hashes y resultado
```
