# Prowlarr Helm Chart

![Downloads](https://img.shields.io/github/downloads/tonylea/helm-chart-prowlarr/total)

Deploys [Prowlarr](https://prowlarr.com/) from LinuxServer.io's image, [`lscr.io/linuxserver/prowlarr`](https://github.com/linuxserver/docker-prowlarr), the one the Servarr wiki's Docker page lists, since the project publishes no image.

The chart is opinionated: Prowlarr keeps its data in Postgres, uses a fixed API key, and always asks for a login.
It runs one replica, and stops the old pod before starting the new one, since two copies of Prowlarr can't share a database.
Only `/config` is on a volume, holding `config.xml`, logs and backups.
It has no metrics of its own, so the chart has no ServiceMonitor.

## Values

| Value | Default |
| --- | --- |
| `image.tag` | the chart's `appVersion` |
| `uid` | `1000` |
| `gid` | `1000` |
| `timezone` | `UTC` |
| `apiKey.secretKeyRef.name` | none; required |
| `apiKey.secretKeyRef.key` | `api-key` |
| `postgres.host` | none; required |
| `postgres.port` | `5432` |
| `postgres.databases.main` | `prowlarr-main` |
| `postgres.databases.log` | `prowlarr-log` |
| `postgres.username.secretKeyRef.name` | none; required |
| `postgres.username.secretKeyRef.key` | `username` |
| `postgres.password.secretKeyRef.name` | none; required |
| `postgres.password.secretKeyRef.key` | `password` |
| `resources` | none |
| `persistence.config.existingClaim` | none; required |
| `httpRoute.enabled` | `false` |
| `httpRoute.parentRefs` | none; required when enabled |
| `httpRoute.hostnames` | none |

## Fixed settings

The chart sets these as environment variables, which override `config.xml`, so they can't be changed in the web UI:

- The API key, from `apiKey`'s Secret, so it is in code rather than generated into `config.xml`.
- Authentication method `Forms` and authentication required `Enabled`.
  Prowlarr parses enum values case-sensitively, so they are spelled exactly so.
  The username and password aren't settings: Prowlarr keeps them in its database and asks for them on first visit.
- The Postgres connection, from `postgres`.

The pod runs in LinuxServer.io's non-root mode: `uid` and `gid` as the user and group, `runAsNonRoot`, and an empty directory on `/run`.
In this mode the image's `PUID` and `PGID` do nothing and it no longer fixes ownership of `/config`, so the pod sets `fsGroup` to `gid`.
All capabilities are dropped and privilege escalation is off.

## Requirements

- Postgres 15 or later, with the main and log databases made beforehand, because Prowlarr doesn't create them.
  Prowlarr's housekeeping runs `VACUUM`, which needs a superuser, so it should connect as one.
  With CloudNativePG, that's its superuser Secret, which holds `username` and `password`.
- A Secret holding the API key, and one or two holding the Postgres user and password.
- A `ReadWriteOnce` claim for `/config`, whose storage class honours `fsGroup`, or whose folder is already writable by `uid`.
- The Gateway API's `HTTPRoute` CRD, when `httpRoute.enabled` is set.

## Usage

```sh
helm repo add prowlarr https://tonylea.github.io/helm-chart-prowlarr
helm install prowlarr prowlarr/prowlarr --values my-values.yaml
```

```yaml
timezone: Europe/London
apiKey:
  secretKeyRef:
    name: prowlarr-api-key
postgres:
  host: prowlarr-postgres-rw
  username:
    secretKeyRef:
      name: prowlarr-postgres-superuser
  password:
    secretKeyRef:
      name: prowlarr-postgres-superuser
persistence:
  config:
    existingClaim: prowlarr-config
```

## Development

Pull requests run:

- `helm lint --strict`, and the rendered chart checked by kubeconform against Kubernetes 1.37.
- helm-unittest's tests in `tests/`.
- An install with chart-testing in a kind cluster, against a throwaway Postgres and a claim for `/config` from `ci/kind`.

Commits follow Conventional Commits.
Releases are versioned and tagged automatically from `main`, and published to the Helm repository on this repo's GitHub Pages.
