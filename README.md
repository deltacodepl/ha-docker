# <img src="images/t9_logo.png" height="25"> Tyzen9 - docker-homeassistant
This is an all-in-one Docker stack for running home assistant and some common "Add-Ons".  This will allow for the running of Home Assistant in a Docker instance, providing the support for ESPHome, MQTT and editing configurations with VSCode in a browser.  This will also offer the means to support long-term storage of sensor values over time, and build beautiful visual reports with Grafana.

<p align="center">
<img src="images/home-assistant-logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <img src="images/esphome_logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp;<img src="images/vscode_logo.png" height="35" style="padding-left: 25px">
<br>
<img src="images/mqtt-logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <img src="images/vm_logo.png" height="35">&nbsp;&nbsp;&nbsp;&nbsp;<img src="images/grafana-logo.png" height="35" style="padding-left: 25px">  
</p> 

## Prerequisites
In production it's generally best to use [Docker Engine](https://docs.docker.com/get-docker/) on a Linux host operating system, and a lightweight service delivery platform designed for managing containerized applications such as [Portainer](https://www.portainer.io/).

This documentation assumes you have a working knowledge of [Docker](https://www.docker.com/), and [Home Assistant](https://www.home-assistant.io/) administration. It is designed to get a Home Assistant environment up and running with the included Docker services.

## Container Configuration
This `docker-compose` implementation is configured using the `environment` section(s) of the `docker-compose.yaml` file.  The values between the `${}` characters are substituted from the `.env` file.  If these values are not configured, the default values preceded by the hyphen will be used.

> [!Note]
> These containers are configured to run as `root`. While is it not impossible to configure them to run otherwise, working to do so otherwise was just too many headaches. After all, HASOS runs as `root` out of the box 🙌.


# What's Inside?
The docker-homeassistant stack contains everything you need for a stable Home Assistant experience running in Docker. Here is what this project has to offer:

1. [Home Assistant](https://www.home-assistant.io/)
1. [VSCode Editor](https://docs.linuxserver.io/images/docker-code-server/)
1. [ESPHome](https://esphome.io/)
1. [MQTT](https://hub.docker.com/_/eclipse-mosquitto/)
1. [VictoriaMertics](https://victoriametrics.com/)
1. [Grafana](https://grafana.com/)
1. [Network UPS Tools (nut)](https://hub.docker.com/r/instantlinux/nut-upsd)


## Getting Started
Deploy the stack into your Docker environment. This can be done any number of ways, to get started quickly log onto your Docker host system, and clone this repository: `git clone https://github.com/tyzen9/docker-homeassistant.git`

1. Set the configuration options as desired in the `docker-compose.yaml` and `.env` files.
1. Navigate to the project's root directory and run the following command:

```
docker compose up
```

If everything works as expected, you should be able to access home assistant at http://hostname:8123

## Home Assistant
Visit the Home Assistant (HA) instance at http://hostname:8123, and the Home Assistant Welcome screen should appear.
Click "Create my smart home", followed by creating a user. Click through the rest of the dialogs as appropriate until you reach the out-of-the-box Home Assistant dashboard.

If you are planning to use the VSCODE Server, then generate a long-lived token for VSCode by clicking on the username in Home Assistant's left hand navigation, and choosing the security tab.  Scroll to the bottom to create a long-lived token and save the token generated in a text file temporarily, we will need this for VSCode Server.

## VS Code
Visit the VS Code Server at http://hostname:8443.  This will prompt for the password configured in the stack environment to be entered, do this and click SUBMIT to gain access to the VSCode editor.

In the extensions settings, install:
- [Home Assistant Config Helper](keesschollaart.vscode-home-assistant) (by keesschollaart) 


Then, make sure the following settings are present in the: `[DEFAULT_WORKSPACE]/.vscode/settings.json` file within the config volume.  You may need to create the `.vscode` directory and the `settings.json` file:

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

## Additional Recommended Extensions ##
These are common VSCode extensions that are used when editing Home Assistant and ESPHome YAML files that are recommended for installation in the VSCode server instance:

1. YAML (redhat.vscode-yaml)
1. Log File Highlighter (by emilast)
1. ESPHome (by ESPHome)
1. Material Design Icons Intellisense (by Lukas Troyer)
1. indent-rainbow (by oderwat)
1. vscode-icons (by vscode-icons-team)
1. Jinja (samuelcolvin.jinjahtml)
1. Prettier (esbenp.prettier-vscode)

## 📢 Connect a local VSCode instance to the Home Assistant config files
This approach uses VSCode's Remote SSH extension to connect to your Docker host, then uses Dev Containers to access the containers directly.  This is a convenient way to use VSCode natively rather than using the VSCode editor in this stack

In the local VSCode instance install the extensions listed above, in addition to the following:

1. Remote-SSH extension (ms-vscode-remote.remote-ssh)
1. Dev Containers extension (ms-vscode-remote.remote-containers)
1. Docker extension (ms-azuretools.vscode-docker)

To connect to the Home Assistant (or ESPHome container) follow these steps:

1. Open VSCode Command Palette (F1)
1. Run "Remote-SSH: Connect to Host"
1. Add your Docker host connection details 
1. Once connected via SSH, use the Docker extension sidebar to find the homeassistant (or esphome) containers
1. Right-click on either container and select "Attach Visual Studio Code"
1. Select the appropriate container, and enter password if needed
1. Navigate the the appropriate folder and click OK


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

## VictoriaMetrics
Since InfluxDB began to focus on enterprise customers, and limiting the capabilities of their core offering for non-commercial users, VictoriaMetrics has been gaining some momentum in the Home Assistant community. Integrating VictoriaMetrics provide a high performing and lightweight long-term data storage solution that replaces InfluxDB.

> [!Note]
> This method uses Home Assistant's InfluxDB integration with VictoriaMetrics' InfluxDB API compatibility.

Once you start the VictoriaMetrics container, it will be accessible at: http://hostname:8428/vmui

Lets configure Home Assistant to pass sensor data to VictoriaMetrics. Add the following to the Home Assistant configuration.yaml file, and RESTART Home Assistant.

```yaml
influxdb:
    api_version: 1
    host: localhost
    port: 8428
    ssl: false
    max_retries: 3
    measurement_attr: entity_id
    tags_attributes:
    - friendly_name
    - unit_of_measurement
    ignore_attributes:
    - icon
    - source
    - options
    - editable
    - min
    - max
    - step
    - mode
    - marker_type
    - preset_modes
    - supported_features
    - supported_color_modes
    - effect_list
    - attribution
    - assumed_state
    - state_open
    - state_closed
    - writable
    - stateExtra
    - event
    - friendly_name
    - device_class
    - state_class
    - ip_address
    - device_file
    - unit_of_measurement
    - unitOfMeasure
    include:
    domains:
        - sensor
        - binary_sensor
        - light
        - switch
        - cover
        - climate
        - input_boolean
        - input_select
        - number
        - lock
        - weather
    exclude:
    entity_globs:
        - sensor.clock*
        - sensor.date*
        - sensor.glances*
        - sensor.time*
        - sensor.uptime*
        - sensor.dwd_weather_warnings_*
        - weather.weatherstation
        - binary_sensor.*_smartphone_*
        - sensor.*_smartphone_*
        - sensor.adguard_home_*
        - binary_sensor.*_internet_access
```

Once Home Assistant restarts, visit VictoriaMetrics at: http://hostname:8428/vmui, and enter click `Query` and paste the following:

```
group by(__name__) ({__name__!=""})
```

You should see a list of metrics names, including, for example, `weather.forecast_home_temperature`.

## Grafana
Grafana is used to communicate with VictoriaMetrics to create dashboards and visuals of your ViewPoint sensor data. You can access Grafana at http://hostname:3000, and login with the username: `admin` and password: `admin`.  Next set a new admin password and you will land on the home page.

In Grafana, navigate to **Menu** > **Connections** > **VictoriaMetrics**. Install the VictoriaMetrics datasource plugin for Grafana. Once installed, click Add a new data source and specify the following settings:

- Name: Victoriametrics
- URL: http://localhost:8428 (this connection uses the internal Docker network)
- Click Save & test. 

You should get the message: *Data source is working*

You now have Grafana and VictoriaMetrics connected and are ready to start [learning to build Grafana visuals](https://grafana.com/docs/grafana/latest/dashboards/).

##  Network UPS Tools (nut) ##
Network UPS Tools, or `nut`, is common to those that have a battery backup on their Home Assistant server that offers a data port. The UPS devices to be monitored are configured in the `/etc/nut` directory of the `nut` container. 

> [!Note]
> If you do not need `nut`, then you can comment this service out entirely. 


The `ups.conf` file configures the UPS devices to be connected to, below is a sample of two UPS devices:

> [!Note]
> Some work will be need to be done to research the port path that each UPS is connected to, and the serial number of each device - (Google is your friend) 

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

Next, the `upsmon.conf` files needs to be updated to tell `nut` what UPS devices to monitor, and the username to use.  `password` in this context is just telling upsmon to use a password for API authentication.  This was set up in compose with the `API_PASSWORD` property.

```conf
MONITOR ups-1@localhost 1 upsmon password primary
MONITOR ups-2@localhost 1 upsmon password primary
RUN_AS_USER nut
```

Once these changes are made, restart the `nut` container, and try to initialze the `nut` integration in Home Assistant.







