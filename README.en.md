# TR-069 ACS — CPE Management Panel for ISPs

![PHP](https://img.shields.io/badge/PHP-777BB4?style=flat&logo=php&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat&logo=mongodb&logoColor=white)
![Alpine.js](https://img.shields.io/badge/Alpine.js-8BC0D0?style=flat&logo=alpinedotjs&logoColor=white)
![TR-069](https://img.shields.io/badge/TR--069%2FCWMP-GenieACS-FF6B00?style=flat)

*[Versão em português](README.md)*

> System **in active production**, used daily by a NOC/support team at a regional ISP, serving real customers. This repository is a portfolio presentation of the project — it shows **what the system does and how it's designed**, without exposing source code, credentials, or customer data.
>
> Designed and built individually — from the architecture (TR-069/CWMP, relational mirror) to running it in production.

## Table of Contents

- [What it is](#what-it-is)
- [Operation at a glance](#operation-at-a-glance)
- [Screen tour](#screen-tour)
- [Technical challenges behind the screens](#technical-challenges-behind-the-screens)
- [Author](#author)

---

## What it is

A single panel where the support team:

- Remotely manages routers/ONTs installed at customers' homes (toggle Wi-Fi, change password, reboot, update firmware, diagnose signal) without needing a truck roll for most cases.
- Tracks in real time which customers are online/offline and their signal quality.
- Actively monitors signal and connectivity, automatically flagging customers with recurring drops before they have to call in.

Under the hood, the system speaks the telecom remote-management protocol (**TR-069/CWMP**) with thousands of devices from different manufacturers, each with its own parameter "dialect" — and hides that complexity from the operator.

---

## Operation at a glance

Real figures pulled directly from the screenshots below (nothing inflated for the portfolio):

| Metric | Value |
|---|---|
| CPEs monitored in real time | ~8,100 |
| Devices online at screenshot time | ~5,800 (71% of the fleet) |
| Devices auto-flagged with critical signal | ~2,600 |
| Dual-stack IPv4+IPv6 coverage across the fleet | 97% |

---

## Screen tour

### 1. Overview dashboard
Consolidated view of the fleet: total devices online/offline, breakdown by manufacturer/model, recent alerts. The landing screen for anyone starting a shift.

![Overview dashboard](screenshots/dashboard.png)

### 2. Device detail page
Opening a specific customer's device shows: connection status, optical signal, IP/PPPoE, configured Wi-Fi networks (with inline name/password change), currently connected devices, and quick actions (reboot, sync, firmware update). Practically every technical support interaction revolves around this screen.

![Device detail page](screenshots/dispositivo.png)

<details>
<summary>See the device page's other tabs (WAN/Internet, Local Network, Wireless)</summary>

**WAN/Internet** — PPPoE credentials, connection status, VLAN, IPv6, bandwidth usage:

![WAN/Internet tab](screenshots/dispositivo_wan.png)

**Local Network** — IP addressing, DHCP server, DNS:

![Local Network tab](screenshots/dispositivo_lan.png)

**Wireless** — 2.4GHz/5GHz radios and up to 8 configurable Wi-Fi networks (SSIDs) per device:

![Wireless tab](screenshots/dispositivo_wifi.png)

</details>

### 3. Remote diagnostics
Ping/traceroute/speed-test tools triggered from the customer's own router (not from the server), to isolate whether an issue is on the customer's network, the link to the central office, or something external.

![Remote diagnostics](screenshots/diagnostico.png)

### 4. Quarantine / active monitoring
Engine that continuously watches signal and connectivity and automatically flags customers with recurring drops — before the customer has to call in. (Queue was empty at screenshot time — a sign the fleet was healthy.)

![Quarantine](screenshots/quarentena.png)

### 5. Customer health
A per-customer scorecard combining signal quality, connection stability, and quarantine history into a single indicator — to prioritize who needs proactive attention.

![Customer health](screenshots/saude_cliente.png)

### 6. Customer-facing connection test
A link sent to the end customer (no login required) that runs a full speed test in their own browser — not on the router — to measure the real internet experience from the user's side.

![Customer-facing connection test](screenshots/teste_cliente.png)

---

## Technical challenges behind the screens

Some real scale and reliability problems that had to be diagnosed and fixed during operation:

- **Clogged command queue.** A daily bulk-sync job was unknowingly firing the most expensive possible command (scanning the *entire* parameter tree) across the whole fleet. On devices with a history of instability this failed, and since the management server doesn't expire stuck tasks on its own, the backlog grew forever — reaching the hundreds of thousands. Fixed by replacing the full scan with targeted updates of only what was needed, plus an automatic cleanup of stalled tasks.

- **Latency that only showed up on certain models.** The same "sync" click in the panel took minutes on some devices and was instant on others. The root cause: for those models the system fell back to a generic, expensive path due to a missing manufacturer-specific mapping. Fixed by correctly mapping each model.

- **Misleading Wi-Fi signal indicator.** On one line of devices, the protocol field literally named "RSSI" is *not* the real signal in dBm — it's just a bar indicator (0 to 5), and the real value is hidden in a differently-named field. Each manufacturer also formats this data differently. Fixed with per-model normalization, documenting which field is trustworthy for each manufacturer.

- **Connected-device count that "froze".** The list of Wi-Fi devices connected to a router sometimes stopped updating its item count, even with individual data staying fresh — a subtlety in how the management protocol distinguishes "refresh an already-known value" from "re-check how many items exist." Fixed by adjusting that setting specifically for variable-size tables.

- **Safety trip-wire against cascading failure.** An integration with the billing system once interpreted a temporary external API instability as "every contract was cancelled," and started deleting real production customer records before being caught and stopped manually. The fix was structural: the sync now aborts entirely if the external API fails on any page of results, and also aborts if it detects a cancellation volume inconsistent with normal patterns — treating that as an error signal, not a real event.

---

## Author

**Luan Castelhano** — [LinkedIn](https://www.linkedin.com/in/luan-castelhano-797372304/) · [GitHub](https://github.com/luanca3213)

---

*System in active production, serving real customers. This repository contains only presentation material for portfolio purposes — no source code, credentials, or customer data. Sensitive data in the screenshots (customer names, CPF/CNPJ, logins) was redacted before publishing.*
