				# Setup squid + spoofdpi = you tube 

Step #1. Installing sppoofdpi on linux.

Run
```bash
curl -fsSL https://raw.githubusercontent.com/xvzc/SpoofDPI/main/install.sh | bash -s linux-amd64
```

Create file for service.
```bash 
touch /etc/systemd/system/spoofdpi.service
```
Insert the following content into the file
```
--------------
[Unit]
Description=spoofDPI service
After=network.target

[Service]
ExecStart=/root/.spoofdpi/bin/spoofdpi -port 3138 -addr 127.0.0.1 -window-size 1
Restart=always
User=root
WorkingDirectory=/root/.spoofdpi/bin/
Environment="PATH=$PATH:/root/.spoofdpi/bin"

[Install]
WantedBy=multi-user.target
---------------
```

Re-reading the systemd.
```bash
systemctl daemon-reload 
```
Starting  service.
```bash
systemctl start spoofdpi.service
```
```bash
systemctl enable spoofdpi.service 
```
Shecking the status of. 
```bash
systemctl status spoofdpi.service
```
Reading a magazine
```bash
journalctl -u spoofdpi.service
```

Step #2 Configuring squid on an upstream proxy.

Adding the following parameters to the squid configuration.
```
# The parent proxy spoofdpi (installed here on the localhost)
cache_peer 127.0.0.1 parent 3138 3138 no-digest allow-miss no-query

# resources for proxying through the parent proxy spoofdpi
acl spoofdpi_url_regex url_regex -i "/etc/squid/lists/spoofdpi_url_regex.txt "

# Restriction (permission) of access to proxy servers based on ACL
cache_peer_access 127.0.0.1 allow spoofdpi_url_regex RawNet auth_users InternetAccess

# requests are never made directly, but to use the parent proxy in cache_peer. allow - allow
never_direct allow spoofdpi_url_regex LocalNet auth_users InternetAccess
never_direct deny all

# requests are always made directly, bypassing the parent proxy (in cache_peer). deny - prohibit such requests.
always_direct deny spoofdpi_url_regex LocalNet auth_users InternetAccess
always_direct allow all

``` 
```bash
touch /etc/squid/lists/spoofdpi_url_regex.txt
```

Add to File spoofdpi_url_regex.txt the next line.

```
(https?://)?([a-z0-9-]+\.)?(youtube|ytimg)\.([a-z]{2,}|com|net|org)(/.*)?
```
