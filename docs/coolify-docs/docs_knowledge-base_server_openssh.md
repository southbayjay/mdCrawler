Skip to content
Menu
Return to top
# OpenSSH ​
Coolify uses SSH to connect to your server and deploy your applications. This is true even when using the `localhost` server where Coolify is running.
Caution
The SSH key must not have a passphrase or 2FA enabled for the user used to run the Coolify installation script or the connection will fail.
## SSH Settings Configuration ​
These settings need to be configured manually before running the Coolify installation script:
  1. Edit SSH Configuration
bash```
nano /etc/ssh/sshd_config
```

Ensure these settings are present:
ssh```
PermitRootLogin prohibit-password
PubkeyAuthentication yes
```

Note
`PermitRootLogin` can be set to `yes`, `without-password`, or `prohibit-password`. We recommend `prohibit-password` for better security.
  2. Restart SSH Service


SystemD:
bash```
systemctl restart sshd
```

OpenRC:
bash```
rc-service sshd restart
```

## Manual Configuration ​
Note
The following steps are handled automatically by the Coolify installation script. Manual configuration is only needed if the automatic setup fails.
  1. Install OpenSSH Server
**Ubuntu/Debian/PopOS**
bash```
apt update && apt install -y openssh-server
systemctl enable --now sshd
```

**CentOS/RHEL/Fedora/Rocky**
bash```
dnf install -y openssh-server
systemctl enable --now sshd
```

**SLES/openSUSE**
bash```
zypper install -y openssh
systemctl enable --now sshd
```

**Arch Linux**
bash```
pacman -Sy --noconfirm openssh
systemctl enable --now sshd
```

**Alpine Linux**
bash```
apk add openssh
rc-update add sshd
rc-service sshd start
```

  2. Configure SSH Settings Follow the SSH Settings Configuration section above.
  3. Generate and Configure SSH Keys
bash```
# Generate SSH key pair
ssh-keygen -t ed25519 -a 100 \
  -f /data/coolify/ssh/keys/id.root@host.docker.internal \
  -q -N "" -C root@coolify
# Set correct ownership
chown 9999 /data/coolify/ssh/keys/id.root@host.docker.internal
# Add public key to authorized_keys
mkdir -p ~/.ssh
cat /data/coolify/ssh/keys/id.root@host.docker.internal.pub >> ~/.ssh/authorized_keys
# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

  4. Configure in Coolify UI
    1. Navigate to `Keys & Tokens` menu
    2. Go to `Private Keys` section
    3. Add the private key
    4. Configure the key in your localhost server settings


