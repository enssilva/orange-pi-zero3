# orange-pi-zero3
## Configure Split DNS
You need to configure the domain `home.local` in the tailscale:
* Go to `DNS` tab
* Click in `Add nameserver`
* Choose `Custom...`
* Add the Orange PI zero3 IP as *Nameserver*
* Activate `Restrict to domain`
* Add the the domain **home.local**

Source: https://tailscale.com/kb/1054/dns#restricted-nameservers

## Generate key to ssl configuration
Generate the CA private key:
```bash
openssl genrsa -out files/myCA.key 3072
```
Generate the CA certificate:
```bash
openssl req -x509 -new -nodes -key files/myCA.key -sha256 -days 1825 -out files/myCA.crt -reqexts v3_req -extensions v3_ca -subj "/C=BR/ST=ES/L=Vitoria/O=Home Local/CN=home.local"
```
Generate traefik private key:
```bash
openssl genrsa -out files/traefik.key 3072
```
Generate traefik Certificate Signing Request (CSR):
```bash
openssl req -new -key files/traefik.key -out files/traefik.csr -subj "/C=BR/ST=ES/L=Vitoria/O=Home Local Traefik/CN=traefik.home.local"
```
Create the configuration file in **files/traefik.ext** as:
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:TRUE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = home.local
DNS.2 = *.home.local
```
Generate traefik SSL certificate:
```bash
openssl x509 -req -in files/traefik.csr -CA files/myCA.crt -CAkey files/myCA.key -CAcreateserial -out files/traefik.crt -days 825 -sha256 -extfile files/traefik.ext
```
Source: https://scriptcrunch.com/create-ca-tls-ssl-certificates-keys/
## Add CA certificate
### Android
You need to install the myCA.crt on android to the Bitwarden APP work. Copy the certificate myCA.crt to the android and search in the configuration for CA certificate.
### Ubuntu
Use those commands to install the CA certificate:
```bash
sudo cp files/myCA.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```
Source: https://documentation.ubuntu.com/server/how-to/security/install-a-root-ca-certificate-in-the-trust-store/index.html
## Joplin
The default user/login is:
* Username: admin@localhost
* Password: admin