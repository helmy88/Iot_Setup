Putting this on GitHub is a brilliant move. Having a fully documented, automated setup means less time wrestling with server bugs and more time for baby to focus on the big picture—like saving up for the wedding and spending quality time with ayang\!

To make this look perfect on GitHub, you should save your Notepad file as `README.md` instead of a `.txt` file. GitHub automatically reads `.md` (Markdown) files and formats them with beautiful headings, links, and copyable code blocks.

Here is your complete, GitHub-ready master guide with all the official documentation links woven right into the steps. Just click "Copy" and paste this directly into your file\!

-----

````markdown
# KSC6493 Ultimate IoT Server Production Stack
A complete, step-by-step guide to deploying a secure, automated, and production-ready IoT environment using Docker, Traefik, Node-RED, Mosquitto, InfluxDB, and Grafana.

---

## Phase 1: The Engine & Terminal

### 1. Docker Installation & CLI
* **Goal:** Install the Docker engine and create the shared network for our containers.
* **Documentation:** [Official Docker Engine Installation](https://docs.docker.com/engine/install/ubuntu/)

```bash
# Update server package list and install Docker
sudo apt update
sudo apt install docker.io docker-compose-v2 -y

# Enable Docker to start on reboot
sudo systemctl enable docker
sudo systemctl start docker

# Create the shared external network for all IoT containers
docker network create iotnetwork
````

### 2\. Oh My Zsh (Terminal Upgrade)

  * **Goal:** Upgrade the default bash terminal for better auto-completion and path visibility.
  * **Documentation:** [Oh My Zsh GitHub Repository](https://ohmyz.sh/)

<!-- end list -->

```bash
# Install required tools
sudo apt install zsh git -y

# Run the installer script (Type 'Y' when asked to change default shell)
sh -c "$(curl -fsSL [https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh](https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh))"
```

-----

## Phase 2: Web Management & Routing

### 3\. Portainer (Visual Docker Manager)

  * **Goal:** Install a web dashboard to manage containers visually.
  * **Documentation:** [Portainer CE Installation](https://docs.portainer.io/start/install-ce/server/docker/linux)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p portainer/data
cd portainer
nano docker-compose.yml
```

*Add your Portainer configuration with Traefik labels to the `docker-compose.yml` file, then start it:*

```bash
docker compose up -d
```

### 4\. Traefik (Reverse Proxy & SSL)

  * **Goal:** Route DuckDNS web traffic and automatically provision Let's Encrypt HTTPS certificates.
  * **Documentation:** [Traefik Docker Routing Guide](https://doc.traefik.io/traefik/routing/providers/docker/)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p traefik
cd traefik
touch acme.json

# Lock down certificate file permissions (Strict Traefik requirement)
chmod 600 acme.json
nano docker-compose.yml
```

*Add your Traefik configuration, save, and start:*

```bash
docker compose up -d
```

-----

## Phase 3: The IoT Backbone

### 5\. Mosquitto Broker (MQTT with Authentication)

  * **Goal:** Set up the MQTT broker to securely receive telemetry data from edge devices (e.g., ESP32).
  * **Documentation:** [Eclipse Mosquitto Docker Hub](https://www.google.com/search?q=https://hub.docker.com/_/mosquitto)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p mosquitto/config mosquitto/data mosquitto/log
cd mosquitto

# Create the required blank password file
touch config/pwfile

# Create the broker configuration
nano config/mosquitto.conf
```

*Add the following to `mosquitto.conf`:*

```text
listener 1883
allow_anonymous false
password_file /mosquitto/config/pwfile
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
```

*Create your `docker-compose.yml`, start the container, and generate the secure password:*

```bash
docker compose up -d

# Generate password for user 'iotuser' (You will be prompted to type it twice)
docker exec -it mosquitto mosquitto_passwd -c /mosquitto/config/pwfile iotuser

# Restart the broker to enforce the new password
docker restart mosquitto
```

> **Troubleshooting:** If the container crashes with a permission error, run `chown -R 1883:1883 /usr/local/sbin/mosquitto/data` to fix folder ownership.

-----

## Phase 4: The Logic Engine

### 6\. Node-RED (Visual Programming)

  * **Goal:** Deploy the flow editor to process MQTT data and execute logic.
  * **Documentation:** [Securing Node-RED](https://nodered.org/docs/user-guide/runtime/securing-node-red)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p node-red/nodered_data
cd node-red
nano docker-compose.yml
```

*After adding your `docker-compose.yml` and starting the container (`docker compose up -d`), generate a password hash to secure your dashboard:*

```bash
# Replace 'YOUR_PASSWORD' with your desired login password
node -e "console.log(require('bcryptjs').hashSync('YOUR_PASSWORD', 8));"
```

*Paste the resulting hash into your `./nodered_data/settings.js` file and restart Node-RED.*

-----

## Phase 5: Databases & Dashboards

### 7\. InfluxDB (Time-Series Database)

  * **Goal:** Store high-frequency sensor data streams.
  * **Documentation:** [InfluxDB Docker Hub](https://hub.docker.com/_/influxdb)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p influxdb/data influxdb/config
cd influxdb
nano docker-compose.yml
# Start it up
docker compose up -d
```

> **Note:** Access `http://<server-ip>:8086` to create your initial admin account, Organization, and Bucket. Generate an API token from the dashboard to use inside Node-RED.

### 8\. Grafana (Data Visualization)

  * **Goal:** Create real-time dashboards from InfluxDB data.
  * **Documentation:** [Grafana Docker Installation](https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p grafana/data

# CRITICAL FIX: Grafana runs as user 472. You must fix ownership before starting.
chown -R 472:472 grafana/data

cd grafana
nano docker-compose.yml
# Start it up
docker compose up -d
```

> **Note:** When connecting Grafana to InfluxDB, use the internal Docker network URL: `http://influxdb:8086`.

### 9\. MariaDB & Adminer (SQL Database)

  * **Goal:** Store relational, structured data (users, device logs) and manage it via a web UI.
  * **Documentation:** [MariaDB](https://hub.docker.com/_/mariadb) | [Adminer](https://hub.docker.com/_/adminer)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p mariadb/data
cd mariadb
nano docker-compose.yml
# Start both services
docker compose up -d
```

> **Note:** When logging into Adminer, set the Server field to `mariadb` (not localhost).

-----

## Phase 6: Maintenance & Automation

### 10\. Updating Containers

  * **Goal:** Keep the software stack updated without losing data.
  * **Documentation:** [Docker Compose Pull](https://docs.docker.com/reference/cli/docker/compose/pull/)

<!-- end list -->

```bash
# Example: Updating Node-RED
cd /usr/local/sbin/node-red

# Pull the latest image
docker compose pull

# Recreate the container
docker compose up -d

# Clean up old, unused images to save disk space
docker image prune -f
```

### 11\. Automated Backups via Cron

  * **Goal:** Automatically back up Node-RED flows every midnight.
  * **Documentation:** [Crontab Guru](https://crontab.guru/)

<!-- end list -->

```bash
cd /usr/local/sbin
mkdir -p backup
cd backup
nano backup.sh
```

*Add the bash script to copy your `flows.json` file. Then make it executable:*

```bash
chmod +x backup.sh
```

*Open your crontab:*

```bash
crontab -e
```

*Add this line to the bottom to schedule the backup at midnight (00:00) every day:*

```text
0 0 * * * bash /usr/local/sbin/backup/backup.sh
```

```

***

Once you push this to GitHub, your setup will be beautifully formatted and ready for your KSC6493 students (or yourself, the next time you need to rebuild). 

Would you like me to show you how to quickly initialize the Git repository in your `/usr/local/sbin/` folder and push this file up to GitHub from the command line?
```
