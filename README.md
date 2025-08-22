
# <img src="https://github.com/tyzen9/tyzen9/blob/main/images/logos/t9_logo.png" height="25"> Tyzen9 - docker-homeassistant
These are my notes on how to install and configure Home Assistant using docker.

# What's Inside?
The docker-homeassistant stack contains everything you need for a stable Home Assistant experience running in Docker. To get started, all you is a Linux host running Docker.  This documentation generally assumes the use of [Portainer](https://www.portainer.io/) to manage the Docker instance.

Here is what this project has to offer:

1. [Home Assistant](https://www.home-assistant.io/)
1. [VSCode Editor](https://docs.linuxserver.io/images/docker-code-server/)
1. [ESPHome](https://esphome.io/)
1. [NUT](https://hub.docker.com/r/instantlinux/nut-upsd)
1. [MQTT](https://hub.docker.com/_/eclipse-mosquitto/)
1. [VictoriaMertics](https://victoriametrics.com/)
1. [Grafana](https://grafana.com/)


## Getting Started
Deploy the stack into your Docker environment.

> [!Note]
> I run my docker containers as `root`, working to do so otherwise was just too many headaches.

## Home Assistant
Visit the Home Assistant (HA) instance at http://hostname:8123, and the Home Assistant Welcome screen should appear.
Click "Create my smart home", followed by creating a user. Click through the rest of the dialogs as appropriate until you reach the out-of-the-box Home Assistant dashboard.

If you are planning to use the VSCODE Server, then generate a long-lived token for VSCode by clicking on the username, and choosing the security tab.  Save the token generated in a text file temporarily, we will need this if VSCode Server will be used to edit HA's YAML files.

## VS Code
Visit the VS Code Server at http://hostname:8443.  This will prompt for the password configured in the stack environment to be entered, do this and click SUBMIT to gain access to the VSCode editor.

In the extensions settings, install:
- [Home Assistant Config Helper](keesschollaart.vscode-home-assistant) (by keesschollaart) 


Then, make sure the following settings are present in the: `[DEFAULT_WORKSPACE]/.vscode/settings.json` file within the config volume:

```yaml
{
    "files.associations": {
        "*.yaml": "home-assistant"
    },
    "vscode-home-assistant.hostUrl": "http://localhost:8123",
    "vscode-home-assistant.longLivedAccessToken": "***long_lived_token_from_HA***",
    "vscode-home-assistant.ignoreCertificates": true
} 
```

### Test File Edits ###
Next, make a small edit to the configuration.yaml file (like adding a comment), and attempt to save the change.  
  

## Additional Recommended Extensions ##

1. YAML (by Red Hat)
1. Log File Highlighter (by emilast)
1. ESPHome (by ESPHome)
1. Material Design Icons Intellisense Preview (by lukas-tr)
1. indent-rainbow (by oderwat)
1. vscode-icons (by vscode-icons-team)

##  Network UPS Tools (nut) ##

The UPS devices to be monitored are configured in the `/etc/nut` directory of the `nut` container. 

The `ups.conf` file configure the UPS devices to be connected to, below is a sample of two UPS devices:

```conf
[ups-1]
    driver = usbhid-ups
    port = /dev/usb/hiddev0
    desc = "APC Back-UPS (1)"
    serial = 4B1236P17533

[ups-2]
    driver = usbhid-ups
    port = /dev/usb/hiddev1
    desc = "APC Back-UPS (2)"
    serial = 4B1227P35221
```

> [!Note]
> Some work will be need to be done to research the port path, and the serial number of each device - (Google is your friend)

Next, the `upsmon.conf` files needs to be updated to tell `nut` what UPS devices to monitor, the password and the user to use.  `password` in this context is just telling upsmon to use a password for API authentication.  This was set up in compose with the `API_PASSWORD` property.

```conf
MONITOR ups-1@localhost 1 upsmon password primary
MONITOR ups-2@localhost 1 upsmon password primary
RUN_AS_USER nut
```

Once these changes are made, restart the `nut` container, and try to initialze the `nut` integration.

## Mosquitto MQTT

## Configuration Steps
Create Mosquitto Configuration:On your host, create a directory for Mosquitto configuration (e.g., `/path/to/mosquitto/config`). Create a basic `mosquitto.conf` file in the `mosquitto-config` volume (e.g., `/path/to/mosquitto/config/mosquitto.conf`)

```conf
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
listener 1883
allow_anonymous false
password_file /mosquitto/config/passwd
```

For authentication, add a password file. Run the following command on the docker host, replacing `username` with the username to use for mosquitto user authentication.

```bash
docker exec -it mosquitto mosquitto_passwd -c /mosquitto/config/passwd [username]
```

## VictoriaMmetrics

Integrate VictoriaMetrics into your existing Home Assistant Docker Compose setup for long-term data storage and visualization.

> [!Note]
> This method uses Home Assistant's InfluxDB integration with VictoriaMetrics' InfluxDB compatibility.









