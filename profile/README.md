# CineThread

A movie trivia game to create the longest chain possible of connected actors and movies.

## Repos

| Repo | Stack | Role |
|---|---|---|
| `cinethread-infra` | Docker, Postgres, Caddy | Shared database, network, and reverse proxy |
| `cinethread-import` | Go | One-shot job, imports movie and actor data and stores using `cinethread-api` |
| `cinethread-api` | Go, Postgres | Owns the schema/migrations, serves the API |
| `cinethread-web` | Vue | Web app, calls `cinethread-api` |
| `cinethread-mobile` | React Native | App, calls `cinethread-api` over the public internet |

## How it links

- `cinethread-infra` owns the single Postgres instance and the `cinethread-net`
  Docker network.
- `cinethread-import` and `cinethread-api` attach to `cinethread-net` to reach
  Postgres by hostname (`db`).
- `cinethread-api` is the only repo that runs migrations — `cinethread-import` just
  writes into existing tables.
- `cinethread-web` and `cinethread-mobile` don't touch Postgres directly; they go through
  `cinethread-api`'s HTTP API.
- One Caddy instance (in `cinethread-infra`) owns ports 80/443 and routes each public
  domain to the right service.

## Deploy order

1. `cinethread-infra` (must be running first — everything else depends on it)
2. `cinethread-api` (runs migrations)
3. `cinethread-import`, `cinethread-web`
