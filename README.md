<p align="center">
<img src="images/home-assistant-logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <img src="images/esphome_logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp;<img src="images/vscode_logo.png" height="35" style="padding-left: 25px">&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <img src="images/mqtt-logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <img src="images/vm_logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp;<img src="images/grafana-logo.png" height="35" style="padding-left: 25px">  
</p> 

# <img src="images/t9_logo.png" height="25"> Tyzen9 - docker-homeassistant
This is an all in one docker stack for running home assistant and some common "Add-Ons" that are generally considered the basic needs.  This will allow for the running of Home Assistant in a docker instance.

## Prerequisites
Install [Docker Engine](https://docs.docker.com/get-docker/) or [Docker Desktop](https://docs.docker.com/desktop/) if you require the Docker user interface.  In production it's generally best to use [Docker Engine](https://docs.docker.com/get-docker/) on a Linux host operating system, and a lightweight service delivery platform designed for managing containerized applications such as [Portainer](https://www.portainer.io/).

This documentation assumes you have a working knowledge of [Docker](https://www.docker.com/), and [Home Assistant](https://www.home-assistant.io/) administration.

## Container Configuration
This `docker-compose` implementation is configured using the `environment` section(s) of the `docker-compose.yaml` file.  The values between the `${}` characters are substituted from the `.env` file.  If you do not configure these values, default values preceeded by the hyphen will be used.

> [!Note]
> These containers are configured to run as `root`. While is it not impossible to configure them to run otherwise, working to do so otherwise was just too many headaches. Afterall HASOS runs as `root` out of the box 🙌.


# What's Inside?
The docker-homeassistant stack contains everything you need for a stable Home Assistant experience running in Docker. To get started, all you is a Linux host running Docker.  This documentation generally assumes the use of [Portainer](https://www.portainer.io/) to manage the Docker instance.

Here is what this project has to offer:

1. [Home Assistant](https://www.home-assistant.io/)
1. [VSCode Editor](https://docs.linuxserver.io/images/docker-code-server/)
1. [ESPHome](https://esphome.io/)
1. [Network UPS Tools (nut)](https://hub.docker.com/r/instantlinux/nut-upsd)
1. [MQTT](https://hub.docker.com/_/eclipse-mosquitto/)
1. [VictoriaMertics](https://victoriametrics.com/)
1. [Grafana](https://grafana.com/)


## Getting Started
Deploy the stack into your Docker environment.



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









