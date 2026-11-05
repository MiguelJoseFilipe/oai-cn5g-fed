# NOVA 5G core

Configuration of the OAI 5G core used for the 5G measurements of the NOVA platform. It is the
`docker-compose-basic-nrf.yaml` deployment of this repository, with three files changed. The
base commit is `2712e714a2` of `oai-cn5g-fed`.

| File | Change |
|---|---|
| `docker-compose/conf/basic_nrf_config.yaml` | PLMN 001/01, TAC 1, one slice (SST 1, SD 000001), one DNN `default` (12.1.1.0/24, 1 Gbit/s AMBR), DNS 8.8.8.8 / 8.8.4.4, subscriptions read from the UDR, UPF SNAT off |
| `docker-compose/docker-compose-basic-nrf.yaml` | fixed container addresses, restart policy `unless-stopped`, AMF started after MySQL and AUSF, database `oai_db-basic.sql`, images pinned to the digests of the measurements |
| `docker-compose/database/oai_db-basic.sql` | five subscribers (placeholders) with static IPv4 12.1.1.2 to 12.1.1.6; upstream subscribers removed |

## Subscribers

The SIM identities and keys are not published. Each subscriber `n` (1 to 5) has three placeholders
in `docker-compose/database/oai_db-basic.sql`: `<IMSI_n>` (15 digits), `<K_n>` and `<OPC_n>`
(32 hexadecimal characters each). Replace them with the values of your SIMs before the first start:

```bash
sed -i 's/<IMSI_1>/001010000000001/g; s/<K_1>/<32 hex>/g; s/<OPC_1>/<32 hex>/g' \
    docker-compose/database/oai_db-basic.sql
```

Delete the rows of unused subscribers in the three `INSERT` statements. The sequence number
starts at 0. MySQL loads the file when its container is created, so recreate the container after an
edit (`docker compose down` then `up`).

## Run

```bash
cd docker-compose
docker compose -f docker-compose-basic-nrf.yaml up -d
docker compose -f docker-compose-basic-nrf.yaml ps      # all services healthy
docker compose -f docker-compose-basic-nrf.yaml down
```

The core network is `192.168.70.128/26`. The AMF is at 192.168.70.132, the UPF at 192.168.70.134.
The gNB joins this network at 192.168.70.129; its configuration is in the `openairinterface5g`
fork (branches `nova-fixes` and `nova-ntn`). UE traffic leaves the UPF on N6 with the UE
address. The `oai-ext-dn` container routes 12.1.1.0/24 through the UPF; any other server on the
data network needs the same route.

## Notes

- SNAT is off so that each UE keeps its own address on N6. A SIP server then distinguishes the UEs
  and relays UE-to-UE media. Set `enable_snat: yes` to give the UEs Internet access instead.
- The image digests fix the core version of the measurements. The `develop` tags they belong to
  move. Remove the `@sha256:` suffix to run the current images.
