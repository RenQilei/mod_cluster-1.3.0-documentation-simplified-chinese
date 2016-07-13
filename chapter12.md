# 第十二章 mod_cluster 中使用 SSL

There are 2 connections between the cluster and the front-end. Both could be encrypted. That chapter describes how to encrypt both connections.

## 12.1. Using SSL between JBossWEB and httpd

As the ClusterListener allows to configure httpd it is adviced to use SSL for that connection. The most easy is to use a virtual host that will only be used to receive information from JBossWEB. Both side need configuration.

### 12.1.1. Apache httpd configuration part

mod_ssl of httpd is using to do that. See in one example how easy the configuration is:

```
Listen 6666
 <VirtualHost 10.33.144.3:6666>
     SSLEngine on
     SSLCipherSuite AES128-SHA:ALL:!ADH:!LOW:!MD5:!SSLV2:!NULL
     SSLCertificateFile conf/server.crt
     SSLCertificateKeyFile conf/server.key
     SSLCACertificateFile conf/server-ca.crt
     SSLVerifyClient require
     SSLVerifyDepth  10 
 </VirtualHost>
 ```
 
The conf/server.crt file is the PEM-encoded Certificate file for the VirtualHost it must be signed by a Certificate Authority (CA) whose certificate is stored in the sslTrustStore of the ClusterListener parameter.

The conf/server.key file is the file containing the private key.

The conf/server-ca.crt file is the file containing the certicate of the CA that have signed the client certificate JBossWEB is using. That is the CA that have signed the certificate corresponding to the sslKeyAlias stored in the sslKeyStore of the ClusterListener parameters.

### 12.1.2. ClusterListener configuration part

There is a wiki describing the SSL parameters of the ClusterListener. See in one example how easy the configuration is:

```
<Listener className="org.jboss.web.cluster.ClusterListener"
           ssl="true"
           sslKeyStorePass="changeit"
           sslKeyStore="/home/jfclere/CERTS/CA/test.p12"
           sslKeyStoreType="PKCS12"
           sslTrustStore="/home/jfclere/CERTS/CA/ca.p12"
           sslTrustStoreType="PKCS12" sslTrustStorePassword="changeit"/>
```

The sslKeyStore file contains the private key and the signed certificate of the client certificate JBossWEB uses to connect to httpd. The certificate must be signed by a Cerficate Authority (CA) who certificate is in the conf/server-ca.crt file of the httpd

The sslTrustStore file contains the CA certificate of the CA that signed the certificate contained in conf/server.crt file.

### 12.1.3. mod-cluster-jboss-beans configuration part

The mod-cluster-jboss-beans.xml in $JBOSS_HOME/server/profile/deploy/mod-cluster.sar/META-INF in the ClusterConfig you are using you should have something like:

```
<property name="ssl">true</property>
<property name="sslKeyStorePass">changeit</property>
<property name="sslKeyStore">/home/jfclere/CERTS/test.p12</property>
<property name="sslKeyStoreType">pkcs12</property>
<property name="sslTrustStore">/home/jfclere/CERTS/ca.p12</property>
<property name="sslTrustStoreType">pkcs12</property>
<property name="sslTrustStorePassword">changeit</property>
```

### 12.1.4. How the diferent files were created

The files were created using OpenSSL utilities see OpenSSL CA.pl (/etc/pki/tls/misc/CA for example) has been used to create the test Certificate authority, the certicate requests and private keys as well as signing the certicate requests.

#### 12.1.4.1. Create the CA

1. Create a work directory and work for there:
```
mkdir -p CERTS/Server cd CERTS/Server
```
2. Create a new CA:
```
/etc/pki/tls/misc/CA -newca
```
That creates a directory for example ../../CA that contains a cacert.pem file which content have to be added to the conf/server-ca.crt described above.
3. Export the CA certificate to a .p12 file:
```
openssl pkcs12 -export -nokeys -in ../../CA/cacert.pem -out ca.p12
```

That reads the file cacert.pem that was created in the previous step and convert it into a pkcs12 file the JVM is able to read.

That is the ca.p12 file used in the sslTrustStore parameter above.

#### 12.1.4.2. Create the server certificate

1. Create a new request:
```
/etc/pki/tls/misc/CA -newreq
```
That creates 2 files named newreq.pem and newkey.pem. newkey.pem is the file conf/server.key described above.
2. Sign the request:
```
/etc/pki/tls/misc/CA -signreq
```
That creates a file named newcert.pem. newcert.pem is the file conf/server.crt described above. At that point you have created the SSL stuff needed for the VirtualHost in httpd. You should use a browser to test it after importing in the browser the content of the cacert.pem file.

