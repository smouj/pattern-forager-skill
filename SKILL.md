---
name: Pattern Forager
version: 1.2.0
description: Discovers hidden patterns and correlations in complex data and codebases through statistical analysis, temporal correlation detection, and behavioral fingerprinting.
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
---

# Pattern Forager

Automatic discovery of hidden patterns, correlations, and anomalies in complex datasets and codebases.

## Purpose

Pattern Forager serves three primary use cases:

### 1. Codebase Pattern Detection
Identifies recurring architectural patterns, code cloning hotspots, and module coupling that aren't visible through standard metrics. For example: discover that all services connecting to Redis use the same retry pattern, or that certain microservices always fail together.

### 2. Log Correlation Analysis
Finds temporal patterns in application logs that predict failures. Discovers sequences like "high memory → GC pause → timeout → cascade failure" that happen 15-45 minutes before incidents.

### 3. Performance Anomaly Detection
Establishes baseline behavior from historical metrics and flags statistical outliers. Correlates CPU spikes with specific query patterns or detects that every database migration coincides with a 3x increase in error rates.

## Scope

### Commands

- `pattern-forager scan <target>` - Scan a codebase or dataset for patterns
  - `--depth <int>`: Analysis depth (1-5, default: 3)
  - `--min-support <float>`: Minimum pattern frequency (0.0-1.0, default: 0.1)
  - `--output <format>`: json, yaml, html, csv (default: json)
  - `--type`: code, logs, metrics, hybrid (default: auto-detect)
  - `--time-window <duration>`: For temporal analysis (e.g., "7d", "24h")
  - `--correlation-threshold <float>`: Minimum correlation coefficient (default: 0.7)

- `pattern-forager correlate <source1> <source2>` - Find correlations between two datasets
  - `--method`: pearson, spearman, kendall, mutual_info
  - `--lag <int>`: Allow time-shifted correlations (for temporal data)
  - `--visualize`: Generate correlation heatmap

- `pattern-forager anomalies <dataset>` - Detect statistical outliers
  - `--sensitivity <float>`: Z-score threshold (default: 3.0)
  - `--seasonality`: Consider periodic patterns (daily, weekly)
  - `--context <file>`: Provide additional contextual signals

- `pattern-forager fingerprint <target>` - Generate behavioral fingerprint
  - `--window <duration>`: Rolling window for fingerprint (default: "1h")
  - `--features <list>`: Specific features to fingerprint (comma-separated)
  - `--compare <reference>`: Compare against baseline fingerprint

- `pattern-forager export <pattern-id> <format>` - Export specific pattern findings
  - `format`: markdown, json, graphml, png
  - `--include-samples`: Include raw data samples
  - `--confidence <float>`: Minimum confidence level (default: 0.8)

### Environment Variables

- `PATTERN_FORAGER_CACHE_DIR`: Cache directory for analysis results (default: `~/.cache/pattern-forager`)
- `PATTERN_FORAGER_MAX_MEMORY`: Memory limit in MB (default: 2048)
- `PATTERN_FORAGER_SAMPLE_RATE`: Sampling rate for large datasets (default: 1.0)
- `PATTERN_FORAGER_N_JOBS`: Parallel jobs for computation (default: CPU count)
- `PATTERN_FORAGER_RANDOM_SEED`: Random seed for reproducible analysis (default: 42)

## Work Process

```bash
# Example: Find patterns in application logs
pattern-forager scan /var/log/app --type logs --time-window 30d --min-support 0.05

# Example: Correlate database metrics with error rates
pattern-forager correlate metrics/db_latency.json metrics/errors.json --method spearman --lag 300 --visualize

# Example: Detect anomalies in API response times
pattern-forager anomalies metrics/api_response_times.csv --sensitivity 2.5 --seasonality daily

# Example: Generate fingerprint of normal behavior
pattern-forager fingerprint /var/log/app --window 24h --features response_time,error_rate,memory_usage > fingerprint_baseline.json

# Example: Export pattern as visualization
pattern-forager export pattern_id_42 graphml --include-samples > pattern_42.graphml
```

### Detailed Steps

1. **Pre-scan validation**
   ```bash
   pattern-forager validate /path/to/target
   ```
   - Check file format compatibility
   - Verify data integrity
   - Ensure minimum data volume (requires at least 1000 records or 1000 lines of code)

2. **Pattern extraction**
   - Load and normalize data
   - Compute feature matrix
   - Apply frequent pattern mining (Apriori/FP-Growth)
   - Run correlation analysis
   - Detect clustering structures

3. **Post-processing**
   - Filter by significance thresholds
   - Merge similar patterns
   - Assign confidence scores
   - Generate human-readable explanations

4. **Output generation**
   - Create structured report
   - Generate visualizations if requested
   - Export raw samples if flagged
   - Calculate actionable insights

## Golden Rules

1. **Never trust data without validation**: Always run `pattern-forager validate` before analysis. Corrupted or malformed data produces false patterns.

2. **Adjust sensitivity by domain**: For financial metrics use `--correlation-threshold 0.9`; for experimental data use `0.6`. Default (0.7) is for general-purpose analysis.

3. **Temporal patterns require sufficient window**: Minimum `--time-window` is 3x the expected pattern period. For weekly seasonality, use at least 21 days of data.

4. **Context is mandatory for anomalies**: Always pair `anomalies` command with `--context` file containing known maintenance windows, deployments, or external events.

5. **Sample rate matters**: For datasets >1M records, reduce `PATTERN_FORAGER_SAMPLE_RATE` to 0.1-0.5 to maintain performance while preserving pattern significance.

6. **Correlation ≠ causation**: Pattern Forager finds statistical relationships only. Never auto-trigger actions based on correlations without domain validation.

