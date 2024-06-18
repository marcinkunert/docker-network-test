# Testing Docker networking issue with Ubuntu

## Docker Run - default bridge

1. Run nginx with docker run 
    ```shell
    docker run --rm -d -p 8080:80 --name nginx nginx:latest
    ```

```plain
"Networks": {
    "bridge": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "MacAddress": "02:42:ac:11:00:02",
        "NetworkID": "9a7659fc928eb3a07740ad969dcbdfd94134937b7f87185f08d15a3c203a1b42",
        "EndpointID": "ef451b4a5cd93f214aa4394cd8cb3bf68bd41fc1e9dd510b53dd7c4bda22cb61",
        "Gateway": "172.17.0.1",
        "IPAddress": "172.17.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "DriverOpts": null,
        "DNSNames": null
    }
}
```


2. Check if other container can connect
    ```shell
    docker run --rm jonlabelle/network-tools nc -zv host.docker.internal 8080
    ``` 
This works fine on Apple Silicon.

## Docker compose - default bridge

1. Create another bridge network
   ```shell
   docker network create my-net
   ```

2. Run nginx with docker run
    ```shell
    docker run --rm -d -p 8080:80 --name nginx --network my-net nginx:latest
    ```
```json
"Networks": {
    "my-net": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": [
            "7d49e4b1b1d2"
        ],
        "MacAddress": "02:42:ac:17:00:02",
        "NetworkID": "414c8a37f34fcd7f81a78d233d71c596c39aaf061440c6487f2a25e471772637",
        "EndpointID": "71238cb53fc7c1d21dab36fa913495d601b5bdccf27fd9025e424201d4cf1d38",
        "Gateway": "172.23.0.1",
        "IPAddress": "172.23.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "DriverOpts": null,
        "DNSNames": [
            "nginx",
            "7d49e4b1b1d2"
        ]
    }
}

```   

3. Check if other container can connect
    ```shell
    docker run --rm jonlabelle/network-tools nc -zv host.docker.internal 8080
    ``` 
This also works fine on Apple Silicon.

 ```shell
 docker run --rm jonlabelle/network-tools tcptraceroute host.docker.internal
 ``` 

## Nginx from compose:

```
"Gateway": "172.2sock1.0.1",
"IPAddress": "172.21.0.2",
"IPPrefixLen": 16,
```

## network tools using docker run

```shell
"Gateway": "172.17.0.1",
"IPAddress": "172.17.0.2",
"IPPrefixLen": 16,
```

tcptraceroute 

```shell
Selected device eth0, address 172.17.0.2, port 50553 for outgoing packets
Tracing the path to host.docker.internal (192.168.65.254) on TCP port 80 (http), 30 hops max
 1  172.17.0.1  0.208 ms  0.201 ms  0.183 ms
 2  192.168.65.254 [closed]  5.194 ms  2.846 ms  7.346 ms
```