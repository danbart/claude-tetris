---
name: clima
description: Consulta el clima actual y el pronóstico usando wttr.in (sin API key). Úsala cuando el usuario pregunte por el clima, la temperatura, si va a llover, el pronóstico, o el tiempo en su ciudad o en cualquier otra. Trigger - "clima", "tiempo", "temperatura", "va a llover", "pronóstico", "weather".
allowed-tools: Bash(curl:*), Read
---

# Clima

Consulta el clima vía `wttr.in`. No necesita API key ni cuenta. Sin ubicación explícita, `wttr.in` geolocaliza por IP.

## Ubicación por defecto

Si existe `.claude/skills/clima/location.txt`, lee ese archivo y usa su contenido como ubicación por defecto. Si no existe, deja que wttr.in use la IP.

## Comandos

Clima actual, resumen de una línea:

```bash
curl -s "https://wttr.in/UBICACION?format=%l:+%c+%t+(sensación+%f)+viento+%w+humedad+%h"
```

Pronóstico de hoy + 2 días (texto ASCII, formato compacto):

```bash
curl -s "https://wttr.in/UBICACION?m&lang=es&2qn"
```

Datos completos en JSON (usar cuando se necesite extraer valores concretos):

```bash
curl -s "https://wttr.in/UBICACION?format=j1"
```

Solo luna:

```bash
curl -s "https://wttr.in/Moon"
```

## Parámetros de URL

- `UBICACION`: ciudad (`Madrid`), `ciudad,pais` (`Bogota,CO`), código de aeropuerto (`MAD`), `~Torre+Eiffel` para un punto de interés, `@dominio.com` para geolocalizar por dominio. Espacios como `+`. Vacío = geolocalización por IP.
- `m` = sistema métrico (°C, km/h). `u` = imperial.
- `lang=es` = descripciones en español.
- `0`/`1`/`2` = número de días de pronóstico (`0` = solo actual).
- `n` = versión estrecha, `q` = sin cabecera, `T` = sin colores ANSI.
- `format=j1` = JSON completo.

## Campos de `format=` personalizado

`%l` ubicación · `%c` icono · `%C` condición · `%t` temperatura · `%f` sensación térmica · `%w` viento · `%h` humedad · `%p` precipitación · `%P` presión · `%m` fase lunar · `%D` amanecer · `%s` atardecer · `%T` hora local.

## Flujo

1. Determinar ubicación: la que dio el usuario > `location.txt` > IP.
2. Ejecutar el `curl`. Añadir `--max-time 10`.
3. Responder en español, breve: temperatura, sensación, condición, y lo que el usuario preguntó (lluvia, viento, etc.).

## Errores

- `curl` falla o timeout: no hay red o wttr.in está caído. Decirlo, no inventar datos.
- Respuesta con `Unknown location`: la ubicación no existe. Pedir que la reescriba o probar `ciudad,pais`.
- HTTP 429: wttr.in limita por IP. Esperar y reintentar, no insistir en bucle.

Nunca inventar valores de clima. Si el comando no devuelve datos, decir que no se pudo consultar.
