# fty-alert-stats

Agent fty-alert-stats computes metric statistics on alerts.

## How to build

To build fty-alert-stats project run:

```bash
./autogen.sh
./configure
make
make check # to run self-test
```

## How to run

To run fty-alert-stats project:

* from within the source tree, run:

```bash
./src/fty-alert-stats
```

For the other options available, refer to the manual page of fty-alert-stats

* from an installed base, using systemd, run:

```bash
systemctl start fty-alert-stats
```

### Configuration file

Configuration file - fty-alert-stats.cfg - is currently ignored.

## Architecture

### Overview

fty-alert-stats is composed of 1 actor:

* fty-alert-stats: main actor

The agent keeps a list of alerts and assets, periodically resynchronized with
the system (every 12 hours). The agent publishes metrics (TTL of 12 minutes),
periodically refreshed as needed to keep them alive.

## Protocols

### Published metrics

Agent publishes `alerts.warning@<asset>` and `alerts.critical@<asset>` metrics,
where each metric is a count of all active alerts on the asset (and, if
applicable, all child assets combined).

### Published alerts

Agent does not publish alerts.

### Mailbox requests

Agent will republish all metrics when receiving mailbox message with `REPUBLISH`
subject. Agent will reply with subject `REPUBLISH` and payload:
 * "OK": metrics were republished.
 * "RESYNC": agent is currently resyncing data, metrics will be republished when resync is done.

### Stream subscriptions

Agent is subscribed to ALERTS and ASSETS streams and publishes to METRICS.

Additionally, the agent will query fty-alert-list and asset-agent (fty-asset)
on startup and periodically thereafter to synchronize itself with the rest of
the system. Failure to synchronize is not fatal, but the agent will not be able
to compute meaningful statistics before its first successful synchronization.
