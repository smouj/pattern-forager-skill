name: Pattern Forager
version: 1.2.0
description: Descubre patrones, correlaciones y anomalías ocultas en conjuntos de datos y bases de código complejas mediante análisis estadístico, detección de correlación temporal y huella de comportamiento.
author: SMOUJBOT Analytics Team
tags: [pattern, discovery, correlation, analysis, insight, data]
maintainer: ops@smouj.ai
homepage: https://github.com/smouj/openclaw-skills/pattern-forager
license: MIT
dependencies:
  - python>=3.9
  - numpy>=1.24.0
  - pandas>=2.0.0
  - scikit-learn>=1.3.0
  - matplotlib>=3.7.0
  - seaborn>=0.12.0
  - networkx>=3.0
  - scipy>=1.10.0
system_requirements:
  memory_min: 512MB
  memory_peak: 2GB
  cpu_cores: 2
  disk_space: 500MB
```

# Pattern Forager

Descubrimiento automático de patrones, correlaciones y anomalías ocultas en conjuntos de datos y bases de código complejas.

## Propósito

Pattern Forager sirve tres casos de uso principales:

### 1. Detección de Patrones en Codebases
Identifica patrones arquitectónicos recurrentes, puntos críticos de clonación de código y acoplamiento de módulos que no son visibles mediante métricas estándar. Por ejemplo: descubrir que todos los servicios que se conectan a Redis usan el mismo patrón de reintento, o que ciertos microservicios siempre fallan juntos.

### 2. Análisis de Correlación en Logs
Encuentra patrones temporales en logs de aplicación que predicen fallos. Descubre secuencias como "alta memoria → pausa de GC → timeout → fallo en cascada" que ocurren 15-45 minutos antes de los incidentes.

### 3. Detección de Anomalías de Rendimiento
Establece el comportamiento base a partir de métricas históricas y marca valores atípicos estadísticos. Correlaciona picos de CPU con patrones de consulta específicos o detecta que cada migración de base de datos coincide con un aumento de 3x en las tasas de error.

## Alcance

### Comandos

- `pattern-forager scan <target>` - Escanea una codebase o conjunto de datos para encontrar patrones
  - `--depth <int>`: Profundidad de análisis (1-5, por defecto: 3)
  - `--min-support <float>`: Frecuencia mínima de patrón (0.0-1.0, por defecto: 0.1)
  - `--output <format>`: json, yaml, html, csv (por defecto: json)
  - `--type`: code, logs, metrics, hybrid (por defecto: auto-detect)
  - `--time-window <duration>`: Para análisis temporal (ej. "7d", "24h")
  - `--correlation-threshold <float>`: Coeficiente de correlación mínimo (por defecto: 0.7)

- `pattern-forager correlate <source1> <source2>` - Encontrar correlaciones entre dos conjuntos de datos
  - `--method`: pearson, spearman, kendall, mutual_info
  - `--lag <int>`: Permitir correlaciones con desfase temporal (para datos temporales)
  - `--visualize`: Generar mapa de calor de correlación

- `pattern-forager anomalies <dataset>` - Detectar valores atípicos estadísticos
  - `--sensitivity <float>`: Umbral de puntuación Z (por defecto: 3.0)
  - `--seasonality`: Considerar patrones periódicos (diario, semanal)
  - `--context <file>`: Proporcionar señales contextuales adicionales

- `pattern-forager fingerprint <target>` - Generar huella de comportamiento
  - `--window <duration>`: Ventana deslizante para huella (por defecto: "1h")
  - `--features <list>`: Características específicas para huella (separadas por comas)
  - `--compare <reference>`: Comparar contra huella de referencia

- `pattern-forager export <pattern-id> <format>` - Exportar hallazgos de patrón específicos
  - `format`: markdown, json, graphml, png
  - `--include-samples`: Incluir muestras de datos brutos
  - `--confidence <float>`: Nivel de confianza mínimo (por defecto: 0.8)

### Variables de Entorno

- `PATTERN_FORAGER_CACHE_DIR`: Directorio de caché para resultados de análisis (por defecto: `~/.cache/pattern-forager`)
- `PATTERN_FORAGER_MAX_MEMORY`: Límite de memoria en MB (por defecto: 2048)
- `PATTERN_FORAGER_SAMPLE_RATE`: Tasa de muestreo para grandes conjuntos de datos (por defecto: 1.0)
- `PATTERN_FORAGER_N_JOBS`: Trabajos paralelos para computación (por defecto: recuento de CPU)
- `PATTERN_FORAGER_RANDOM_SEED`: Semilla aleatoria para análisis reproducible (por defecto: 42)

## Proceso de Trabajo

```bash
# Ejemplo: Encontrar patrones en logs de aplicación
pattern-forager scan /var/log/app --type logs --time-window 30d --min-support 0.05

