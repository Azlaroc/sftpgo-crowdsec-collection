# SFTPGo CrowdSec Collection

This collection provides a parser and scenario for detecting bruteforce attacks on SFTPGo, supporting both FTP (port 2121) and SFTP (port 2022) protocols.

## Components
1. **Parser**: `Azlaroc/sftpgo-logs` - Parses SFTPGo JSON logs for failed and successful login attempts.
2. **Scenario**: `Azlaroc/sftpgo-bf` - Triggers a 4-hour ban after 5 failed logins within 2 minutes.

## Installation
Until merged into the CrowdSec Hub (see PR: https://github.com/crowdsecurity/hub/pull/1461), install manually:
```bash
sudo cscli collections install https://raw.githubusercontent.com/Azlaroc/sftpgo-crowdsec-collection/main/collections/Azlaroc/sftpgo.yaml
```
Once merged, use:
```bash
sudo cscli collections install Azlaroc/sftpgo
```


## Requirements
1. SFTPGo with verbose logging enabled (JSON format).
2. Log file at `/opt/sftpgo/logs/sftpgo.log`.
3. CrowdSec v1.6.11 or later.

## Testing
1. Simulate 6 failed logins using an FTP/SFTP client.
2. Check bans with `sudo cscli decisions list`.
3. Verified with 4 bans triggered in production.

## Author
1. Azlaroc (https://github.com/Azlaroc)
