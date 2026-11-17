# sonyftp

A secure FTPS server you start when you need it, for uploading photos from a Sony camera. Runs on macOS and Linux and is built on [pure-ftpd](https://www.pureftpd.org/). Verified with an A7 IV.

- Uses TLS only (logins and file data); plain FTP is refused
- The camera verifies the server against your own private root CA, with no "Root Certificate Error"
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

1. Copy `certs/cacert.pem` to the root of the memory card in **slot 1**, then go to **Network → Network Option → Import Root Certificate**.
2. In **FTP Transfer Func → Server Setting**, set:
   - Hostname: the IP from `./sonyftp status`
   - Secure Protocol: On
   - Root Certificate Error: Does Not Connect
   - Port: 21
   - Passive Mode: On
   - User / Password: from `.env`
   - Directory: empty
3. Turn on **FTP Function** and transfer.

Give the server a fixed IP (a DHCP reservation on your router) so the camera setting keeps working. Keep `certs/ca.key` private.

## How the camera verifies the server

Importing the root certificate is not enough. Before each transfer the camera makes a separate check connection and asks an OCSP responder whether the server certificate has been revoked. A certificate without an OCSP address fails that check even when it chains correctly to the imported root.

So `sonyftp` puts an OCSP address in the server certificate and runs a small responder (`openssl ocsp`, port 8022) next to the FTP server. `start` and `stop` handle both.

With **Root Certificate Error** set to connect anyway, a failed check does not stop the transfer. The camera reconnects without verifying the server and shows "Transferring... (Root Certificate Error)". The data is still encrypted, but the camera could be talking to any server.

## Troubleshooting

| On the camera | Cause | Fix |
| --- | --- | --- |
| Failed to connect to server | Wrong Hostname, or the camera is on another network | Use the IP from `./sonyftp status`; put the camera on the same network |
| Root Certificate Error | The root was not imported, or the camera cannot reach the OCSP responder | Import `certs/cacert.pem` again; run `./sonyftp test`; allow port 8022 through your firewall |
| Root Certificate Error after the server's IP changed | The certificate was reissued for the new IP | Update Hostname on the camera (no new import needed) |

`./sonyftp test` checks the FTPS upload, the OCSP answer, and that plain FTP is refused.
