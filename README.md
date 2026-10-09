# Monitoring preset

The whole monitoring stack for [my-sites-ide](https://github.com/yiendos/my-sites-ide) in one
`require`: a Composer metapackage that installs the five monitoring plugins.

Written for: developers running sites in my-sites-ide who want metrics, logs and traces without
picking the plugins one by one.

| Plugin | Service | Role |
|---|---|---|
| [monitoring-grafana](https://github.com/yiendos/my-sites-ide-monitoring-grafana) | `grafana` | the UI, http://localhost:3000 |
| [monitoring-prometheus](https://github.com/yiendos/my-sites-ide-monitoring-prometheus) | `prometheus` | metrics, http://localhost:9090 |
| [monitoring-loki](https://github.com/yiendos/my-sites-ide-monitoring-loki) | `loki` | log storage |
| [monitoring-tempo](https://github.com/yiendos/my-sites-ide-monitoring-tempo) | `tempo` | traces |
| [monitoring-alloy](https://github.com/yiendos/my-sites-ide-monitoring-alloy) | `alloy` | collector - container logs, and your apps' OpenTelemetry on `alloy:4318`; UI at http://localhost:12345 |

## Installation

Add the preset to the `require` section of the IDE's `composer.local.json`:

```json
"yiendos/my-sites-ide-preset-monitoring": "@dev"
```

Then, from the IDE root:

```
composer update
```

The preset only lists the plugins - it has no code or commands of its own. To pick and choose
instead, require the plugins you want individually; each works with whichever of the others are
installed.

The plugins aren't released yet, only branches, so the IDE has to accept dev versions of a
preset's dependencies: its root `composer.json` sets `"minimum-stability": "dev"` with
`"prefer-stable": true`, so everything with a stable release still gets one.

### Where the plugins come from

With the plugin repositories cloned into the IDE's `Packages/yiendos/`, the IDE's path repository
(`Packages/*/my-sites-ide-*`) finds them all. Without the clones, add the repositories to
`composer.local.json`:

```json
"repositories": [
    { "type": "vcs", "url": "git@github.com:yiendos/my-sites-ide-preset-monitoring.git" },
    { "type": "vcs", "url": "git@github.com:yiendos/my-sites-ide-monitoring-grafana.git" },
    { "type": "vcs", "url": "git@github.com:yiendos/my-sites-ide-monitoring-prometheus.git" },
    { "type": "vcs", "url": "git@github.com:yiendos/my-sites-ide-monitoring-loki.git" },
    { "type": "vcs", "url": "git@github.com:yiendos/my-sites-ide-monitoring-tempo.git" },
    { "type": "vcs", "url": "git@github.com:yiendos/my-sites-ide-monitoring-alloy.git" }
]
```

## Starting the stack

None of the plugins autostart. Any order works - Alloy, Tempo and Grafana write their config from
what's installed, not what's running - but starting the backends first saves a few connection
errors in the others' logs:

```
php my-sites-ide monitoring:prometheus-start
php my-sites-ide monitoring:loki-start
php my-sites-ide monitoring:tempo-start
php my-sites-ide monitoring:alloy-start
php my-sites-ide monitoring:grafana-start
```

Or add them to `APP` in the IDE's `.env` for `ide:spark` to start - after the start commands have
run once, since they write the configs the containers read.

Every IDE container's logs are in Loki straight away. For traces, point your apps' OpenTelemetry
exporter at Alloy:

```
OTEL_EXPORTER_OTLP_ENDPOINT=http://alloy:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_SERVICE_NAME=<your site>
```

Then open Grafana at http://localhost:3000 - **Explore** for logs, metrics and traces, with links
from a trace to its logs and metrics.

## Stopping it

```
php my-sites-ide monitoring:grafana-stop
php my-sites-ide monitoring:alloy-stop
php my-sites-ide monitoring:tempo-stop
php my-sites-ide monitoring:loki-stop
php my-sites-ide monitoring:prometheus-stop
```

Everything each plugin keeps is in `storage/plugins/<service>/`, so stopping or recreating the
containers loses nothing.
