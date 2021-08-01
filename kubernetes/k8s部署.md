# K8s部署

## kubespray集群搭建

```
Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-v2izm4wt/cryptography/setup.py", line 14, in <module>
        from setuptools_rust import RustExtension
    ModuleNotFoundError: No module named 'setuptools_rust'
```

解决方式：

```bash
python3.6 -m pip install --upgrade pip
```



```
fatal: [izbp10mvd8x4dp62yo51viz]: FAILED! => {
    "assertion": "ip in ansible_all_ipv4_addresses",
    "changed": false,
    "evaluated_to": false,
    "msg": "'['172.17.123.253']' do not contain '47.98.39.233'"
}
```

ipv4出问题，解决方式：之前使用的是服务器的公网IP，而这里需要使用的是网卡的IP。

```
fatal: [VM-0-5-centos]: FAILED! => {
    "assertion": "inventory_hostname is match(\"[a-z0-9]([-a-z0-9]*[a-z0-9])?(\\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*$\")",
    "changed": false,
    "evaluated_to": false,
    "msg": "Hostname must consist of lower case alphanumeric characters, '.' or '-', and must start and end with an alphanumeric character"
}
```

hostname不能包含特殊字符；

```
FAILED - RETRYING: ensure containerd packages are installed (4 retries left).Result was: {
...
  "Failure talking to yum: failure: repodata/repomd.xml from docker-ce: [Errno 256] No more mirrors to try.\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable\"\nhttps://download.docker.com/linux/centos/7/x86_64/stable/repodata/repomd.xml: [Errno 14] curl#7 - \"Failed to connect to 2600:9000:21b5:ca00:3:db06:4200:93a1: Network is unreachable
}
```

看错误是docker无法安装，解决方式：下载好镜像，确保443端口已经开放。

```
ss://Y2hhY2hhMjAtaWV0Zi1wb2x5MTMwNTo2NDQyMzUxNy0yZjJjLTQxNzYtOTdkYy1iZmUxMzU5NzYwZDQ@ngzyd-1.okex-tradebot.xyz:30001#HK-HKBN%20%7C%20SS%20%7C%20%E5%B9%BF%E7%A7%BB%E4%B8%AD%E8%BD%AC
```



```
chacha20-ietf-poly1305:64423517-2f2c-4176-97dc-bfe1359760d4
```

```
server: ngzyd-1.okex-tradebot.xyz
server_port: 30001
password: 64423517-2f2c-4176-97dc-bfe1359760d4
method: chacha20-ietf-poly1305
```



代理安装前提：gcc，gcc c++，libsodium解密工具；