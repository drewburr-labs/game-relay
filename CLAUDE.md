# CLAUDE.md

## Always commit and push

Claude runs on a remote development machine. The user can only test, review or try work once it is on GitHub, so:

- **Commit and push every finished piece of work without asking.** Don't stop at "want me to commit?" — just do it.
- Push straight to `main` on `origin`, not to a feature branch. From a worktree branch: `git fetch origin && git rebase origin/main && git push origin HEAD:main`.
- Never force-push or rewrite history that's already pushed unless the user asks.
- Pushing is the finish line. Don't open pull requests, or offer to, unless the user asks for one.

## Project

A room-code relay for online games: GDScript on a headless Godot 4.7, shipped as a container image. It isn't tied to any one game. See README.md for how games use it, how to run it, and its metrics.

- The scripts sit at the repo root and load each other by relative path, because games copy `relay_protocol.gd` and `relay_client.gd` into folders of their own. Those two must depend on nothing but each other.
- Nothing here may know about a particular game. Saucer Showdown (`drewburr-labs/saucer-showdown`) is one user: it keeps a copy of `relay_protocol.gd`, `relay_client.gd`, `relay_server.gd` and `relay_metrics.gd` in its `relay/` folder, refreshed from a tag of this repo by its `tools/update_relay.sh`. A change here reaches the game only after a new tag and that script.
- Bump `VERSION` in `relay_protocol.gd` only when the relay's wire format changes; clients with another version are refused, so games must update alongside the relay.
- Metrics (`relay_metrics.gd`, listed in README.md) are served on a private TCP port, never the games' UDP port, and must never name a room code or a player's address.
- A new server script needs adding to the Dockerfile's `COPY`.
- The image is `ghcr.io/drewburr-labs/game-relay`, pushed by `.github/workflows/image.yml` on every push to `main` that touches more than docs or `example/`.

## Commands

```bash
godot --headless --path . --import
godot --headless --path . --script tests/test_relay_metrics.gd
godot --headless --path . --script res://run_relay.gd -- --port=24680 --metrics-port=24681
godot --headless --path . --script res://relay_probe.gd -- 127.0.0.1:24680
docker build -t game-relay .
```

Run the tests before every commit. After a change to the server, also run it and point the probe at it. Tests have a `TIME_LIMIT` (see `tests/watchdog.gd`); don't raise one to make a slow test pass.
