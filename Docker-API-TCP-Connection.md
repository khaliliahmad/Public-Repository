Docker Systemd Override Steps
To resolve the conflict between `/etc/docker/daemon.json` and the default Docker service configuration,
also with this configuration user can call docker api by tcp connections :
Please follow these steps:

1. Create the Override Directory
Run the following command to create the necessary directory:

```

sudo mkdir -p /etc/systemd/system/docker.service.d

```

2. Create the Override File
Create a file named `override.conf` inside that directory:

```

sudo nano /etc/systemd/system/docker.service.d/override.conf

```

3. Add the Configuration
Paste the following content into the file. The empty `ExecStart=` line is crucial as it clears the default flags:

```

[Service]

ExecStart=

ExecStart=/usr/bin/dockerd

```

4. Reload and Restart Docker
Apply the changes by reloading the systemd manager and restarting the Docker service:

```

sudo systemctl daemon-reload

sudo systemctl restart docker

```
