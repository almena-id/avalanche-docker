# Private Avalanche Network With 5 Full Nodes + 1 Validator

This project spins up a private Avalanche network using five full AvalancheGo instances plus an additional validator node, all managed by Docker Compose. It reuses the genesis file and staking keys that Avalanche Labs publishes for the public “local” network, but the cluster runs with a custom `network-id` (1337) so the bundled genesis can be loaded without conflicting with the standard presets.

## Requirements

- Docker 20.10+ and Docker Compose v2 (CLI or plugin).
- At least 8 GB of available RAM and ~10 GB of disk space for node data.

## Getting Started

1. Clone this repository and switch into the directory.
2. (Optional) Edit `.env` to bump `avaplatform/avalanchego` to a different release.
3. Start the network:

   ```bash
   docker compose up -d
   ```

4. Check the nodes by tailing logs or calling the Info API, for example:

   ```bash
   curl -s http://localhost:9650/ext/info \
     -X POST \
     -H 'content-type: application/json' \
     -d '{"jsonrpc":"2.0","id":1,"method":"info.getNodeID"}'
   ```

   Repeat with ports `9652`, `9654`, `9656`, `9658` for the other full nodes and `9660` for the validator.

5. Stop every container:

   ```bash
   docker compose down
   ```

6. For a clean reset, delete the folders under `data/` before starting the services again.

## What Does Each Node Expose?

| Node      | HTTP | Staking | NodeID                                     |
|-----------|------|---------|--------------------------------------------|
| node1     | 9650 | 9651    | NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg   |
| node2     | 9652 | 9653    | NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ   |
| node3     | 9654 | 9655    | NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN   |
| node4     | 9656 | 9657    | NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu   |
| node5     | 9658 | 9659    | NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5   |
| validator | 9660 | 9661    | NodeID-MgiveP2p1Gni7jXWFG2UBF2aKesw4eq4R   |

All services mount `artifacts/genesis/genesis.json` (the official local genesis) plus the TLS/BLS keys stored in `artifacts/staking/node*/`. The network ID is set to `1337`, so it never collides with the built-in IDs. These keys **must only be used for local or lab environments**.

Each container owns a static IP inside the Compose bridge (`172.28.0.11`–`172.28.0.30`) so the `bootstrap-ips` values always reference valid addresses. The dedicated `validator` service is the only one mounting a BLS signer key, so it actively participates in consensus while the rest operate as full (non-validating) nodes.

## Project Layout

```
artifacts/
  genesis/genesis.json       # Avalanche local genesis
  staking/node*/             # TLS (staker.crt/key) and BLS (signer.key) files for full nodes
  staking/validator/         # TLS/BLS bundle for the dedicated validator
config/node*/config.json     # Full node configs
config/validator/config.json # Validator parameters
 data/node*/                 # Persistent DB and log folders for the 5 full nodes
 data/validator/             # Validator DB/logs
 docker-compose.yml          # Orchestrates all containers
 .env                        # AvalancheGo version + Compose project name
 README.md
```

## Quick Customization

- Adjust ports or other params per node in `config/nodeX/config.json` before launching.
- Add validators by duplicating a block in `docker-compose.yml`, provisioning new keys, and updating the genesis to include the new NodeID.
- To test upgrades, change `AVALANCHEGO_VERSION` in `.env` and rerun `docker compose up -d`.

## Notes

- C-Chain and other chain data live in `data/node*/db`; expect the footprint to grow as you run tests.
- Default RPC endpoints are accessible from the host (for example `http://localhost:9650/ext/bc/C/rpc` for node1’s C-Chain).
- If you expose nodes outside your machine, set `public-ip` in each `config.json` to the appropriate public address.
