# Day 07: Docker Networking

## Objectives

By the end of this session, you should be able to:

- understand how Docker networking works internally
- explain the role of namespaces, bridges, veth pairs, and NAT
- create and use Docker networks
- publish container ports to the host

## 1. Docker Networking Internals

Docker networking allows containers to communicate with each other, with the host machine, and with external systems. Internally, Docker uses Linux networking features to create isolated network spaces for each container.

When a container starts, Docker creates a separate network namespace for it. Inside that namespace, the container gets its own:

- network interfaces
- IP address
- routing table
- ports

Docker then connects this namespace to the host using virtual network interfaces called `veth` pairs.

## 2. How it works internally

1. Docker creates a new network namespace for the container.
2. A virtual Ethernet pair is created.
3. One end of the pair is attached to the container as `eth0`.
4. The other end is attached to a Docker-managed bridge such as `docker0`.
5. Docker configures IP addressing and routing.
6. NAT and port publishing rules are added so traffic can reach the container from outside.

This is why containers can have their own IP addresses while still being reachable from the host.

## 3. Default bridge network

By default, Docker uses a bridge network for containers on a single host.

```bash
docker network ls
docker network inspect bridge
docker info | findstr "bridge"
```

The `bridge` network is useful for communication between containers on the same Docker host.

## 4. Create your own network

```bash
docker network create day07-net
```

## 5. Connect containers to the same network

```bash
docker run -d --name web1 --network day07-net nginx
docker run -d --name web2 --network day07-net nginx
```

Containers on the same network can reach each other by container name.

```bash
docker exec web1 ping web2
docker exec web1 ping 172.18.0.3
```

## 6. Container DNS and name resolution

Docker provides built-in DNS for containers on the same user-defined network.

Example:

```bash
docker exec web1 nslookup web2
docker exec web2 nslookup web1
```

This makes service discovery easier than using raw IP addresses.

## 7. Port publishing

If you want to access a container from the host or outside, you publish a host port to a container port.

```bash
docker run -d --name web-app -p 8080:80 nginx
```

This means:

- host port `8080`
- container port `80`

You can access the app by visiting `http://localhost:8080`.

## 8. Common network modes

| Mode | Description |
| --- | --- |
| `bridge` | Default network for containers on one host |
| `host` | Container shares the host network namespace |
| `none` | Container has no network |
| `overlay` | Used for multi-host networking in Swarm |
| `macvlan` | Container appears like a separate device on the network |

## 9. Basic networking commands

```bash
docker ps
docker inspect web1
docker network inspect day07-net
docker logs web1
```

## 10. Key networking concepts

- `namespace`: gives each container its own network stack
- `veth pair`: connects container network namespace to the Docker bridge
- `bridge`: virtual switch used for container-to-container communication
- `iptables/NAT`: handles routing and port publishing
- `DNS`: enables container-to-container name resolution

## 11. Key Takeaways

- Docker networking uses namespaces, bridges, veth pairs, and NAT to connect containers securely and efficiently.
- User-defined networks make container communication easier than using raw IPs.
- Port publishing exposes services from the container to the host and outside world.
