## Install Home Assistant Container

{% if page.installation_type != 'alternative' %}
These below instructions are for an installation of Home Assistant Container running in your own container environment, which you manage yourself. Any [OCI](https://opencontainers.org/) compatible runtime can be used, however this guide will focus on installing it with Docker.

<div class='note'>
<b>Prerequisites</b>

This guide assumes that you already have an operating system setup and a container runtime installed (like Docker).
  
If you are using Docker then you need to be on at least version 19.03.9, ideally an even higher version, and `libseccomp` 2.42 or newer.
</div>

### Platform Installation

Installation with Docker is straightforward. Adjust the following command so that:

* `/PATH_TO_YOUR_CONFIG` points at the folder where you want to store your configuration and run it.
* `MY_TIME_ZONE` is a [tz database name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), like `TZ=America/Los_Angeles`.

{% endif %}

{% if page.installation_type == 'raspberrypi' %}
#### Raspberry pi 3

  {% include installation/container/cli.md image=site.installation.container.raspberrypi3 %}

#### Raspberry pi 4

  {% include installation/container/cli.md image=site.installation.container.raspberrypi4 %}

{% elsif page.installation_type == 'alternative' %}
  {% include installation/container/alternative.md %}

{% else %}
  {% include installation/container/cli.md %}
{% endif %}

Once the Home Assistant Container is running Home Assistant should be accessible using `http://<host>:8123` (replace <host> with the hostname or IP of the system). You can continue with onboarding.

{% include getting-started/next_step.html step="Onboarding" link="/getting-started/onboarding/" %}

### Restart Home Assistant

If you change the configuration you have to restart the server. To do that you have 3 options.

1. In your Home Assistant UI go to the **Configuration** panel -> **Server management** and click the "Restart" button.
2. You can go to the **Developer Tools** -> **Services**, select the service `homeassistant.restart` and click "Call Service".
3. Restart it from a terminal.

{% tabbed_block %}

- title: Docker CLI
  content: |

    ```bash
    docker restart homeassistant
    ```

- title: Docker Compose
  content: |

    ```bash
    docker-compose restart
    ```

{% endtabbed_block %}

### Docker Compose

<div class="note tip">

  In order to use `docker-compose` you first need to [install `docker-compose`](https://docs.docker.com/compose/install/) on your system.

</div>

As the Docker command becomes more complex, switching to `docker-compose` can be preferable and support automatically restarting on failure or system restart. Create a `docker-compose.yml` file:

{% if page.installation_type == 'raspberrypi' %}
#### Raspberry pi 3

  {% include installation/container/compose.md image=site.installation.container.raspberrypi3 %}

#### Raspberry pi 4

  {% include installation/container/compose.md image=site.installation.container.raspberrypi4 %}
{% else %}
  {% include installation/container/compose.md %}
{% endif %}

Start it by running:

```bash
docker-compose up -d
```

### Exposing Devices

In order to use Z-Wave, Zigbee or other integrations that require access to devices, you need to map the appropriate device into the container. Ensure the user that is running the container has the correct privileges to access the `/dev/tty*` file, then add the device mapping to your container instructions:

{% tabbed_block %}

- title: Docker CLI
  content: |

    ```bash
    docker run ... --device /dev/ttyUSB0:/dev/ttyUSB0 ...
    ```

- title: Docker Compose
  content: |

    ```yaml
    version: '3'
    services:
      homeassistant:
        ...
        devices:
          - /dev/ttyUSB0:/dev/ttyUSB0
    ```

{% endtabbed_block %}

### Optimizations

The Home Assistant Container is using an alternative memory allocation library [jemalloc](http://jemalloc.net/) for better memory management and Python runtime speedup.

As jemalloc can cause issues on certain hardware, it can be disabled by passing the environment variable `DISABLE_JEMALLOC` with any value, for example:

{% tabbed_block %}

- title: Docker CLI
  content: |

    ```bash
    docker run ... -e "DISABLE_JEMALLOC=true" ...
    ```

- title: Docker Compose
  content: |

    ```yaml
    version: '3'
    services:
      homeassistant:
      ...
      environment:
        DISABLE_JEMALLOC: true
    ```

{% endtabbed_block %}

The error message `<jemalloc>: Unsupported system page size` is one known indicator.
