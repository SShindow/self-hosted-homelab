# Self-Hosted Homelab

A personal homelab spread over **two sites in two countries**, linked by a **Tailscale** mesh with no port forwarding and nothing exposed to the internet. It replaces paid cloud services with infrastructure I own, and it's documented as an engineering project: every module records the goal, the decisions, what broke, and how it was verified.

```
        🇩🇪 Germany (Darmstadt)                              🇻🇳 Vietnam (Ho Chi Minh City)
┌───────────────────────────────┐                ┌───────────────────────────────┐
│ pve-de: Dell OptiPlex Micro   │                │ truenas-scale: repurposed PC  │
│ Proxmox VE                    │◄── Tailscale ──►│ TrueNAS CE, ZFS mirror        │
│ apps + VMs, smart home,       │   (WireGuard   │ storage, photos, media,       │
│ monitoring, backup target     │    mesh)       │ automation, game server       │
└───────────────────────────────┘                └───────────────────────────────┘
              ▲                                                 ▲
              └──────── laptop · phone · tablet · desktop ──────┘
                         (any network, anywhere)
```

Each site does what it's best placed to do. **Storage stays in Vietnam**, where the disks are and electricity is cheap. **Latency-sensitive and always-on services live in Germany**, close to where I am. Each site watches and backs up the other. They're two independent servers, deliberately not a cluster: if one site goes down, the other keeps running.

## Sites

### 🇻🇳 [Vietnam Node: TrueNAS NAS](docs/vietnam-node-truenas.md)
An old desktop PC (Pentium G3240, 16 GB, 2 × 2 TB ZFS mirror) at the family home, administered entirely remotely. It runs file sharing, photo backup (Immich), media streaming and automation (Jellyfin and the arr-stack), observability (Prometheus + Grafana) and a Minecraft server for friends in four countries. → [Node page](docs/vietnam-node-truenas.md)

### 🇩🇪 [Germany Node: Proxmox VE](docs/germany-node-proxmox.md)
A used Dell OptiPlex 5060 Micro (i5-8500T, 16 GB), validated before use and running Proxmox VE. It's the home for apps and VMs: smart home, monitoring of the NAS from outside, and cross-site backups. Assembly into a 10" rack with a managed switch is pending. → [Node page](docs/germany-node-proxmox.md)

## Modules

Each module is a self-contained build with its own goals, decisions, problems hit, and verification.

### Vietnam node

| Module | What it does | Status |
|---|---|---|
| **[Google Drive → NAS Backup](docs/google-drive-backup.md)** | One-way (Pull + Copy) Cloud Sync so cloud-side deletions can't propagate to the local copy. | ✅ Operational |
| **[Power-Loss Resilience](docs/power-loss-resilience.md)** | Root-caused a failing CMOS battery that silently reset the BIOS auto-power-on setting; verified with a real fuse cut. | ✅ Verified |
| **[ZFS Snapshot Strategy](docs/zfs-snapshots.md)** | Daily and weekly periodic snapshots for instant local point-in-time recovery, layered with the off-site cloud copy. | ✅ Operational |
| **[Tailscale Exit Node](docs/tailscale-exit-node.md)** | The NAS as a personal VPN egress, plus a subnet router extension reaching the whole home LAN remotely. | ✅ Verified |
| **[Camera NVR (Frigate)](docs/camera-nvr-frigate.md)** | Tapo C200 RTSP recorded to the ZFS pool with no vendor cloud, plus end-to-end performance benchmarking. Retired once two-way audio outweighed local recording. | ⏹️ Decommissioned |
| **[Observability](docs/monitoring-prometheus-grafana.md)** | Prometheus + Grafana with a host-installed node_exporter, persistent host-path config, reboot-verified. | ✅ Operational |
| **[Media Streaming](docs/media-jellyfin.md)** | Jellyfin serving a growing library from the ZFS pool, with transcoding behaviour and real remote-throughput limits measured. | ✅ Operational |
| **[Media Automation (arr-stack)](docs/arr-stack.md)** | Seerr → Prowlarr → Sonarr/Radarr/Lidarr → download client → Bazarr request-to-playback pipeline, plus Wizarr/Cleanuparr for friend access. See the module's own disclaimer. | ✅ Operational (Lidarr paused) |
| **[Dashboard (Homepage)](docs/homepage.md)** | One YAML-configured page reaching every service on the box, with live widgets, working identically from the LAN or from Germany. | ✅ Operational |
| **[Photo Management](docs/photos-immich.md)** | Immich replacing a paid 50 GB cloud photo tier — mobile auto-backup, a library that stays readable without the app, and a checksum-verified off-site copy. | ✅ Operational |
| **[Game Server (Minecraft)](docs/game-server-minecraft.md)** | A modded Forge 1.20.1 server for friends in four countries, sized to a 2-core CPU without AVX — three builds, a memory rebalance, and an ISP-route finding to get there. | ✅ Operational |

