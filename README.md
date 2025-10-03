## Why I made this

I could not manage to find a guide to setup NextCloud and Tailscale without getting stuck. The best setup I found was almost perfect, I ran into small issues here and there and this is why I made this repository. So people can see an actual final compose.yaml that does work. I also try to remove as much noise as possible from the [original setup](https://github.com/nextcloud/all-in-one/discussions/6817) I used to make this repository.

## 1. Configuring Tailscale Account

You will need to have a Tailnet already set up, and Tailscale installed and running on both your client and your server, let's say your phone and your desktop.

See the [intallation steps here for both device](https://tailscale.com/download/linux/arch)

Once you have your Tailscale set up both on your Desktop and your phone, you need to enable the following configs in Tailscale from [the DNS Tab](https://login.tailscale.com/admin/dns):

- Make sure Magic DNS is enabled
- Enable HTTPS Certificates

## 2. Installing Nextcloud AIO

Copy the compose.yaml file from this repository to your desktop computer.

[Here](https://github.com/BenjaminLesne/nextcloud-tailscale-setup/pull/1#pullrequestreview-3298399391) you can find all the changes I applied to the [default compose.yaml](https://github.com/nextcloud/all-in-one/blob/main/compose.yaml) from the Nextcloud AIO repository.

In the directory of the compose.yaml you just copied, run the container with:

```bash
docker compose up -d
```

## 3. Setting up a domain

Nextcloud will require a proper domain with HTTPS. You can serve a tailnet domain from your desktop using the following command.

```bash
tailscale serve --bg http://127.0.0.1:11000
```

If you're running Systemd it is a good idea to set this as a service on startup, so create the service by pasting the following content into `/etc/systemd/system/tailscale-serve-nextcloud.service`:

```ini
[Unit]
Description=Serve Nextcloud backend through Tailscale
After=network.target

[Service]
ExecStart=/usr/bin/bash -c 'tailscale serve --bg http://127.0.0.1:11000'
Type=oneshot

[Install]
WantedBy=multi-user.target
```

## 4. Nextcloud Setup Wizard

On your desktop, open your browser and open the Tailscale IP address of your server followed by port 8080. E.g.: 100.123.145.67:8080. You can get its IP from the [Tailscale dashboard](https://login.tailscale.com/admin/machines) by clicking on the address that corresponds to your desktop machine.

You should arrive at the Nextcloud setup page, there follow the Nextcloud instructions:

- Copy the passphrase (and store it)
- Log in, and paste the passphrase

It will ask you for the domain of your server, the domain will be the one that we have set up earlier.

You can get it from:

```bash
tailscale serve status
```

Alternatively, it will be visible via the [Tailscale Dashboard](https://login.tailscale.com/admin/machines). Click on the IP address, and copy the domain name that is formatted like: `name.tail0a12b3.ts.net`, then paste it on the domain field requested by NextCloud in your browser.

Afterwards, continue the setup wizard: select Nextcloud Hub 25 if you want (I did), select your desired containers (I selected none), set up the TimeZone, and finally "Download and start containers".

Once that finishes setting up, try opening a browser tab with the tailnet domain that you provided to Nextcloud, you should get to the login page!
Now try with your phone with tailscale running, you should get to the login page too!
