# Uptime Probe downloads

Public Linux binaries and installer for API Uptime Monitoring. The application and its credentials are not part of this repository. Pro includes up to three private probe locations per organization; a binary download does not grant monitoring access.

## Install

The download command becomes available after the first published release.

Requires Linux amd64 or arm64, a running systemd manager, curl, ca-certificates and sha256sum. No Go compiler, Docker, inbound port or GitHub account is required.

1. In the application, open **Settings → Private probes**, register a location and save its one-time token.
2. Enter the private subnets this worker may reach, then download `uptime-probe.env` onto the Linux host. Keep it private.
3. In the directory containing that file, run:

   ```sh
   chmod 600 uptime-probe.env
   curl --proto '=https' --tlsv1.2 -fsSL https://github.com/pawelst/uptime-probe/releases/latest/download/install.sh | sudo sh -s -- --config ./uptime-probe.env
   ```

The installer detects the CPU architecture, downloads a binary and service unit from one fixed release, verifies their SHA-256 checksums, creates an unprivileged `uptime-probe` account, installs the configuration as root with mode 0600 and starts the service. Wait for **Online** in the app, then select the private location in your monitor. Allow outbound HTTPS to the application, DNS and HTTP/S to the monitored endpoints.

To inspect the installer first, download `install.sh` from [Releases](https://github.com/pawelst/uptime-probe/releases), review it, and run `sudo sh install.sh --config ./uptime-probe.env`. Checksums detect corrupted downloads; they are distributed by the same GitHub release publisher.

## Upgrade, rotate tokens and troubleshoot

Re-run the installation command to upgrade. After token rotation, download the new configuration and re-run it with `--config`. To preserve an existing configuration during an upgrade, omit `--config`; the installer uses `/etc/uptime-probe/agent.env`. Use `--no-start` to install without starting or restarting the service. Use `--version probe-vX.Y.Z` to select a published version explicitly. There are no automatic background upgrades.

The previous binary is retained at `/usr/local/bin/uptime-probe.previous`; an explicitly replaced configuration is backed up at `/etc/uptime-probe/agent.env.previous`. Backups stay on your host and may contain an old token.

```sh
systemctl status uptime-probe
journalctl -u uptime-probe -n 30
/usr/local/bin/uptime-probe --version
```

Never publish tokens, environment files or unredacted credentials when requesting support. Private addresses require an explicit `UPTIME_PRIVATE_NETWORKS` allowlist; loopback, link-local and metadata addresses remain blocked. Internal DNS must resolve on the worker. For a private certificate authority, configure a trusted CA bundle using the service's environment file (`SSL_CERT_FILE`); TLS verification stays enabled.