# Ejemplo: Correlacionar métricas de base de datos con tasas de error
pattern-forager correlate metrics/db_latency.json metrics/errors.json --method spearman --lag 300 --visualize

# Ejemplo: Detectar anomalías en tiempos de respuesta de API
pattern-forager anomalies metrics/api_response_times.csv --sensitivity 2.5 --seasonality daily

# Ejemplo: Generar huella de comportamiento normal
pattern-forager fingerprint /var/log/app --window 24h --features response_time,error_rate,memory_usage > fingerprint_baseline.json

# Ejemplo: Exportar patrón como visualización
pattern-forager export pattern_id_42 graphml --include-samples > pattern_42.graphml
```

### Pasos Detallados

1. **Validación previa al escaneo**
   ```bash
   pattern-forager validate /path/to/target
   ```
   - Verificar compatibilidad de formato de archivo
   - Verificar integridad de datos
   - Asegurar volumen mínimo de datos (requiere al menos 1000 registros o 1000 líneas de código)

2. **Extracción de patrones**
   - Cargar y normalizar datos
   - Calcular matriz de características
   - Aplicar minería de patrones frecuentes (Apriori/FP-Growth)
   - Ejecutar análisis de correlación
   - Detectar estructuras de agrupación

3. **Post-procesamiento**
   - Filtrar por umbrales de significancia
   - Fusionar patrones similares
   - Asignar puntuaciones de confianza
   - Generar explicaciones legibles

4. **Generación de salida**
   - Crear informe estructurado
   - Generar visualizaciones si se solicita
   - Exportar muestras brutas si se marca
   - Calcular ideas accionables

## Reglas de Oro

1. **Nunca confíes en los datos sin validación**: Siempre ejecuta `pattern-forager validate` antes del análisis. Datos corruptos o con formato incorrecto producen patrones falsos.

2. **Ajusta la sensibilidad por dominio**: Para métricas financieras usa `--correlation-threshold 0.9`; para datos experimentales usa `0.6`. El valor por defecto (0.7) es para análisis de propósito general.

3. **Los patrones temporales requieren ventana suficiente**: El mínimo `--time-window` es 3 veces el período de patrón esperado. Para estacionalidad semanal, usa al menos 21 días de datos.

4. **El contexto es obligatorio para anomalías**: Siempre combina el comando `anomalies` con un archivo `--context` que contenga ventanas de mantenimiento conocidas, despliegues o eventos externos.

5. **La tasa de muestreo importa**: Para conjuntos de datos >1M de registros, reduce `PATTERN_FORAGER_SAMPLE_RATE` a 0.1-0.5 para mantener el rendimiento preservando la significancia del patrón.

6. **Correlación ≠ causalidad**: Pattern Forager solo encuentra relaciones estadísticas. Nunca actives acciones automáticamente basándote en correlaciones sin validación de dominio.

7. **Versiona tus huellas**: Las huellas generadas deben ser confirmadas al repositorio. `fingerprint_baseline.json` es un artefacto crítico.

8. **Los umbrales de memoria son límites estrictos**: Si el análisis excede `PATTERN_FORAGER_MAX_MEMORY`, el proceso se termina. Aumenta el límite o reduce `--depth` antes de reintentar.

9. **El soporte mínimo previene ruido**: Los patrones que aparecen en <10% de los datos (por defecto) son probablemente espurios. Aumenta `--min-support` para resultados más limpios.

10. **Valida cruzadamente con conocimiento de dominio**: Siempre revisa los patrones contra diagramas de arquitectura del sistema, manifiestos de despliegue y líneas de tiempo de incidentes.

## Ejemplos

### Ejemplo 1: Encontrar Microservicios Acoplados

```bash
# Entrada: Datos de tracing distribuido (JSON)
pattern-forager scan /data/traces/2024-03/ --type code --min-support 0.15 --depth 4
```

**Salida (extracto)**:
```json
{
  "patterns": [
    {
      "id": "pat_7f3a1b",
      "type": "service_dependency_cycle",
      "confidence": 0.94,
      "description": "Circular dependency detected: service-A → service-B → service-cache → redis → service-A",
      "affected_services": ["service-A", "service-B"],
      "sample_count": 142,
      "samples": [
        {"trace_id": "x7f9a1b", "sequence": ["A→B", "B→C", "C→Redis", "Redis→A"]}
      ],
      "recommendation": "Break cycle by having service-A bypass redis for cache writes, or introduce async refresh"
    }
  ]
}
```

### Ejemplo 2: Predecir Incidentes desde Logs

```bash
pattern-forager correlate logs/error.json logs/metrics.json --method mutual_info --lag 1800
```

**Salida**:
```json
{
  "correlations": [
    {
      "source": "cache_miss_rate",
      "target": "timeout_errors",
      "coefficient": 0.87,
      "lag_seconds": 420,
      "p_value": 0.001,
      "interpretation": "Cache miss rate spikes precede timeout errors by ~7 minutes",
      "sample_periods": 89
    }
  ]
}
```

### Ejemplo 3: Detectar Degradación de Rendimiento Después de Despliegue

```bash
# Antes del despliegue (línea base)
pattern-forager fingerprint /metrics/prod-2024-03-01.json --window 24h > baseline.json

