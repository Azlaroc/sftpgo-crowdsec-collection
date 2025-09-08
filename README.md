# SFTPGo CrowdSec Collection

This collection provides a parser and scenario for detecting bruteforce attacks on SFTPGo, supporting both FTP (port 2121) and SFTP (port 2022) protocols.

## Components
1. **Parser**: `Azlaroc/sftpgo-logs` - Parses SFTPGo JSON logs for failed and successful login attempts.
2. **Scenario**: `Azlaroc/sftpgo-bf` - Triggers a 4-hour ban after 5 failed logins within 2 minutes.

## Manual Installation for SFTPGo CrowdSec Collection

This guide provides the steps to manually install the SFTPGo CrowdSec collection from the GitHub repository (https://github.com/Azlaroc/sftpgo-crowdsec-collection) for CrowdSec versions (e.g., v1.6.11) where `cscli collections install` does not support direct URL or local file installation. It also includes instructions for configuring SFTPGo to produce logs compatible with the collection's parser. The process involves downloading the collection, parser, and scenario files, placing them in the correct directories, enabling the components, and setting up the SFTPGo log source.

## Prerequisites

1. Ensure CrowdSec is installed and running:

   ```
   sudo systemctl status crowdsec
   ```

2. Verify SFTPGo is installed and running, with logging configured (see "Configuring SFTPGo Logs" below).

3. Ensure `wget` is installed (`sudo apt install wget`) or use `curl` as an alternative.

4. Back up your CrowdSec configuration before proceeding:

   ```
   sudo cp -r /etc/crowdsec /etc/crowdsec_backup_$(date +%Y%m%d)
   ```

## Configuring SFTPGo Logs

To ensure SFTPGo produces logs that the `Azlaroc/sftpgo-logs` parser can process, configure SFTPGo to write verbose logs to `/opt/sftpgo/logs/sftpgo.log` in a syslog-compatible format. If using Docker, the following configuration in your `docker-compose.yml` achieves this:

1. **Key Settings**:
   Include these environment variables in your SFTPGo service definition:

   ```
   environment:
     - SFTPGO_LOG_FILE_PATH=/var/log/sftpgo/sftpgo.log
     - SFTPGO_LOG_VERBOSE=true
     - SFTPGO_LOG_MAX_SIZE=10
     - SFTPGO_LOG_MAX_BACKUPS=5
   ```

   - `SFTPGO_LOG_FILE_PATH`: Sets the log file path inside the container.
   - `SFTPGO_LOG_VERBOSE`: Enables detailed logging for parsing login attempts and errors.
   - `SFTPGO_LOG_MAX_SIZE`: Limits log file size to 10MB to manage disk usage.
   - `SFTPGO_LOG_MAX_BACKUPS`: Keeps up to 5 log file backups.

2. **Volume Mapping**:
   Map the container's log directory to the host:

   ```
   volumes:
     - /opt/sftpgo/logs:/var/log/sftpgo
   ```

   This ensures logs are written to `/opt/sftpgo/logs/sftpgo.log` on the host, accessible to CrowdSec.

3. **Apply Configuration**:
   If using the provided `docker-compose.yml`, deploy or update the SFTPGo service:

   ```
   docker-compose up -d
   ```

4. **Verify Log File**:
   Check that logs are being written:

   ```
   ls -l /opt/sftpgo/logs/sftpgo.log
   ```

   Ensure the file is readable by the `crowdsec` user:

   ```
   sudo chmod 644 /opt/sftpgo/logs/sftpgo.log
   sudo chown sftpgo:crowdsec /opt/sftpgo/logs/sftpgo.log
   ```

## Installation Steps

1. **Create Directories**  
   Set up the directory structure for the collection, parser, and scenarios:

   ```
   sudo mkdir -p /etc/crowdsec/collections/Azlaroc
   sudo mkdir -p /etc/crowdsec/parsers/s01-parse/Azlaroc
   sudo mkdir -p /etc/crowdsec/scenarios/Azlaroc
   ```

2. **Download YAML Files**  
   Download the collection, parser, and scenario files from the GitHub repository:

   ```
   sudo wget https://raw.githubusercontent.com/Azlaroc/sftpgo-crowdsec-collection/main/collections/Azlaroc/sftpgo.yaml -O /etc/crowdsec/collections/Azlaroc/sftpgo.yaml
   sudo wget https://raw.githubusercontent.com/Azlaroc/sftpgo-crowdsec-collection/main/parsers/s01-parse/Azlaroc/sftpgo-logs.yaml -O /etc/crowdsec/parsers/s01-parse/Azlaroc/sftpgo-logs.yaml
   sudo wget https://raw.githubusercontent.com/Azlaroc/sftpgo-crowdsec-collection/main/scenarios/Azlaroc/sftpgo-bf.yaml -O /etc/crowdsec/scenarios/Azlaroc/sftpgo-bf.yaml
   sudo wget https://raw.githubusercontent.com/Azlaroc/sftpgo-crowdsec-collection/main/scenarios/Azlaroc/sftpgo-slow-bf.yaml -O /etc/crowdsec/scenarios/Azlaroc/sftpgo-slow-bf.yaml
   ```

   Alternatively, clone the repository and copy files:

   ```
   git clone https://github.com/Azlaroc/sftpgo-crowdsec-collection.git
   sudo cp sftpgo-crowdsec-collection/collections/Azlaroc/sftpgo.yaml /etc/crowdsec/collections/Azlaroc/
   sudo cp sftpgo-crowdsec-collection/parsers/s01-parse/Azlaroc/sftpgo-logs.yaml /etc/crowdsec/parsers/s01-parse/Azlaroc/
   sudo cp sftpgo-crowdsec-collection/scenarios/Azlaroc/sftpgo-bf.yaml /etc/crowdsec/scenarios/Azlaroc/
   sudo cp sftpgo-crowdsec-collection/scenarios/Azlaroc/sftpgo-slow-bf.yaml /etc/crowdsec/scenarios/Azlaroc/
   ```

3. **Set Permissions and Ownership**  
   Ensure files have correct permissions (644) and ownership (`crowdsec:crowdsec`):

   ```
   sudo chmod 644 /etc/crowdsec/collections/Azlaroc/sftpgo.yaml
   sudo chmod 644 /etc/crowdsec/parsers/s01-parse/Azlaroc/sftpgo-logs.yaml
   sudo chmod 644 /etc/crowdsec/scenarios/Azlaroc/sftpgo-bf.yaml
   sudo chmod 644 /etc/crowdsec/scenarios/Azlaroc/sftpgo-slow-bf.yaml
   sudo chown crowdsec:crowdsec /etc/crowdsec/collections/Azlaroc/sftpgo.yaml
   sudo chown crowdsec:crowdsec /etc/crowdsec/parsers/s01-parse/Azlaroc/sftpgo-logs.yaml
   sudo chown crowdsec:crowdsec /etc/crowdsec/scenarios/Azlaroc/sftpgo-bf.yaml
   sudo chown crowdsec:crowdsec /etc/crowdsec/scenarios/Azlaroc/sftpgo-slow-bf.yaml
   ```

4. **Enable Parsers and Scenarios**  
   Install the parser and scenarios using `cscli`:

   ```
   sudo cscli parsers install /etc/crowdsec/parsers/s01-parse/Azlaroc/sftpgo-logs.yaml
   sudo cscli scenarios install /etc/crowdsec/scenarios/Azlaroc/sftpgo-bf.yaml
   sudo cscli scenarios install /etc/crowdsec/scenarios/Azlaroc/sftpgo-slow-bf.yaml
   ```

   Note: The `sftpgo.yaml` collection file is a manifest and does not require separate installation. Enabling the parser and scenarios activates the collection.

5. **Configure SFTPGo Log Source**  
   Edit `/etc/crowdsec/acquis.yaml` to monitor SFTPGo logs:

   ```
   sudo nano /etc/crowdsec/acquis.yaml
   ```

   Add the following at the end:

   ```
   filenames:
     - /opt/sftpgo/logs/sftpgo.log
   labels:
     type: syslog
   ```

   Save and exit. This configuration matches the SFTPGo log format expected by the `Azlaroc/sftpgo-logs` parser.

6. **Restart CrowdSec**  
   Apply changes by restarting the CrowdSec service:

   ```
   sudo systemctl restart crowdsec
   ```

7. **Verify Installation**  
   Confirm the components are installed and enabled:

   ```
   sudo cscli parsers list
   sudo cscli scenarios list
   sudo cscli collections list
   ```

   Look for `Azlaroc/sftpgo-logs` (parser), `Azlaroc/sftpgo-bf`, and `Azlaroc/sftpgo-slow-bf` (scenarios). The collection `Azlaroc/sftpgo` may appear as “local” or “tainted” since it’s not from the official hub.

8. **Check Metrics**  
   Verify that SFTPGo logs are being parsed and scenarios are active:

   ```
   sudo cscli metrics
   ```

   To test, simulate SFTPGo activity (e.g., failed logins) and check for decisions:

   ```
   sudo cscli decisions list
   sudo cscli alerts list
   ```

## Troubleshooting

1. **Log File Access**  
   If logs aren’t parsed, ensure `/opt/sftpgo/logs/sftpgo.log` exists and is readable:

   ```
   sudo ls -l /opt/sftpgo/logs/sftpgo.log
   sudo chmod 644 /opt/sftpgo/logs/sftpgo.log
   sudo chown sftpgo:crowdsec /opt/sftpgo/logs/sftpgo.log
   ```

2. **Validation Errors**  
   If `cscli install` fails, check YAML syntax:

   ```
   sudo cat /etc/crowdsec/parsers/s01-parse/Azlaroc/sftpgo-logs.yaml
   ```

   Run a hub test:

   ```
   sudo cscli hubtest run
   ```

3. **CrowdSec Update**  
   If issues persist, consider updating CrowdSec:

   ```
   sudo apt update && sudo apt upgrade crowdsec
   ```

## Notes

1. This method is required because `cscli collections install` does not support direct URLs or local files for custom collections in CrowdSec v1.6.11.
2. If the collection is merged into the official CrowdSec Hub, install with:

   ```
   sudo cscli collections install Azlaroc/sftpgo
   ```

3. For offline setups, transfer files manually (e.g., via USB) and follow steps 3–8.


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
