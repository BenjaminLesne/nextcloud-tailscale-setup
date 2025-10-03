Useful links:

- [original setup](https://github.com/nextcloud/all-in-one/discussions/6817)

## 1. Configuring Tailscale Account

You will need to have a Tailnet already set up, and Tailscale installed and running on both your client and your server. For a quick intro, see: Tailscale quickstart (tailscale.com)

Once you have your Tailscale set up both on your server and your client, you need to enable the following configs in Tailscale from the DNS Tab (tailscale.com):

Make sure Magic DNS is enabled
Enable HTTPS Certificates

## 2. Installing Nextcloud AIO

Copy the contents from the default compose.yaml - Nextcloud AIO (github.com).

On your server, go to the directory where you will configure the Nextcloud AIO container and create a docker compose file (e.g.: ~/docker/nextcloud/compose.yaml). In there, paste the contents of the compose.yaml from their GitHub.

Inside that file, uncomment the environment section and the two properties shown below.

environment:
APACHE_PORT: 11000
APACHE_IP_BINDING: 127.0.0.1
In the compose.yaml directory, run the container with:

docker compose up -d

## 3. Setting up a domain

Nextcloud will require a proper domain with HTTPS. Depending on your setup, you can follow two routes. If Nextcloud is the only HTTPS service that you have in your machine, you can follow option A, however, if your machine subdomain is already taken by another service, option B or an alternative 2 should be used.

OPTION A: Tailscale serve
You can serve a domain from your machine serve using the following command. Notice the http://.

tailscale serve --bg http://127.0.0.1:11000
If you're running Systemd it is a good idea to set this as a service on startup, so create the service by pasting the following content into /etc/systemd/system/tailscale-serve-nextcloud.service:

[Unit]
Description=Serve Nextcloud backend through Tailscale
After=network.target

[Service]
ExecStart=/usr/bin/bash -c 'tailscale serve --bg http://127.0.0.1:11000'
Type=oneshot

[Install]
WantedBy=multi-user.target
And enable it and check that it's running

systemctl enable --now tailscale-serve-nextcloud.service
tailscale serve status

OPTION B: Tailscale Docker Proxy
If you have another service in your Tailnet that is already using the subdomain of your machine name, you can use TsDProxy (github.com) to launch Nextcloud or your other service as a separate machine, providing it with it's own subdomain. Below will be provided the minimum config to get it working. For more information about TsDProxy see the official documentation: TsDProxy - Docs (github.io).

Create a new directory to set up the TsDProxy (e.g.: ~/docker/tsdproxy)
Create a compose.yml in the newly created dir
Paste the following:
services:
tsdproxy:
image: almeidapaulopt/tsdproxy:1
volumes: - /var/run/docker.sock:/var/run/docker.sock - datadir:/data - ./config:/config
restart: unless-stopped
ports: - "8081:8080"

volumes:
datadir:
Change the first port number to an unused port if necessary. E.g.: 8082:8080. Don't use 8080 because you will need it (temporarily) for the Nextcloud setup wizard. You can change this later if needed.
Run the container with docker compose up -d
In your service's compose.yml file add the following property:
services:
SERVICE_NAME:

- labels:
-      - "tsdproxy.enable=true"
  Rebuild the Nextcloud container by going into the directory of the docker compose and run: docker compose down && docker compose up -d
  From your client, go to the IP address of TsDProxy. E.g.: http://100.123.145.67:8080
  You should arrive at a dashboard where you should see your service, click on Authenticate and complete the Tailscale login. You should see in the Tailscale Dashboard (tailscale.com) that a new ephemeral machine has been created with the name of your service.
  In the next section, when prompted with the Nextcloud domain, use the one that is running the Nextcloud docker container.

## 4. Nextcloud Setup Wizard

On your client, which should be running Tailscale and having its DNS at 100.100.100.100 (or any other valid Tailscale IP). Open your browser and open the Tailscale IP address of your server followed by port 8080. E.g.: 100.123.145.67:8080. You can get its IP from the Tailscale dashboard (tailscale.com) and choosing the one that corresponds with the machine that is running the container.

Tip

Make sure that no other service is using port 8080 on your server
If you hit any DNS related issue, try running resolvectl status from the client, you should see Tailscale with "Default Route" as yes.
You should arrive at the Nextcloud setup page, there follow the Nextcloud instructions:

Copy the passphrase (and store it)
Log in, and paste the passphrase
Tip

If you want to use a custom Tailscale domain name, change it before submitting the domain on Nextcloud. Otherwise things will break, and you might be better off restarting the Nextcloud containers from scratch. You can change it by clicking the "Rename Tailnet" button at DNS (tailscale.com)

It will ask you for the domain of your server, the domain will be the one that we have set up earlier.

If you followed option A you can get it from:

tailscale serve status
Alternatively, regardless of what you used, it will be visible via the Tailscale Dashboard (tailscale.com). Click on the IP address, and copy the one that is formatted like: name.tail0a12b3.ts.net, then paste it on the domain field.

Note

The Tailscale domain is a very convenient way of having a certified HTTPS domain that only you can access. We need to use a domain since Nextcloud requires it.

Afterwards, continue the setup wizard: select your desired containers, set up the TZ, the backup location, and finally "Download and start containers".

Once that finishes setting up, try opening the domain that you configured Nextcloud to run on in your web browser, and you should see yourself in the login page!
