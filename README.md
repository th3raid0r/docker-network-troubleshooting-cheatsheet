# Docker Networking Troubleshooting Cheat Sheet

## I. Outbound Troubleshooting

1. **Can't pull image from Docker Hub or other registries**
   - Check your internet connection.
   - Validate if Docker daemon is running: `systemctl status docker`
   - Confirm your DNS settings: `cat /etc/resolv.conf`
   - Use a different DNS in Docker: `/etc/docker/daemon.json` should look like this:
     ```json
     {
       "dns": ["8.8.8.8", "8.8.4.4"]
     }
     ```
     Then restart Docker: `sudo systemctl restart docker`

2. **Containers can't reach the internet**
   - Check if the container has an IP: `docker inspect -f '{{ .NetworkSettings.IPAddress }}' container_name_or_id`
   - Check network configuration for the container: `docker network inspect bridge`
   - Ping an external IP to verify internet connectivity from within the container: `docker exec -it container_name_or_id ping 8.8.8.8`

3. **Inter-container communication isn't working**
   - Check if containers are in the same network: `docker network inspect network_name`
   - Check if Docker's built-in DNS is resolving container names correctly by pinging the target container.

4. **Network errors when trying to link containers**
   - Check if the `--link` flag was used correctly. It's often better to use user-defined networks over default networks for container communication.

## II. Host Level Issues

1. **IP Forwarding**
   - Make sure that the host machine has IP forwarding enabled: `sysctl net.ipv4.ip_forward` (1 indicates it's enabled)
   - If not enabled, you can enable it with: `sysctl -w net.ipv4.ip_forward=1`

2. **Firewalld Issues**
   - Check status: `sudo firewall-cmd --state`
   - Add docker to trusted zones: `sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0`
   - Reload firewalld: `sudo firewall-cmd --reload`

3. **SELinux Issues**
   - Check SELinux status: `getenforce` or `sestatus`
   - You may have to set SELinux in permissive mode temporarily for troubleshooting: `sudo setenforce 0`
   - Make sure to understand the security implications before disabling or adjusting SELinux settings.

## III. Docker Network Configuration

1. **Check Docker Network**
   - View details: `docker network inspect network_name`
   - Ensure the network isn't private.
   - Check for the correct subnet and gateway.
   - Check if the containers in the network have correct IP addresses assigned.

2. **DNS Issues in Docker**
   - Containers not able to resolve DNS.
   - Check `/etc/resolv.conf` inside the container.
   - If necessary, you can override the DNS settings for a container or for Docker daemon.

3. **Routing Issues**
   - Incorrect or missing route to a network.
   - Check route on host using `ip route` and inside the container using `docker exec -it container_name_or_id ip route`.
   - Check if there's a correct route to the destination.

## IV. Inbound Troubleshooting

1. **Can't access exposed ports on container from outside host**
   - Check if the port was exposed correctly during container creation (`-p` or `-P` flag).
   - Check if the application inside the container is actually listening on the exposed port.
   - Ensure there are no host-level firewall rules blocking inbound connections.
   
2. **Can't access service running on the host from within a container**
   - For user-defined bridge networks or custom networks, you can access the host using the gateway's IP.
   - You can also use `host.docker.internal` as the hostname to refer to the host system from the container.

3. **Swarm Services not accessible from outside**
   - Check if the service's ports were correctly published during service creation (`--publish` flag).
   - Validate if Docker Swarm mode is active: `docker info | grep Swarm`
