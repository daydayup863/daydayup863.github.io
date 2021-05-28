---
title: OpenSSL的简单使用 
date: 2021-04-16 19:18:18
tags: 
- openssl
- ssl
categories: 
- openssl
- ssl
top: 41 
description: 
password: 

---


OpenSSL 不仅仅是 SSL。它可以实现消息摘要、文件的加密和解密、数字证书、数字签名 和随机数字。关于 OpenSSL 库的内容非常多，远不是一篇文章可以容纳的。
OpenSSL 不只是 API，它还是一个命令行工具。命令行工具可以完成与 API 同样的工作， 而且更进一步，可以测试 SSL 服务器和客户机。它还让开发人员对 OpenSSL 的能力有一个 认识。要获得关于如何使用 OpenSSL 命令行工具的资料, 请参阅[官方手册](https://www.openssl.org/docs/apps/openssl.html)

<!--more-->

# 编译安装

```
$ ./config
$ make
$ make test
$ make install
```

# 产生私钥
```
jintao@jintao-ThinkPad-L490:~/personal/code$ openssl genrsa -out rsa_private_key.pem 1024
Generating RSA private key, 1024 bit long modulus (2 primes)
.......+++++
..............+++++
e is 65537 (0x010001)
jintao@jintao-ThinkPad-L490:~/personal/code$ cat rsa_private_key.pem
-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQC/DCmUwW3FwCLB4Wtm4jw+b5RCIRJwl/QustuIyHmWsTXApp9v
pnRfZtojgh9FUjIsxWmg3Gc5gLn8r4k+pI8Fk77lff2ewSOM2ZVHZkmrFtg7vSmC
tVY+uPLjdw3aONYUXj3+cZgbpemLY0v07EkrwGS8jzztABlFauWreEvX1wIDAQAB
AoGAa/+8IdOW8otDGscLqAWMeN8quJdpjSzZZOzHHfP2iDF1aYrH5p36e1PxTNFq
TA3DP3v50m3GDMOwYB/7PeZY0pvvrKPSunq0/tbcVVjr2pKs3X1rRtZQ2X8EaH8f
IzKmQM+NT0Av0gLK6txJ4vONzbzuiGTqFtF/mRAJiHOMoIECQQDom1CFf49zVvtu
g88GwXlRGrOlfGHQ0vBj+h8dYc0AbWlrPjbkcOoThLvkgUhXCquaof/Ku4qDG/VH
0YgAzuAfAkEA0kLdv/IafPTNCc83U/hjUCoZhHYYiM5ghRRa2jMjmKQDogcHhvfb
7PaLOvxuWxWx/adIKLXRvcnGU9E/LpgxSQJBAIvD/18n5bdFVbDzLGt/x3ivVbCj
C1dh2CYKvbV29apDE+vnpy4eltgBkrDb6e67L5+rpbpYdAMRwpFT2qe5prsCQHRd
STgnlv08xhT9t1MjjmMZSZIDgcSE4uoDv9wunS6m5tPPLB1II1DbiWaVucVzFlSZ
NdhB99gfSUGt9lelJvECQQDRS7+I5nX0/ZhQDwb29ZMw1EhzczCQr0fIjDYHsCZ4
HJqOoycG9EvOQW99NDlYV5vUAhKu3QfdfCDsfjm0bCr5
-----END RSA PRIVATE KEY-----
jintao@jintao-ThinkPad-L490:~/personal/code$ openssl genrsa -out rsa_private_key.pem 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
..+++++
.+++++
e is 65537 (0x010001)
jintao@jintao-ThinkPad-L490:~/personal/code$ cat rsa_private_key.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAuaUSgbK540vy3TMpgREW2+apXbfYO3TvMmxWiupN9WypGOzA
1yFo/udaYWKvoPhEau2S9Z+oS5CV8e8KhEoJc6TJklipAj+SAtAKpBL0138Uwsv1
cRiGLKLWHz1hfqUZE6nReLz1rkTvnicestJvVvG7P1Ey4d9GF73VMWM+TGYXv+Mc
fsi2XeJGD9mz7bitsnKMsehqd0yy6rmOthu/3CIj1xycOPs7dESt3gYUv+Ve7vGD
ZNKzieNlfO3Kv64KWsoXGNvkDMMklxQVLY8ceTtsqTBtqr5Cz0M27YkG39WtH5xJ
fgqUBu48mr7xZ5RoiXweB43xxvDweZAPZkW7JwIDAQABAoIBAF5Jzvp80hj1zCY5
BA1/poKNxtvIslSQcaTEjbgEhQ/v3nrAVLlvSKqeyrLHsrgpIGbGv2ttacbnaqD0
bfz+tYo82fwwd7ainwA6tgzba+u/qcW+XazRbeRh2CsJu+jc5b2s9K1EG+wlKybC
a78bTl10bUwda6B5DFqmmg95fnzCSDgWOSGNs3k4Z3B45a0Rl73pQYTdGHb+SuX8
sYmuGMNaWz0nWPfLbzg2cxIeUTSJSmMhSYX8yChXKouUXwYmuBlAQeKxrsrTiQZI
EgW7hKykcebGvCBV/H9AVtoeBx0i31+TVrt672hVO6ZeGU6/rydzIJ9BExJI3uTr
OrGJUeECgYEA3WXyVQ+MiCLduV2pY234ubsBQcvmxLs8Qx+2K9BZQAGiK6Vm6hVj
jKIcRPopisxqU215oDsEm/lsH+qLqMLU7thng7yfw5EzGhhg23NW+4rrr8Iz1F0v
0jLvjfh1Ff1E5ZVZpiAoXjv5ASiCPmoyapmo3I4JIpjdeFeYRhW+X9kCgYEA1qil
Yh20dW0OS+6Hc5HkAHXfpXNQj7j3dRj8pxkDzqyfY9r/b3sQ4k6B2GOBNOnoyYuP
sYQryB+08jY9MkH17l0kk/vPNkZatpqQRo6xMiw2+/09RcV0/e3zqT7/Cv6Tc7Sh
veD0cjfjIL8WM2/gqYM2PMPQvzfznqpzNpi/Ev8CgYEAvcxi5gbxa9ewCvQ/fYzO
WLL3Tee2SttUuxqZepAfox6DXzVpt61kbTCgWYW4TVQWprTIOtO9jNVTmzzgQ2nb
T3LXsvjmYaq9i1Zw2lDTtcsPZ9ptwlWs5F9kPGpOPe6kvMi/VQpmcPqq6hJHLaiu
1fIq8AEX1cAExOEbGqITVWkCgYABnjfQ64RmtjG7ZMrklh7v2fObnajnzG8hFNUi
tU+QCUESUZ5HStgvvIPCC833hiPZERI+Nk7WLVcB1GLVtCWUbGNQMj+3mwQoCDY6
Me0oAalQcPI7Sme9WkPR7MWjYZPe9WeatM1i5wTxRD94l8lLvc902c0DA/r0ITjJ
GpGmJQKBgQCTM38D80Z2JnzsG/wP6N0LBeiBRbPOU4FD+GUHiSKE7EKA+MWZTs7C
G5IAXyG7eHsGqRNKu8VBNtpEpWMfcpWDdalWx2NG4uqk/M3/7NMsPIrJ6VryD809
1pBKr1HyRafzDNiJvZ/SIoyrKZs0dkz/jaOvx6dmrpegWMExKktEEA==
-----END RSA PRIVATE KEY-----
```
# 产生公钥
```
jintao@jintao-ThinkPad-L490:~/personal/code$ openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
writing RSA key
jintao@jintao-ThinkPad-L490:~/personal/code$ cat rsa_public_key.pem
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuaUSgbK540vy3TMpgREW
2+apXbfYO3TvMmxWiupN9WypGOzA1yFo/udaYWKvoPhEau2S9Z+oS5CV8e8KhEoJ
c6TJklipAj+SAtAKpBL0138Uwsv1cRiGLKLWHz1hfqUZE6nReLz1rkTvnicestJv
VvG7P1Ey4d9GF73VMWM+TGYXv+Mcfsi2XeJGD9mz7bitsnKMsehqd0yy6rmOthu/
3CIj1xycOPs7dESt3gYUv+Ve7vGDZNKzieNlfO3Kv64KWsoXGNvkDMMklxQVLY8c
eTtsqTBtqr5Cz0M27YkG39WtH5xJfgqUBu48mr7xZ5RoiXweB43xxvDweZAPZkW7
JwIDAQAB
-----END PUBLIC KEY-----
```

# 生成证书
```
jintao@jintao-ThinkPad-L490:~/personal/code$ openssl req -new -key rsa_private_key.pem -out zongbao.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
jintao@jintao-ThinkPad-L490:~/personal/code$ cat zongbao.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICijCCAXICAQAwRTELMAkGA1UEBhMCQVUxEzARBgNVBAgMClNvbWUtU3RhdGUx
ITAfBgNVBAoMGEludGVybmV0IFdpZGdpdHMgUHR5IEx0ZDCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBALmlEoGyueNL8t0zKYERFtvmqV232Dt07zJsVorq
TfVsqRjswNchaP7nWmFir6D4RGrtkvWfqEuQlfHvCoRKCXOkyZJYqQI/kgLQCqQS
9Nd/FMLL9XEYhiyi1h89YX6lGROp0Xi89a5E754nHrLSb1bxuz9RMuHfRhe91TFj
PkxmF7/jHH7Itl3iRg/Zs+24rbJyjLHoandMsuq5jrYbv9wiI9ccnDj7O3RErd4G
FL/lXu7xg2TSs4njZXztyr+uClrKFxjb5AzDJJcUFS2PHHk7bKkwbaq+Qs9DNu2J
Bt/VrR+cSX4KlAbuPJq+8WeUaIl8HgeN8cbw8HmQD2ZFuycCAwEAAaAAMA0GCSqG
SIb3DQEBCwUAA4IBAQCJkU68WPfMrMKiNJlXvPbPMLSyF6Q7zx8mL/JGeKctqRzm
4DUZHUiPjLiXBPXtXM87HMApEbA8UyU8g7Bx1GWnFc2ZDihaXgdAPs9CEaEBvu0x
naWT1BviOMy4CbjybrQjE5QvRHGcKt2b28cmfpAKiXYHKEFw7DH/yDkWbQPDyuPY
5JHC35olabvVc7H/+V8fQUssorj+9NKrUKhahH4oITfRvizkdPiF1acWF+XtbtHm
YK7rTHYCr/n8lLAY9etUtAHaTPQMDH5T7ZjngljT2kvMHdUAf+jIJHeIAoLZd9PO
zYYzuhFuPXKb0seQ1u9LlL+FjdwUVJS3O3r9pFdR
-----END CERTIFICATE REQUEST-----
```
