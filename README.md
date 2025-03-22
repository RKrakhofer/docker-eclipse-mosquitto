# Mosquitto Docker Compose Setup

This repository contains a Docker Compose configuration for setting up an Eclipse Mosquitto MQTT broker with authentication and health checks.

## Prerequisites

- Docker
- Docker Compose

## Configuration

### Environment Variables

Create a `.env` file in the root of the repository with the following content:

```plaintext
MQTT_USER=your_healthcheck_username
MQTT_PASSWORD=your_healthcheck_password
```

Replace `your_healthcheck_username` and `your_healthcheck_password` with the desired MQTT credentials for the health check user.

**Note**: The user specified in these environment variables is intended only for the health check and should not have write access to any MQTT topic. Ensure that this user has read-only permissions.

### Mosquitto Configuration

Ensure you have the following files in the `mosquitto-config-volume`:

- `mosquitto.conf`: Mosquitto configuration file
- `passwordfile`: Mosquitto password file created using `mosquitto_passwd`

Example `mosquitto.conf`:

```plaintext
allow_anonymous false
password_file /mosquitto/config/passwordfile
```

To restrict the health check user to read-only access, you can add ACL (Access Control List) rules in the `aclfile`. Example `aclfile` content:

```plaintext
user your_healthcheck_username
topic read #

# other user permissions
```

Then, reference the `aclfile` in your `mosquitto.conf`:

```plaintext
acl_file /mosquitto/config/aclfile
```

## Docker Compose File

Below is the `docker-compose.yaml` file used to set up the Mosquitto service:

```yaml
version: '3.8'

services:
  mosquitto:
    image: eclipse-mosquitto
    container_name: mosquitto
    ports:
      - "1883:1883"
    volumes:
      - mosquitto-config-volume:/mosquitto/config
      - mosquitto-data-volume:/mosquitto/data
      - mosquitto-log-volume:/mosquitto/log
    environment:
      - MQTT_USER=${MQTT_USER}
      - MQTT_PASSWORD=${MQTT_PASSWORD}
    healthcheck:
      test: ["CMD", "mosquitto_sub", "-u", "${MQTT_USER}", "-P", "${MQTT_PASSWORD}", "-h", "localhost", "-t", "#", "-C", "1"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  mosquitto-config-volume:
    external: true
  mosquitto-data-volume:
    external: true
  mosquitto-log-volume:
    external: true
```

## Running the Service

To start the Mosquitto service, run the following command:

```sh
docker-compose up
```

This will start the Mosquitto service with the specified configuration and environment variables.

## Health Check

The health check is configured to ensure the Mosquitto service is running correctly by subscribing to a topic using the provided MQTT credentials.

## Volumes

The Docker Compose file uses external volumes for the Mosquitto configuration, data, and logs. Ensure these volumes exist before running the service.

- `mosquitto-config-volume`
- `mosquitto-data-volume`
- `mosquitto-log-volume`

## Troubleshooting

If you encounter any issues, ensure that the environment variables are correctly set and that the Mosquitto configuration files are correctly placed in the `mosquitto-config-volume`.

## License

This project is licensed under the MIT License.
