# chasqui-dashboard-data

Histórico diario de *Tu Chasqui Virtual*: cotizaciones de futuros, tipos de
cambio e importaciones SUNAT.

## Serie canónica: el settlement

**La serie canónica de precios es el `settlement`, y arranca el 2026-07-09.**

Hasta septiembre de 2026 el archivo guardaba el precio que el bot veía al
momento de correr. Eso no es una serie comparable: 27 de 30 corridas
programadas arrancan entre las 10:00 y las 14:00 UTC, antes de que liquide
ICE Sugar #11 (~17:00 UTC), y GitHub dispara el cron entre las 10:00 y las
21:00. Medido sobre los días solapados, 8 de 9 difieren del settlement, hasta
un ~1.8%.

Por eso ahora conviven dos métricas por contrato y día:

| métrica | qué es |
|---|---|
| `settlement` | el cierre oficial, tomado de price-history al día siguiente. **Canónica.** |
| `intradia` | el precio que el bot vio durante la sesión. Puede faltar. |

La columna `origen` dice de dónde salió cada fila:

- `settlement_historico` — de la página de historia de Barchart.
- `captura_vivo` — foto durante la sesión, con `hora_captura_utc` registrada.
- `captura_vivo_hora_desconocida` — igual, pero de antes de que se registrara
  la hora. No se puede recuperar.

## Días sin sesión

`sin_sesion = true` marca días en que el mercado no operó (feriados). En esos
días la página seguía mostrando el precio de la sesión anterior y el bot lo
archivaba como si fuera del día, produciendo planicies que parecen datos
reales.

**Esas filas hay que excluirlas de cualquier tendencia, media móvil o
variación día contra día.**

Días sin sesión detectados: ninguno todavía

## Huecos, sin rellenar

Nunca se interpola ni se arrastra un valor de un día a otro. Un hueco visible
es mejor que una línea suave que miente.

| fecha | motivo |
|---|---|
| 2026-05-29 | no_hubo_corrida |
| 2026-06-30 | fuera_de_rango_historia_gratuita |

Del 2026-05-20 al 2026-07-08 no hay `settlement`: la historia gratuita de
Barchart no llega tan atrás (`motivo_sin_settlement =
fuera_de_rango_historia_gratuita`). Esos días conservan su `intradia`.

## SUNAT

El modelo de SUNAT **no cambia**. Una declaración de aduana no se mueve
durante el día: no hay settlement ni problema de hora de captura. Sus filas
dejan vacías las columnas `origen`, `sin_sesion` y `hora_captura_utc`.

La comparación es siempre contra el día hábil anterior (lunes contra viernes).
Esa regla está implementada, pero **no contempla feriados peruanos**: si el
día base cayó feriado, la variación queda inflada. El pipeline avisa cuando el
día base no tiene ninguna declaración.

## Archivos

- `historico_serie.csv` — formato largo, una fila por
  (fecha, categoría, instrumento, contrato, métrica).
- `manifest.json` — índice de días con su procedencia y sus huecos.
- `historico/<fecha>.json` — snapshot crudo de cada día en vivo.
- `latest.json` — último día publicado.

Generado automáticamente. Última actualización: 2026-09-05T20:05:41+00:00.