#### 12.1.4.3. Create the client certificate

1. Create a work directory and work for there:
```
mkdir -p CERTS/Client cd CERTS/Client
```
2. Create request and key for the JBossWEB part.
```
/etc/pki/tls/misc/CA -newreq
```
That creates 2 files: Request is in newreq.pem, private key is in newkey.pem
3. Sign the request.
```
/etc/pki/tls/misc/CA -signreq
```
That creates a file: newcert.pem
4. Don’t use a passphrase when creating the client certicate or remove it before exporting:
```
openssl rsa -in newkey.pem -out key.txt.pem mv key.txt.pem newkey.pem
```
5. Export the client certificate and key into a p12 file.
```
openssl pkcs12 -export -inkey newkey.pem -in newcert.pem -out test.p12
```
That is the sslKeyStore file described above (/home/jfclere/CERTS/CA/test.p12)

## 12.2. Using SSL between httpd and JBossWEB

Using https allows to encrypt communications betwen httpd and JBossWEB. But due to the ressources it needs that no advised to use it in high load configuration.

(See Encrypting connection between httpd and TC [http://www.jboss.org/community/docs/DOC-
9701] for detailed instructions).

httpd is configured to be a client for AS/TC so it should provide a certificate AS/TC will accept and have a private key to encrypt the data, it also needs a CA certificate to valid the certificate AS/TC will use for the connection.

```
SSLProxyEngine On
SSLProxyVerify require
SSLProxyCACertificateFile conf/cacert.pem
SSLProxyMachineCertificateFile conf/proxy.pem
```

conf/proxy.pem should contain both key and certificate. The certificate must be trusted by Tomcat via the CA in truststoreFile of <connector/>.

conf/cacert.pem must contain the certificat of the CA that signed the AS/TC certificate. The correspond key and certificate are the pair specificed by keyAlias and truststoreFile of the <connector/>. Of course the <connector/> must be the https one (normally on port 8443).

### 12.2.1. How the diferent files were created

The files were created using OpenSSL utilities see OpenSSL [http://www.openssl.org/] CA.pl (/etc/pki/tls/misc/CA for example) has been used to create the test Certificate authority, the certicate
requests and private keys as well as signing the certicate requests.

#### 12.2.1.1. Create the CA

(See above)

#### 12.2.1.2. Create the server certificate

(See above)

The certificate and key need to be imported into the java keystore using keytool

make sure you don't use a passphare for the key (don't forget to clean the file when done)

1. Convert the key and certificate to p12 file:
```
openssl pkcs12 -export -inkey key.pem -in newcert.pem -out test.p12
```
make sure you use the keystore password as Export passphrase.
2. Import the contents of the p12 file in the keystore:
```
keytool -importkeystore -srckeystore test.p12 -srcstoretype PKCS12
```
3. Import the CA certificate in the java trustore: (Fedora13 example).
```
keytool -import -trustcacerts -alias "caname" \
-file ../../CA/cacert.pem -keystore /etc/pki/java/cacerts
```
4. Edit server.xml to have a <connector/> similar to:
```
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true" keyAlias="1" truststoreFile="/etc/pki/java/cacerts" maxThreads="150" scheme="https" secure="true" clientAuth="true" sslProtocol="TLS" />
```
5. Start TC/AS and use openssl s_client to test the connection:
```
openssl s_client -CAfile /home/jfclere/CA/cacert.pem -cert newcert.pem -key newkey.pem \
-host localhost -port 8443
```
There shouldn't be any error and you should be able to see your CA in the "Acceptable client certificate CA names".

## 12.3. Forwarding SSL browser informations when using http/https between httpd and JBossWEB

When using http or https beween httpd and JBossWEB you need to use the SSLValve and export the SSL variable as header in the request in httpd. If you are using AJP, mod_proxy_ajp will read the SSL variables and forward them to JBossWEB automaticaly.

(See Forwarding SSL environment when using http/https proxy [http://www.jboss.org/community/docs/DOC-11988] for detailed instructions).

The SSL variable used by mod_proxy_ajp are the following:

1. "HTTPS" SSL indicateur.
2. "SSL_CLIENT_CERT" Chain of client certificates.
3. "SSL_CIPHER" Cipher used.
4. "SSL_SESSION_ID" Id of the session.
5. "SSL_CIPHER_USEKEYSIZE" Size of the key used.

