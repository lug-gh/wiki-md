# Using LDAPS with a certificate signed by an untrusted (e.g. internal) CA

## Problem

When connecting Nextcloud to an LDAP server via LDAPS, the connection fails if the server certificate was signed by a CA that is not in the trusted CA list used by the `php-fpm` process of the snap (e.g. a company-internal CA).

Disabling certificate validation works as a workaround, but it weakens security and is not recommended.

## Solution

Provide the CA certificate to `snap.nextcloud.php-fpm` via the `LDAPTLS_CACERT` environment variable.

The systemd unit (`/etc/systemd/system/snap.nextcloud.php-fpm.service`) reads its environment variables from `/etc/environment`, so the variable can be set there.

### 1. Store the CA certificate

Place the CA certificate (or the full chain) in a directory the snap can access. A good choice is:

```
/var/snap/nextcloud/current/certs/custom/
```

This is also the place for custom certificates used by Nextcloud itself. In principle, any path listed in the output of `nextcloud.enable-https custom -h` can be used.

### 2. Set the environment variable

Add the following line to `/etc/environment`:

```
LDAPTLS_CACERT="/var/snap/nextcloud/current/certs/custom/chain.pem"
```

### 3. Restart PHP-FPM

```
sudo systemctl restart snap.nextcloud.php-fpm
```

Nextcloud can now validate the LDAPS server certificate against your CA.

## Note: Do not use a systemd override

Setting the variable through a systemd drop-in (`override.conf`) works initially:

```ini
[Service]
Environment="LDAPTLS_CACERT=/var/snap/nextcloud/current/certs/custom/chain.pem"
```

However, the override is **not** preserved when the snap performs an automatic refresh, so LDAPS stops working after the update. The `/etc/environment` method is persistent and survives snap updates.
