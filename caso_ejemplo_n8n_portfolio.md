# Caso de Ejemplo Demostrable — Agente n8n para Portfolio (Fiverr/Resultise/AutoMatch)

## 1. Nombre del producto
**Universal Miner & Trader Monitor** — Agente de monitoreo con alertas IA para rigs de minería y bots de trading.

## 2. Qué hace (entrada → salida)
- **Entrada:** Webhook desde rig remoto (hashrate, temperatura, consumo) y desde gateway de trading (P&L, estado de orden, balance USDT).
- **Procesamiento:** Pipeline n8n → filtrado de umbrales → validación con modelo local Ornith-1.5:9b (Ollama, costo $0) → decisión de alerta.
- **Salida:** Mensaje de Telegram estructurado: "⚠️ Hashrate cayó 35% en 10 min en rig X. Revisar conexión." o "✅ Todo estable."

## 3. Datos de ejemplo (sin secretos, públicos simulados)
**Input rig (webhook):**
```json
{
  "timestamp": "2026-11-23T09:45:00Z",
  "rig_id": "rig-01",
  "hashrate_ths": 86.4,
  "hashrate_prev_10min_ths": 133.0,
  "power_w": 287,
  "temp_c": 68,
  "pool": "pearlhash.xyz",
  "coin": "PRL"
}
```
**Umbral disparado:** caída del 35% (>30% configurado) → agente envía alerta.

**Output Telegram:**
```
⚠️ ALERTA RIG-01
Hashrate cayó 35% en 10 min (133.0 → 86.4 TH/s).
Posible causa: desconexión pool o GPU offline.
Acción sugerida: revisar conexión y temperatura.
```

**Input trading (webhook):**
```json
{
  "timestamp": "2026-11-23T10:02:00Z",
  "bot": "dca_robot",
  "pnl_usdt": -2.45,
  "pnl_pct": -2.1,
  "order_status": "open",
  "pair": "PEARL/USDC",
  "entry_price": 1.7265
}
```
**Umbral disparado:** P&L < -2% → alerta crítica.

**Output Telegram:**
```
🚨 ALERTA TRADING DCA
P&L acumulado -2.1% (–$2.45 USDT).
Orden buy 729778466 @1.7265 sigue abierta desde 18/11.
No hay nueva compra. Revisar trail_stop y book.
```

## 4. Estructura n8n (workflow base, no hardcodeado)
```
Webhook (rig/gateway) → Split by source → 
  Rama Rig: Filter (hashrate_drop > 30% en 10 min) → Ornith LLM (redactar alerta) → Telegram
  Rama Trading: Filter (pnl_pct < -2%) → Ornith LLM (redactar alerta) → Telegram
  Rama OK: No-op (silencio, no spam)
```

## 5. Configuración por cliente (config.yaml base — se copia y ajusta por cliente)
```yaml
cliente: "nombre_cliente"
umbrales:
  hashrate_drop_percent: 30
  hashrate_timeframe_min: 10
  pnl_threshold_pct: -2
  error_rate_percent: 5
canales:
  telegram_bot_token: "${TELEGRAM_BOT_TOKEN}"
  telegram_chat_id: "${TELEGRAM_CHAT_ID}"
motor:
  modelo: "ornith-1.5:9b"
  endpoint: "http://localhost:11434"
  costo_api: 0
```

## 6. Secretos (NUNCA hardcodeados)
Archivo `.env` por cliente, cargado en runtime:
```
TELEGRAM_BOT_TOKEN=tu_token_aquí
WEBHOOK_SECRET=tu_secreto_aquí
OLLAMA_HOST=http://localhost:11434
```

## 7. Por qué vende (valor para el cliente)
- Evita downtime del rig (cada hora parada = pérdida de mining).
- Detecta órdenes varadas antes de que se pierda una oportunidad de compra/venta.
- Motor local: no paga API de LLM por cada alerta, costo operativo cero.
- Template reutilizable: se vende a múltiples clientes cambiando solo el `config.yaml`.

## 8. Precio de entrada sugerido (Némesis ya lo aprobó)
- **Básico:** $150 — 1 fuente de datos (rig o trading), 1 umbral, 1 canal Telegram.
- **Estándar:** $300 — 2 fuentes, múltiples umbrales, configuración personalizada.
- **Premium:** $500 — rig + trading + dashboard diario + soporte 7 días.

## 9. Qué falta para publicar
- Darío debe ejecutar el flow OAuth para restaurar `google_token.json` (opcional: no bloquea el gig).
- Némesis debe dar OK final para publicar en Fiverr/Resultise/AutoMatch.
- No se gasta ni un centavo de los USD 3.

---

Cúspide — listo para revisión de Némesis antes de cualquier publicación.