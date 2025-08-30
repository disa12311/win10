### Instructions for setting up Docker in **Codespace**

1. **Check available storage**
In the terminal, run the following command to view the partitions and available storage:

```bash
df -h
```

Choose the partition with the largest storage capacity.

2. **Create the Docker folder**
Run the following command to create the folder where Docker will store the data:

```bash
sudo mkdir -p /tmp/docker-data
```

3. **Configure Docker**
Now, edit the Docker configuration file:

```bash
sudo nano /etc/docker/daemon.json
```

Paste the following content:

```json
{
"data-root": "/tmp/docker-data"
}
```

4. **Restart your Codespace**
Power cycle your Codespace to apply the changes.

5. **Verify the configuration**
To ensure Docker is configured correctly, run:

```bash
docker info
```

---

### Create `windows10.yml` file

Create a file named `windows10.yml` with the following contents:

```yaml
services:
windows:
image: dockurr/windows:latest
container_name: windows
# keep your restart and grace period
restart: always
stop_grace_period: 2m

# preserve network admin capability (tun) for VPN/bridge if needed
cap_add:
- NET_ADMIN

# devices needed for KVM + tun; Added optional GPU passthrough below 
devices: 
- "/dev/kvm:/dev/kvm" 
- "/dev/net/tun:/dev/net/tun" 
# optional GPU (if you want passthrough) — uncomment and adjust on hosts that support it 
# - "/dev/dri/card0:/dev/dri/card0" 
# - "/dev/dri/renderD128:/dev/dri/renderD128" 

ports: 
- "8006:8006" # noVNC / web viewer (installation / console) 
- "3389:3389/tcp" # RDP (TCP only) 
# optional: expose spice / other ports if you add them 
# - "5900:5900/tcp" 

environment: 
# Windows versions & configurations 
VERSION: "10" #10, 11, win81, xp, server2019, ...
EDITION: "pro" # pro/enterprise/education
LANGUAGE: "en-US" # eg en-US, vi-VN, de-DE
LOCALE: "en-US"
EULA: "accept" # must accept eula if image requires
# default user created in windows
USERNAME: ${WINDOWS_USERNAME}
PASSWORD: ${WINDOWS_PASSWORD}

# VM resources (depending on host)
RAM_SIZE: "4.5G" # ex: "8G", "16G"
CPU_CORES: "4" # ex: "2", "8"
DISK_SIZE: "64G" # ex: "256G"
VIRTIO_DRIVERS: "auto" # auto | custom path

# network / time zone / extras
TIMEZONE: "Asia/Bangkok"
TZ: "Asia/Bangkok"

# share host -> Windows folder (you still have to mount the volume below)
SHARED_FOLDERS: "/mnt/windows-data:/host" # optional mapping as host:guest

# update / debug
AUTO_UPDATE: "true" # if image supports update script
DEBUG: "false"

# security / access
RDP_PASSWORD: ${WINDOWS_PASSWORD} # if image uses this variable to set RDP pass
# depending on image, additional variables may be needed: USER_UID / USER_GID, SYSTEM_USER, etc.

volumes:
# where to store QCOW2/virtual disk: CRITICALLY persist VM state here

- /tmp/docker-data:/mnt/disco1 # path of your choice (host bind)

- windows-data:/mnt/windows-data # named volume to backup/move

- ./win-iso-cache:/iso-cache # (optional) cache ISO to avoid reloading

- ./virtio-drivers:/virtio # (optional) if you want to mount virtio drivers
# mount docker socket if you want to let the container initialize nested containers (be careful)

# - /var/run/docker.sock:/var/run/docker.sock:ro

# resources and limits to avoid taking over the entire host
deploy:
resources:
limits:
memory: 12G
cpus: '2.0'

# ulimits for QEMU to run stably
ulimits:
nofile: 65536

# logging: rotate logs to avoid disk overflow
logging:
driver: "json-file"
options:
max-size: "50m"
max-file: "5"

# healthcheck: try accessing web console (requires container with curl)
healthcheck:
test: ["CMD-SHELL", "curl -fsS http://localhost:8006/ || exit 1"]
interval: 30s
timeout: 10s
retries: 5
start_period: 60s

# security notes: run as privileged only if you understand the risks; KVM usually needs devices + cap_add (no privileged needed)

# privileged: false

# optional: use host network if you want RDP to listen directly (caution)

# network_mode: "bridge"

# optional entrypoint/command overrides (usually not needed)

# command: ["--some-flag"]

# labels: metadata, monitoring
labels:
- "org.opencontainers.image.title=dockurr-windows"
- "com.example.project=windows-lab"

volumes:
windows-data:
driver: local
```

---

### Create `.env` file

Create a `.env` file in the same folder as `windows10.yml` to define sensitive environment variables, such as the username and password. Password:

```ini
WINDOWS_USERNAME=YourUsername
WINDOWS_PASSWORD=YourPassword
```

This file should not be uploaded to public repositories for security reasons. Make sure to include it in your `.gitignore` if you're working with version control.

---

### Starting the Container

1. **Start the container** by running the following command:

```bash
docker-compose -f windows10.yml up
```

2. **Start the container** with:

```bash
docker start windows
```

---

That's it! With these steps, you've successfully configured Docker and created the **Windows 10** container in your Codespace. If you have any further questions or need additional assistance, don't hesitate to ask.

---

This file now reflects the changes we made to the `docker-compose.yml` file, such as managing credentials through the `.env` file and removing unnecessary ports and configurations. Also, remember to keep your sensitive environment variables safe and out of version control.