# Después del despliegue
pattern-forager fingerprint /metrics/prod-2024-03-15.json --window 24h --compare baseline.json > comparison.json
```

**comparison.json**:
```json
{
  "fingerprint": {
    "p95_response_time": {"baseline": 120, "current": 340, "change_pct": 183},
    "error_rate": {"baseline": 0.001, "current": 0.015, "change_pct": 1400}
  },
  "anomaly_score": 0.91,
  "verdict": "SIGNIFICANT_DEGRADATION"
}
```

## Comandos de Reversión

Si el descubrimiento de patrones activa automatización no deseada o alertas:

1. **Deshabilitar alerta basada en patrón** (si está integrado con monitoreo):
   ```bash
   # Eliminar regla de alerta correlacionada
   curl -X DELETE https://api.monitoring/v1/rules/rule_pattern_7f3a1b
   ```

2. **Restaurar huella de referencia anterior**:
   ```bash
   git checkout HEAD~1 fingerprint_baseline.json
   pattern-forager validate fingerprint_baseline.json
   ```

3. **Limpiar caché de patrones** (si el análisis fue cacheado):
   ```bash
   rm -rf $(PATTERN_FORAGER_CACHE_DIR)/pat_*
   ```

4. **Revertir cambios de código auto-generados** (si los patrones activaron refactorización):
   ```bash
   git revert --no-commit <commit-hash>
   # Revisar cambios, luego commit
   git commit -m "Rollback pattern-forager auto-refactor"
   ```

5. **Deshabilitar escaneo de patrón programado** (si se usa cron/systemd):
   ```bash
   # Para cron
   crontab -l | grep -v pattern-forager | crontab -
   
   # Para systemd
   systemctl disable pattern-forager-scanner.timer
   systemctl stop pattern-forager-scanner.timer
   ```

6. **Restaurar tasa de muestreo original** (si el rendimiento se degradó):
   ```bash
   # Resetear variable de entorno a por defecto
   export PATTERN_FORAGER_SAMPLE_RATE=1.0
   # O actualizar en archivo de servicio systemd y recargar
   systemctl daemon-reload
   ```

## Pasos de Verificación

Después de instalar Pattern Forager:

```bash
# 1. Verificar dependencias
pattern-forager --version
# Esperado: Pattern Forager 1.2.0

# 2. Validar instalación
python -c "import pattern_forager; print(pattern_forager.__version__)"
# Esperado: 1.2.0

# 3. Ejecutar prueba de humo con datos de ejemplo
pattern-forager scan tests/sample_data/logs.jsonl --type logs --min-support 0.1
# Código de salida esperado: 0
# Esperado: Salida JSON con array "patterns" (puede estar vacío)

