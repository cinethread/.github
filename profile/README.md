# Movie Clash

A movie trivia game to create the longest chain possible of connected actors and movies.

## Repos

| Repo | Stack | Role |
|---|---|---|
| `movieclash-infra` | Docker, Postgres, Caddy | Shared database, network, and reverse proxy |
| `movieclash-import` | Go | One-shot job, imports movie and actor data into Postgres |
| `movieclash-api` | Go, Postgres | Owns the schema/migrations, serves the API |
| `movieclash-web` | Vue | Web app, calls `movieclash-api` |
| `movieclash-mobile` | React Native | App, calls `movieclash-api` over the public internet |

## How it links

- `movieclash-infra` owns the single Postgres instance and the `movieclash-net`
  Docker network.
- `movieclash-import` and `movieclash-api` attach to `movieclash-net` to reach
  Postgres by hostname (`db`).
- `movieclash-api` is the only repo that runs migrations — `movieclash-import` just
  writes into existing tables.
- `movieclash-web` and `movieclash-mobile` don't touch Postgres directly; they go through
  `movieclash-api`'s HTTP API.
- One Caddy instance (in `movieclash-infra`) owns ports 80/443 and routes each public
  domain to the right service.

## Deploy order

1. `movieclash-infra` (must be running first — everything else depends on it)
2. `movieclash-api` (runs migrations)
3. `movieclash-import`, `movieclash-web`
