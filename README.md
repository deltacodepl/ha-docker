
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


## Getting Started
Deploy the stack into your Docker environment.

> [!Note]
> This stack is using [LinuxServer.io](https://docs.linuxserver.io/images/docker-homeassistant/)'s Home Assistant image.  This image allows Home Assistant to run as a user other than `root`. 

## Home Assistant
Visit the Home Assistant (HA) instance at http://hostname:8123, and the Home Assistant Welcome screen should appear.
Click "Create my smart home", followed by creating a user. Click through the rest of the dialogs as appropriate until you reach the out-of-the-box Home Assistant dashboard.

If you are planning to use the VSCODE Server, then generate a long-lived token for VSCode by clicking on the username, and choosing the security tab.  Save the token generated in a text file temporarily, we will need this if VSCode Server will be used to edit HA's YAML files.

## VS Code
Visit the VS Code Server at http://hostname:8443.  This will prompt for the password configured in the stack environment to be entered, do this and click SUBMIT to gain access to the VSCode editor.

In the extensions settings, install:
- [Home Assistant Config Helper](keesschollaart.vscode-home-assistant) (by keesschollaart) 

Open the command pallet `[ctrl + shift + p]` and choose `Home Assistant: Manage Home Assistant Authentication`, and choose `Set Token`
- Enter the URL to the Home Assistant instance (http://hostname:8123)
- Enter the long-lived token

Test the connection using the `Home Assistant: Test Home Assistant Connection` command pallet option, and hopefully a `Successfully Connected` message is displayed in the bottom right corner.

Restart the stack, and reload the VSCode editor.

Open the command pallet, and choose `Home Assistant: Check Configuration (remote!)`

### Test File Edits ###
Next, make a small edit to the configuration.yaml file (like adding a comment), and attempt to save the change.  
  

## Additional Extensions ##

1. YAML (by Red Hat)
1. Log File Highlighter (by emilast)
1. ESPHome (by ESPHome)
1. Material Design Icons Intellisense Preview (by lukas-tr)
1. indent-rainbow (by oderwat)
1. vscode-icons (by vscode-icons-team)


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










