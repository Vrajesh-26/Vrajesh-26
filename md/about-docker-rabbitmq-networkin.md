# About Doker network understanding using RabbitMQ

> Java app → OS picks ephemeral port 50278 → connects to 127.0.0.1:5672 → Docker's host-side proxy (PID 31636) accepts it → forwards into container → container's internal RabbitMQ (different PID, different namespace, invisible to your netstat) handles it.

NETSTAT COLUMNS:
  Local Address | Foreign Address | State | PID
 
  - netstat lists SOCKETS, not connections.
  - One TCP connection = 2 sockets = 2 rows (if both
    processes are on same machine, e.g. 127.0.0.1).
 
EXAMPLE:
  127.0.0.1:5672   127.0.0.1:50278  ESTABLISHED  31636
  127.0.0.1:50278  127.0.0.1:5672   ESTABLISHED  22988
 
  Row 1 = server-side socket (listening on 5672)
  Row 2 = client-side socket (your Java app)

---

EPHEMERAL PORT (e.g. 50278):
  - Random free port OS assigns to an OUTGOING
    connection's source side.
  - Range approx 49152-65535.
  - Not Docker, not the target service. No fixed
    meaning. Ignore it.

---

CONNECTION FLOW (Java -> RabbitMQ in Docker):
  Java app
    -> OS assigns ephemeral port (e.g. 50278) \
    -> connects to 127.0.0.1:5672 (Windows host) \
    -> docker-proxy accepts (host-side PID) \
    -> forwards into container network \
    -> RabbitMQ process inside container handles it 

---

DOCKER PORT PUBLISHING (Docker Desktop / Win-Mac):
  - Containers run inside a hidden VM.
  - "-p 5672:5672" creates a host-side FORWARDING
    process (docker-proxy) that binds 127.0.0.1:5672
    on Windows and relays into the container.
  - So the PID owning host port 5672 is usually
    docker-proxy, NOT RabbitMQ itself.
  - Real RabbitMQ process lives inside the container,
    invisible to Windows netstat.
 
VERIFY WHO OWNS A PORT:
  - netstat -ano | findstr :5672
  - tasklist /FI "PID eq <pid>" \
  -> if result = docker-proxy.exe / com.docker.backend.exe
     => confirmed Docker forwarding, not the app directly.

---

## What does do netstat ?

netstat on a given machine only ever shows sockets owned by that machine. <br> 
Each row's "Local Address" is always local to the machine you ran netstat on.

The "Foreign Address" column is just the remote endpoint that local socket is talking to \
it doesn't mean that endpoint's socket is also listed in the same netstat output. \
It's listed separately, in its own right, only if a process on this same machine \
also owns that address.

In your case:

Java's socket (127.0.0.1:50278) is local → shown
Docker-proxy's socket (127.0.0.1:5672) is also local (both processes run on your Windows host, container networking aside) → also shown

*linked as one connection" in netstat's eyes, but because both endpoints of that connection happen to physically reside on the machine you queried.*

### If you ran netstat on a remote machine where only the RabbitMQ side lived, you'd see just:

```
<remote-ip>:5672   <your-ip>:50278   ESTABLISHED
```

> one row, no matching "Java" row - because Java's socket isn't local to that machine.\
Your read is correct: destination shown = wherever that socket actually lives, and in this case that's the same box.