7. **Version control your fingerprints**: Generated fingerprints must be committed to the repository. `fingerprint_baseline.json` is a critical artifact.

8. **Memory thresholds are hard limits**: If analysis exceeds `PATTERN_FORAGER_MAX_MEMORY`, the process terminates. Increase limit or reduce `--depth` before retrying.

9. **Minimum support prevents noise**: Patterns appearing in <10% of data (default) are likely spurious. Increase `--min-support` for cleaner results.

10. **Cross-validate with domain knowledge**: Always review patterns against system architecture diagrams, deployment manifests, and incident timelines.

## Examples

### Example 1: Finding Coupled Microservices

```bash
# Input: Distributed tracing data (JSON)
pattern-forager scan /data/traces/2024-03/ --type code --min-support 0.15 --depth 4
```

**Output (excerpt)**:
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

### Example 2: Predicting Incidents from Logs

```bash
pattern-forager correlate logs/error.json logs/metrics.json --method mutual_info --lag 1800
```

**Output**:
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

### Example 3: Detecting Performance Degradation After Deploy

```bash
# Before deployment (baseline)
pattern-forager fingerprint /metrics/prod-2024-03-01.json --window 24h > baseline.json

# After deployment
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

## Rollback Commands

If pattern discovery triggers unwanted automation or alerts:

1. **Disable pattern-based alert** (if integrated with monitoring):
   ```bash
   # Remove correlating alert rule
   curl -X DELETE https://api.monitoring/v1/rules/rule_pattern_7f3a1b
   ```

2. **Restore previous fingerprint baseline**:
   ```bash
   git checkout HEAD~1 fingerprint_baseline.json
   pattern-forager validate fingerprint_baseline.json
   ```

3. **Clear pattern cache** (if analysis was cached):
   ```bash
   rm -rf $(PATTERN_FORAGER_CACHE_DIR)/pat_*
   ```

4. **Rollback auto-generated code changes** (if patterns triggered refactoring):
   ```bash
   git revert --no-commit <commit-hash>
   # Review changes, then commit
   git commit -m "Rollback pattern-forager auto-refactor"
   ```

5. **Disable scheduled pattern scan** (if using cron/systemd):
   ```bash
   # For cron
   crontab -l | grep -v pattern-forager | crontab -
   
   # For systemd
   systemctl disable pattern-forager-scanner.timer
   systemctl stop pattern-forager-scanner.timer
   ```

6. **Restore original sampling rate** (if performance degraded):
   ```bash
   # Reset environment variable to default
   export PATTERN_FORAGER_SAMPLE_RATE=1.0
   # Or update in systemd service file and reload
   systemctl daemon-reload
   ```

## Verification Steps

After installing Pattern Forager:

```bash
# 1. Check dependencies
pattern-forager --version
# Expected: Pattern Forager 1.2.0

# 2. Validate installation
python -c "import pattern_forager; print(pattern_forager.__version__)"
# Expected: 1.2.0

# 3. Run smoke test with sample data
pattern-forager scan tests/sample_data/logs.jsonl --type logs --min-support 0.1
# Expected exit code: 0
# Expected: JSON output with "patterns" array (may be empty)

# 4. Verify visualization backend
pattern-forager scan tests/sample_data/metrics.csv --visualize --output png
# Expected: Creates `patterns_visualization.png`

# 5. Test correlation method
pattern-forager correlate tests/sample_data/series_a.json tests/sample_data/series_b.json --method pearson
# Expected: JSON with "correlations" array, coefficient between -1 and 1

# 6. Check cache permissions
touch $(PATTERN_FORAGER_CACHE_DIR)/test.tmp && rm $(PATTERN_FORAGER_CACHE_DIR)/test.tmp
# Expected: No permission errors

# 7. Verify memory limits (simulate large dataset)
pattern-forager scan tests/large_dataset/ --max-memory 1024
# Expected: Either completes or exits with clear memory error message
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `MemoryError` during scan | Dataset too large for default memory | Set `PATTERN_FORAGER_MAX_MEMORY` higher or use `--sample-rate 0.5` |
| No patterns found | `--min-support` too high | Lower to 0.01-0.05 for exploratory analysis |
| `ValueError: time window too small` | Insufficient data for temporal analysis | Increase `--time-window` to at least 3x expected pattern period |
| Correlation coefficients always 0 | Non-stationary data without differencing | Use `--method spearman` instead of pearson, or pre-difference data |
| Slow performance on code scan | `--depth` > 4 on large codebase | Reduce depth or set `PATTERN_FORAGER_N_JOBS` to 1 for lower CPU overhead |
| Out of disk space in cache | Cache accumulation | Run `pattern-forager cache clean --older-than 7d` or set `PATTERN_FORAGER_CACHE_DIR` to tmpfs |
| `ImportError: No module named sklearn` | Missing dependencies | Install: `pip install scikit-learn` |
| Visualization produces empty PNG | All pattern scores below threshold | Lower `--min-support` or check data contains meaningful variation |
| `Permission denied` on cache dir | Wrong directory ownership | `mkdir -p ~/.cache/pattern-forager && chmod 755 ~/.cache/pattern-forager` |

## Advanced Usage

### Pipeline Integration

```bash
# Pipe pattern results to ticketing system
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

### Scheduled Pattern Discovery

Create systemd timer:

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

Enable:
```bash
systemctl enable pattern-forager-scanner.timer
systemctl start pattern-forager-scanner.timer
```

## Support

- Documentation: https://github.com/smouj/openclaw-skills/pattern-forager/docs
- Issues: https://github.com/smouj/openclaw-skills/pattern-forager/issues
- Chat: `#pattern-forager` on OpenClaw Slack
```