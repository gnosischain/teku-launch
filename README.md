# Running a Teku client for GBC

This document describes how to run Teku beacon node + Teku validator for the Gnosis Beacon Chain.

See a similar repo with Prysm node setup - https://github.com/gnosischain/prysm-launch
See a similar repo with Lighthouse node setup - https://github.com/gnosischain/lighthouse-launch
See a similar repo with Nimbus node setup - https://github.com/gnosischain/nimbus-launch

## Assumptions
* This document assumes that you already have an xDai node available for your use (or public JSON RPC endpoint)
* You have already generated your validator accounts using the fork of the official deposit-cli - https://github.com/gnosischain/deposit-cli. You will need validator keystores and passwords for them to run the validator client.
* You might start your node and validator first, and only them make a deposit, once everything works.
* You have a persistent linux VM with docker installed on it, which is accessible from the public internet via a fixed IP address. We recommend using a VM with at least 4 vCPU, 8 GB RAM, 100 GB SSD

## Configuration
1) Create an empty directory somewhere on the VM (e.g. `/root/gbc`)
2) Clone the repository with all necessary configs:
```bash
cd /root/gbc
git clone https://github.com/gnosischain/teku-launch .
```
3) Copy all validators keystore files to the `./keys/validator_keys` directory. Ensure that copied keystores are only used on a single VM instance.
4) Write keystore passwords to the `./keys/passwords` directory. Password for `./keys/validator_keys/validator_abc.json` keystore should be named `./keys/validator_keys/validator_abc.txt`.
5) Create `.env` file from the example at `.env.example`. Put the valid external IP address of your VM and an xDAI RPC url in the config. Other values can be left without changes.

If it is required to send the node and validator logs to a remote syslog server the following actions can be done (it is assumed that the node and validator will be run by using the `docker-compose-syslog.yml` file with the `docker-compose` command in the instructions below).

6) Copy `./syslog/etc/logrotate.d/docker-logs` to `/etc/logrotate.d/`.
7) Copy `./syslog/etc/rsyslog.d/30-gbc-local.conf` and `./syslog/etc/rsyslog.d/35-gbc-remote-logging.conf` to `/etc/rsyslog.d/`.
8) Modify `target` and `port` in `/etc/rsyslog.d/35-gbc-remote-logging.conf` to point to a remote syslog server.
9) Restart the rsyslog service by `systemctl restart rsyslog`.

## Running staking node
1) Run the following command to start the beacon node with a validator:
```bash
docker-compose up -d node
docker-compose logs -f node
```

2) Your node should find other peers and start syncing with the rest of the network
3) Validators will be run in the same container with the beacon node

## Advanced configuration for remote validator clients
1) It is possible to run validators in a separate container from the beacon node.
This can be useful for setups which run beacon nodes and validator clients on different nodes.
Feel free to adopt running configs for your particular setup, if that is your case.

Reading:
- https://docs.teku.consensys.net/en/latest/Reference/CLI/Subcommands/Validator-Client/
- https://docs.teku.consensys.net/en/latest/HowTo/External-Signer/Use-External-Signer/

## Collecting metrics
1) Run the following command to start the prometheus server, it will allow to use grafana's dashboards (configured separately):
```bash
docker-compose up -d prometheus
```