### Germany node

| Module | What it does | Status |
|---|---|---|
| **[Proxmox VE base](docs/germany-node-proxmox.md)** | A used OptiPlex Micro validated (SMART, MemTest), Proxmox VE on ext4, no-subscription updates, fixed IP, Tailscale with tagged non-expiring keys, TOTP 2FA. | ✅ Operational (rack assembly pending) |
| Uptime Kuma | Up/down checks and alerts for both sites, run from the *other* site so a dead NAS can still be reported. | ⏳ Next |
| Home Assistant | Smart home and the planned room assistant, migrated off a Hyper-V VM on the desktop. | ⏳ Planned |
| Cross-site backups | A 2.5" SSD as a ZFS target for replication from the NAS, plus Germany → Vietnam copies. | ⏳ Planned |

### Cross-site

| Topic | Where |
|---|---|
| Tailscale mesh, exit node and subnet router | [Tailscale Exit Node](docs/tailscale-exit-node.md) |
| Germany → Vietnam route quality: Telekom and O2 lose 6–10 %, Vodafone 0 % | [Game Server](docs/game-server-minecraft.md), [Germany Node](docs/germany-node-proxmox.md#cross-site-link-test-vodafone--vnpt) |
| Service names: `https://<service>.<tailnet>.ts.net` for every web UI via Tailscale Services, no domain, no open ports | [Service Naming](docs/service-naming-tailscale.md) |
| Key expiry and tagging for headless nodes | [Germany Node](docs/germany-node-proxmox.md#remote-access-tailscale-on-the-host) |

**[Troubleshooting Playbook](docs/troubleshooting.md)**: recurring problem patterns collected across every module, grouped by root cause rather than by which module happened to surface each one first.

## Highlights

- **A real disk failure handled 9,000 km away with zero data loss.** A ZFS mirror disk faulted three times and was diagnosed entirely remotely from kernel logs and SMART data.
- **Paid cloud photo storage replaced.** A multi-thousand-asset library auto-backs-up to Immich, is stored as a plain year/month folder tree, and was verified byte-identical against an off-site copy on another continent.
- **A personal VPN exit node verified properly.** Verified by a change of originating network (ASN) from two different access networks, not just a changed IP, with the real cost of the distance measured (~28 % download, +230 ms).
- **A full media request-to-playback pipeline** (Seerr → Prowlarr → Sonarr/Radarr → download client → Bazarr → Jellyfin), confirmed end to end, with self-service invites for friends.
- **A modded game server for friends in four countries**, sized to a 2-core CPU without AVX. A player's disconnects were traced to one ISP's international route.
- **A second site brought up from a €133 used mini PC.** Validated before use, behind 2FA, reachable from anywhere, and measured at 0 % loss to the NAS over a different German ISP.
- **Every web UI on a memorable HTTPS name, for free.** Eleven services moved from `IP:port` to Tailscale Services names with automatic certificates. Tagging a node that carried 8 friend shares was a documented unknown; it was prepared for, tested with a real friend, and survived.
- **A module retired on evidence.** The camera NVR was decommissioned after measuring its real cost, which funded the photo library.

## Lessons Learned

The ten that apply across the whole homelab. The rest are on each node's page.

- Mesh VPNs like Tailscale remove almost all the traditional networking pain (port forwarding, dynamic DNS, firewall rules) from remote NAS access.
- No single backup mechanism covers every failure mode — off-site cloud copy protects against hardware loss, while local ZFS snapshots protect against accidental deletion/edits with near-zero recovery time. Layering both is more robust than relying on either alone.
- When a GUI reports success and the feature still doesn't work, stop trusting the layer above and query the daemon's own state. `tailscale debug prefs` located a fault that neither the TrueNAS UI nor the Tailscale admin console surfaced at all — and its *other* fields (`NetfilterMode`, `NoSNAT`) ruled out whole categories of cause at the same time.
- A test that cannot fail isn't a test. Checking a public IP while sitting on the same network as the exit node would have "passed" whether or not the feature worked; comparing ASNs across two genuinely different access networks was what made the result mean something.
- A benchmark can be confidently wrong. `dd if=/dev/zero` on an lz4-compressed ZFS pool measures compression, not disk throughput — and the giveaway was that the result was *too good* (3+ GB/s from two mechanical drives). Implausibly good numbers deserve the same scrutiny as bad ones.
- A measured figure has a shelf life. A throughput number measured once in August drove two significant architecture decisions; re-measuring during unrelated work produced results 5–10× higher. Load-bearing measurements deserve periodic re-testing, not permanent trust.
- Removing a module is a result, not a failure. Retiring the NVR was a measured trade — 36.4 GB and 13–15% CPU against a feature the vendor app already did better — and it directly funded the module that replaced it.
- The same category of bug recurs across every new integration in a stack, not just once. `localhost`-vs-container-name was hit on at least five separate app pairings; recognizing the pattern turns a 20-minute debug into a 20-second fix on the sixth occurrence. See the [Troubleshooting Playbook](docs/troubleshooting.md) for the full set of recurring patterns collected across this project.
- "It only breaks for one person" is a routing clue before it's a server clue. The same Germany→Vietnam disconnect reproduced on two consumer carriers and vanished through a data center in the same city — distance was never the variable, transit was.
- A security default is worth one click to verify. The NAS carried `tag:server` and was still two months from key expiry. Nothing would have shown that until it dropped off the tailnet with nobody on site.

## Roadmap

**Vietnam node:** replace both ZFS mirror disks (WD Green → WD Red Plus, active) · CPU upgrade to an i7-4790 · Prometheus alert rules · a whitelist or login mod for the game server. Full list on the [node page](docs/vietnam-node-truenas.md#future-improvements).

**Germany node:** assemble the rack and swap in the TL-SG108E · Uptime Kuma · Home Assistant migration · a 2.5" SSD backup target · a second Tapo C200. Full list on the [node page](docs/germany-node-proxmox.md#status-and-next-steps).

**Cross-site:** scheduled backups in both directions · Tailscale ACLs limiting shared users to the ports they need · self-hosted DNS (AdGuard Home / Pi-hole) as the tailnet resolver · re-test the Vodafone route at different times · a combined two-site architecture diagram.

---

**Stack:** Proxmox VE · TrueNAS CE · ZFS · Tailscale (WireGuard mesh, exit node, subnet router) · SMB/Samba · rclone (Cloud Sync) · Prometheus + Grafana + node_exporter · Jellyfin · Immich (PostgreSQL + Redis) · Prowlarr · Sonarr · Radarr · Lidarr · Bazarr · Seerr · Wizarr · Cleanuparr · Homepage · Minecraft (Forge, itzg/minecraft-server)
