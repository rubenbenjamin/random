## Setting up https with a valid certificate for a local domain
###Setting up and installing a certificate #

First create a new folder called cert in your project folder. This is where we will save our certificates and keys. If you plan to use git for the project, I recommend adding the cert folder to your .gitignore file.

###Create the root.cnf
Create the file root.cnf with the following content and save it in the cert folder.
```
# OpenSSL configuration for Root CA

[ req ]

prompt             = no
string_mask        = default

# The size of the keys in bits:
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = x509_ext

[ req_distinguished_name ]

# Note that the following are in 'reverse order' to what you'd expect to see.

countryName = US
organizationName = Ruben Benjamin
commonName = Ruben Benjamin Root CA

[ x509_ext ]

basicConstraints=critical,CA:true,pathlen:0
keyUsage=critical,keyCertSign,cRLSign
```
You should replace organizationName and commonName with something related to your project and something you will remember lately.

Next switch to the command line and go into your cert folder you created before and run:
```
openssl req -x509 -new -keyout root.key -out root.cer -config root.cnf
```

The script will ask for a PEM pass phrase – enter something – it should be secure and you should remember it.
###Create the server.cnf
```
# OpenSSL configuration for end-entity cert

[ req ]

prompt             = no
string_mask        = default

# The size of the keys in bits:
default_bits       = 2048
distinguished_name = req_distinguished_name

x509_extensions    = x509_ext

[ req_distinguished_name ]

# Note that the following are in 'reverse order' to what you'd expect to see.

countryName = US
organizationName = Ruben Benjamin
commonName = *.rbenjamin.com

[ x509_ext ]

keyUsage=critical,digitalSignature,keyAgreement

subjectAltName = @alt_names

# Multiple Alternate Names are possible
[alt_names]
DNS.1 = *.rbenjamin.com
```
You have to change the following things here. 
* First, change organizationName to the same name you have set in your root.cnf. 
* Next, change DNS.1 to your local domain name. You can also use multiple here by using DNS.2, DNS.3 and so on.
* After saving the file, open your command line again and run the following:

```
openssl req -nodes -new -keyout server.key -out server.csr -config server.cnf
```
###Generate the certificate
Stay in the command line and run the following next:

```
openssl x509 -days 365 -req -in server.csr -CA root.cer -CAkey root.key -set_serial 123 -out server.cer -extfile server.cnf -extensions x509_ext
```
###Copy keys to /etc/ssl with a preferred name of your choice
```
cp server.cer /etc/ssl/rbenjamin.com.cer
cp server.key /etc/ssl/rbenjamin.com.key
```
###Changes in NGINX
```
server {
	listen 80;
	listen [::]:80;
        listen 443 ssl;
	listen [::]:443 ssl;

	ssl_certificate /etc/ssl/rbenjamin.com.cer;
	ssl_certificate_key /etc/ssl/rbenjamin.com.key;
```
###Changes in Apache
```
<VirtualHost *:443>
    ServerName www.testing.rbenjamin.com
    SSLEngine on
    SSLCertificateFile /etc/ssl/rbenjamin.com.cer
    SSLCertificateKeyFile /etc/ssl/rbenjamin.com.key
</VirtualHost>
```
###Adding certs to local machine 
* Copy **root.cer** as **rbenjamin-root.cer** to your Windows / Mac Certificate Store
* Copy **rbenjamin.com.cer** to your Windows / Mac Certificate Store
* Add & trust these certificates so that browsers will detect them as valid


