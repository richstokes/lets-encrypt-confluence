# lets-encrypt-confluence
Generate and install a free lets-encrypt SSL Certificate on your Confluence site.

At some point I may BASH-Scriptorize this. The following guide assumes default install locations for Confluence. Adjust paths and domain name as needed.


## Pre-requisits
###Install java keytool or use the one under '/opt/atlassian/confluence/jre/bin'
sudo apt install openjdk-9-jre-headless -Y
sudo mpdir -p /var/atlassian/keystores
**cd /var/atlassian/keystores**


## 1. Create a new Java Keystore
keytool -genkeypair -alias simple-cert -keyalg RSA -keysize 2048 -keystore letsencrypt.jks -dname "CN=appsbyrich.com" -storepass hunter22


## 2. Create CSR
keytool -certreq -alias simple-cert -keystore letsencrypt.jks -file jks-appsbyrich.com.csr -storepass password123 -ext san=dns:www.appsbyrich.com


## 3. Install CertBot (formally lets-encrypt-auto)
git clone https://github.com/certbot/certbot.git


## 4. Request public certificate
./certbot-auto certonly --manual --csr /var/atlassian/keystores/jks-appsbyrich.com.csr --preferred-challenges "dns"

## 5. Verify DNS ownership
When prompted by CLI tool, add verification records to DNS.

## 6. Review results
On success you should get something like:
```
Server issued certificate; certificate written to /var/atlassian/keystores/certbot/0000_cert.pem

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /var/atlassian/keystores/certbot/0001_chain.pem. Your cert will
   expire on 2017-09-21. To obtain a new or tweaked version of this
   certificate in the future, simply run certbot-auto again. To
   non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
```

## 7. Move new certificate up into your keystore directory
mv *.pem /var/atlassian/keystores
cd /var/atlassian/keystores

## 8. Import new public certificate into your Java Keystore
keytool -importcert -alias simple-cert -keystore letsencrypt.jks -storepass password123 -file 0001_chain.pem
Answer the messaage: "..is not trusted. Install reply anyway? [no]:"  yes
You should see "Certificate reply was installed in keystore"


## 9. Edit the confluence server.xml file
/opt/atlassian/confluence/conf/server.xml

Uncomment SSL connector section
set password
set keystoreFile="/var/atlassian/keystoresletsencrypt.jks"

Here's a working example:
```
<Connector port="8443" maxHttpHeaderSize="8192"
                   maxThreads="150" minSpareThreads="25"
                   protocol="org.apache.coyote.http11.Http11NioProtocol"
                   enableLookups="false" disableUploadTimeout="true"
                   acceptCount="100" scheme="https" secure="true"
                   clientAuth="false" sslProtocols="TLSv1,TLSv1.1,TLSv1.2" sslEnabledProtocols="TLSv1,TLSv1.1,TLSv1.2" SSLEnabled="true"
                   URIEncoding="UTF-8" keystorePass="password123"
                   keystoreFile="/var/atlassian/keystores/letsencrypt.jks"
/>
```

##10. Restart Confluence
service confluence stop
service confluence start
service confluence status  - Review/fix any errors


##11. Change Base URL
As confluence admin, change base URL in settings to be https://appsbyrich.com:8443

