# sonyftp

A secure FTPS server you start when you need it, for uploading photos from a Sony camera (tested setup: A7 IV). Runs on macOS and Linux and is built on [pure-ftpd](https://www.pureftpd.org/).

- Uses TLS only (logins and file data); plain FTP is refused
- The camera checks the server's certificate against your own private root CA
- A single virtual FTP user, confined to the upload folder
- Runs only between `start` and `stop`

## Install

```sh
brew install pure-ftpd openssl          # macOS
sudo apt install pure-ftpd openssl curl # Debian/Ubuntu (then: sudo systemctl disable --now pure-ftpd)

./sonyftp setup   # creates .env, certificates, the FTP user and the upload folder
```

Settings live in `.env` (see `.env.example`). The password, certificates and keys stay on the machine and are gitignored.

## Use

```sh
./sonyftp start | stop | status | log | test
./sonyftp passwd   # new password
./sonyftp cert     # new server cert (start does this automatically if the IP changed)
```

## Camera (A7 IV)

1. Copy `certs/cacert.pem` to the root of the memory card, then go to **Network → Network Option → Import Root Certificate**.
2. In **FTP Transfer Func → Server Setting**, set:
   - Hostname: the IP from `./sonyftp status`
   - Secure Protocol: On
   - Port: 21
   - Passive Mode: On
   - User / Password: from `.env`
3. Turn on **FTP Function** and transfer.

Give the server a fixed IP (a DHCP reservation on your router) so the camera setting keeps working. Keep `certs/ca.key` private.
