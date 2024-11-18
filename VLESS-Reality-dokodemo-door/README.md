# README

Here is an overview of how to obfuscate WireGuard by passing it through an Xray Reality tunnel, so that it appears to an eavesdropper to be HTTPS traffic.

## Server

1. Install Xray on the server.
2. Generate a UUID for Xray.
3. Configure Xray on the server.
4. Run Xray as a systemd service on the server.
5. Install and configure WireGuard on the server.

## Client

1. Install the Xray command-line client.
2. Configure the Xray client to use `config_client.json` in this repository with `dokodemo-door` input on `127.0.0.1`.
3. Install WireGuard on the client.
4. Calculate AllowedIPs to allow all traffic except traffic for the server itself to pass through WireGuard (use the WireGuard AllowedIPs Calculator)
5. Connect and bring up both Xray and WireGuard tunnels
