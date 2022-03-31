# Cipher-TLS-removing-vulnerabilities-from-openvas
Cipher TLS removing vulnerabilities from openvas



##CVE-2016-2183 SWEET32 Birthday attacks


```bash

nmap --script ssl-enum-ciphers -p 443 SERVER_IP_ADDRESS_HERE
Starting Nmap 5.51 ( http://nmap.org ) at 2016-11-30 17:57 EST
Nmap scan report for xxxxxxxx (xxx.xxx.xxx.xxx)
Host is up (0.018s latency).
PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers: 
|   TLSv1.2
|     Ciphers (14)
|       TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
|       TLS_RSA_WITH_3DES_EDE_CBC_SHA
|       TLS_RSA_WITH_AES_128_CBC_SHA
|       TLS_RSA_WITH_AES_128_CBC_SHA256
|       TLS_RSA_WITH_AES_128_GCM_SHA256
|       TLS_RSA_WITH_AES_256_CBC_SHA
|       TLS_RSA_WITH_AES_256_CBC_SHA256
|       TLS_RSA_WITH_AES_256_GCM_SHA384
|     Compressors (1)
|_      uncompressed


```
As you can see in the output above, this server is affected by this vulnerability as its allowing for the following 3DES ciphers:

```bash
TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA
TLS_RSA_WITH_3DES_EDE_CBC_SHA
```
Disabling this in Apache is pretty easy. Simply navigate to where ever you have your SSLCipherSuite configuration defined and disable 3DES. Typically this should be in /etc/httpd/conf.d/ssl.conf, however some may also have this defined in each individual Apache vhost. If you are unsure where its configured, you should be able to locate it on your server by running:

```bash
# CentOS / Red Hat
[root@web01 ~]# egrep -R SSLCipherSuite /etc/httpd/*

# Ubuntu / Debian
[root@web01 ~]# egrep -R SSLCipherSuite /etc/apache2/*
```
Once you locate the config(s) that contain this directive, you simple add !3DES to the end of the SSLCipherSuite line as shown below:

```bash
[root@web01 ~]# vim /etc/httpd/conf.d/ssl.conf
...
SSLCipherSuite EECDH+AESGCM:EECDH+AES256:EECDH+AES128:EDH+AES:RSA+AESGCM:RSA+AES:!ECDSA:!NULL:!MD5:!DSS:!3DES
...
```
Once that is done, restart Apache by:


```bash
# CentOS / Red Hat
[root@web01 ~]# service httpd restart

# Ubuntu / Debian
[root@web01 ~]# service apache2 restart
```

##Finally, retest using nmap to confirm no ciphers using 3DES show up:

```bash
[user@workstation ~]# nmap --script ssl-enum-ciphers -p 443 SERVER_IP_ADDRESS_HERE
Starting Nmap 5.51 ( http://nmap.org ) at 2016-11-30 18:03 EST
Nmap scan report for xxxxxxxx (xxx.xxx.xxx.xxx)
Host is up (0.017s latency).
PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers: 
|   TLSv1.2
|     Ciphers (12)
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
|       TLS_RSA_WITH_AES_128_CBC_SHA
|       TLS_RSA_WITH_AES_128_CBC_SHA256
|       TLS_RSA_WITH_AES_128_GCM_SHA256
|       TLS_RSA_WITH_AES_256_CBC_SHA
|       TLS_RSA_WITH_AES_256_CBC_SHA256
|       TLS_RSA_WITH_AES_256_GCM_SHA384
|     Compressors (1)
|_      uncompressed
```
If no 3DES ciphers are returned like in the listing above, you should be good to rerun your vulnerability scan!

## implement cipher on kubernetes

vim /etc/kubernetes/manifests/kube-apiserver.yaml
```bash
- --tls-cipher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- --tls-min-version=VersionTLS12
```

## If you need to edit e deployment 



```bash
kubectl edit deployment metrics-server  -n kube-system
```
Paste the line bellow, afte the line " - --kubelet-insecure-tls"
```bash
- --tls-cipher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- --tls-min-version=VersionTLS12
```

```bash
kubectl logs -f metrics-server-7b87557888-ckhr6 -n kube-system
```


## Implement Cipher on SSH, (Weak Key Exchange)

edit the sshd configuration file:
```bash
vim /etc/ssh/sshd_config
```

```bash
KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
```
Restart the ssh service

```bash
systemctl restart sshd
```