# 4. Verificar backend de visualización
pattern-forager scan tests/sample_data/metrics.csv --visualize --output png
# Esperado: Crea `patterns_visualization.png`

# 5. Probar método de correlación
pattern-forager correlate tests/sample_data/series_a.json tests/sample_data/series_b.json --method pearson
# Esperado: JSON con array "correlations", coeficiente entre -1 y 1

# 6. Verificar permisos de caché
touch $(PATTERN_FORAGER_CACHE_DIR)/test.tmp && rm $(PATTERN_FORAGER_CACHE_DIR)/test.tmp
# Esperado: Sin errores de permisos

# 7. Verificar límites de memoria (simular gran conjunto de datos)
pattern-forager scan tests/large_dataset/ --max-memory 1024
# Esperado: O bien completa o sale con mensaje de error de memoria claro
```

## Solución de Problemas

| Síntoma | Causa Probable | Solución |
|---------|----------------|----------|
| `MemoryError` durante escaneo | Conjunto de datos demasiado grande para memoria por defecto | Configurar `PATTERN_FORAGER_MAX_MEMORY` más alto o usar `--sample-rate 0.5` |
| No se encontraron patrones | `--min-support` muy alto | Bajar a 0.01-0.05 para análisis exploratorio |
| `ValueError: time window too small` | Datos insuficientes para análisis temporal | Aumentar `--time-window` a al menos 3x el período de patrón esperado |
| Coeficientes de correlación siempre 0 | Datos no estacionarios sin diferenciación | Usar `--method spearman` en lugar de pearson, o pre-diferenciar datos |
| Rendimiento lento en escaneo de código | `--depth` > 4 en codebase grande | Reducir profundidad o configurar `PATTERN_FORAGER_N_JOBS` a 1 para menor sobrecarga de CPU |
| Sin espacio en disco en caché | Acumulación de caché | Ejecutar `pattern-forager cache clean --older-than 7d` o configurar `PATTERN_FORAGER_CACHE_DIR` a tmpfs |
| `ImportError: No module named sklearn` | Dependencias faltantes | Instalar: `pip install scikit-learn` |
| Visualización produce PNG vacío | Todas las puntuaciones de patrón bajo umbral | Bajar `--min-support` o verificar que los datos contienen variación significativa |
| `Permission denied` en dir de caché | Propiedad incorrecta de directorio | `mkdir -p ~/.cache/pattern-forager && chmod 755 ~/.cache/pattern-forager` |

## Uso Avanzado

### Integración en Pipeline

```bash
# Pipe resultados de patrón a sistema de tickets
pattern-forager scan /data --output json | jq -r '.patterns[] | "\(.id)|\(.description)|\(.confidence)"' |
while IFS='|' read id desc conf; do
  if (( $(echo "$conf > 0.85" | bc -l) )); then
    curl -X POST https://tickets.api/create \
      -d "title=Pattern detected: $desc" \
      -d "body=Confidence: $conf\nID: $id" \
      -d "priority=high"
  fi
done
```

### Descubrimiento de Patrones Programado

Crear timer systemd:

`/etc/systemd/system/pattern-forager-scanner.service`:
```ini
[Unit]
Description=Daily pattern discovery scan
After=network.target

[Service]
Type=oneshot
Environment="PATTERN_FORAGER_OUTPUT_DIR=/var/patterns"
ExecStart=/usr/local/bin/pattern-forager scan /var/log/app --type logs --time-window 7d --output json --min-support 0.05
StandardOutput=append:/var/log/pattern-forager/scans.log
StandardError=journal
```

`/etc/systemd/system/pattern-forager-scanner.timer`:
```ini
[Unit]
Description=Run pattern-forager scanner daily at 2 AM

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

Habilitar:
```bash
systemctl enable pattern-forager-scanner.timer
systemctl start pattern-forager-scanner.timer
```

## Soporte

- Documentación: https://github.com/smouj/openclaw-skills/pattern-forager/docs
- Issues: https://github.com/smouj/openclaw-skills/pattern-forager/issues
- Chat: `#pattern-forager` en OpenClaw Slack
```