# Key Managment System Using (PyKMIP) and VMs

This project sets up a Key Management System (KMS) using PyKMIP, a Python implementation of the Key Management Interoperability Protocol (KMIP).

The project involves setting up a virtual machine (VM) as the KMS server and configuring a client to use the KMS.

## VM Setup

1. Create a VM using virtualization software of your choice.
2. Load the Ubuntu server ISO and complete the setup.
3. After the setup is complete, reboot the VM.
4. Eject the ISO and restart the VM.
5. Run the following commands in the terminal to update and upgrade Ubuntu and install the desktop user interface:

```sh
sudo apt update
sudo apt full-upgrade
sudo apt install ubuntu-desktop^
```

## Server Setup

1. SSH into the VM as the `client` user.
2. Install the PyKMIP dependencies and create the necessary directories:

```sh
sudo -i
apt-get update
apt-get upgrade
mkdir /usr/local/PyKMIP
mkdir /etc/pykmip
mkdir /var/log/pykmip
chown client: -R /usr/local/PyKMIP
chown client: -R /etc/pykmip
chown client: -R /var/log/pykmip
apt-get install python3-dev libffi-dev libssl-dev libsqlite3-dev
```

3. Create a self-signed certificate for TLS authentication:

```sh
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/selfsigned.key -out /etc/ssl/certs/selfsigned.crt
```

When prompted for the Common Name, enter the VM's IP address.

4. Change ownership of the certificate files:

```sh
chown client: -R /etc/ssl/private
chown debian: /etc/ssl/certs/selfsigned.crt
exit
```

5. Clone the PyKMIP repository and install it:

```sh
cd /usr/local
git clone https://github.com/OpenKMIP/PyKMIP
cd /usr/local/PyKMIP
sudo python3 setup.py install
```

6. Configure the PyKMIP server by editing the `/etc/pykmip/server.conf` file:

```sh
nano /etc/pykmip/server.conf
```

Edit the file with the following settings:

```ini
[server]
database_path=/etc/pykmip/pykmip.database
hostname=192.168.64.9
port=5696
certificate_path=/etc/ssl/certs/selfsigned.crt
key_path=/etc/ssl/private/selfsigned.key
ca_path=/etc/ssl/certs/ca-certificates.crt
auth_suite=TLS1.2
policy_path=/usr/local/PyKMIP/examples/
enable_tls_client_auth=False
tls_cipher_suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
logging_level=DEBUG
```

7. Make the PyKMIP server run automatically when the VM starts by editing the crontab file:

```sh
crontab -e
```

Add the following line:

```ini
@reboot ( sleep 30s; python3 /usr/local/PyKMIP/bin/run_server.py & )
```

8. Reboot the VM.

9. Tests
- Check background processes:
``` ps aux | grep PyKMIP```

## CLIENT SETUP:

Setup PyKMIP Process:
Installing pyKMIP and dependencies: 
**Same as server**

Configuration File:
nano /etc/pykmip/pykmip.conf
```
[client]
host=192.168.64.9 (server’s  VM IP)
port=5696
keyfile=/etc/ssl/private/selfsigned.key
certfile=/etc/ssl/certs/selfsigned.crt
cert_reqs=CERT_REQUIRED
ssl_version=PROTOCOL_TLS
ca_certs=/etc/ssl/certs/ca-certificates.crt
do_handshake_on_connect=True
suppress_ragged_eofs=True
username=pykmip
password=password
```

## Client Usage of KMS:
Go to: https://github.com/OpenKMIP/PyKMIP/tree/master/kmip/demos/pie

You will find many script that you can run from the client VM

## Attributions
This project makes use of PyKMIP, an open source implementation of the Key Management Interoperability Protocol (KMIP) in Python. PyKMIP is developed by the OpenKMIP project and can be found on GitHub at https://github.com/OpenKMIP/PyKMIP. 

