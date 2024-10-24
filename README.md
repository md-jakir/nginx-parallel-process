# nginx-parallel-process
Parallel process by Nginx and System 
# 1. Nginx Configuration for Parallel Processing
Nginx uses a multi-threaded and multi-process architecture. To enable and optimize parallel processing in Nginx, you can tune the following directives in your Nginx configuration (`nginx.conf`):
## a. Worker Processes (`worker_processes`)

  - This controls the number of worker processes that Nginx will spawn. Each worker process can handle multiple connections, so increasing the number of worker processes will improve Nginx's ability to handle parallel requests.
  - *Recommended Setting:* Set `worker_processes` to the number of CPU cores available on your server to fully utilize parallelism.
  - Example:

        worker_processes auto;
This will automatically set the number of worker processes to match the number of CPU cores (auto-detect).

## b. Worker Connections (`worker_connections`)
- This sets the maximum number of simultaneous connections each worker process can handle. A higher value allows each worker to process more parallel connections.
- *Recommended Setting:* A good starting point is 1024, but you may increase it depending on your traffic and server capacity.
- Example:

      events {
          worker_connections 1024;
      }
## c. Multi-threading with `worker_threads` (optional)
- Nginx workers are generally single-threaded, but if you have high-latency operations or I/O-bound processes, you can enable multi-threading using the threads directive (available in Nginx 1.7.11+).
- This allows Nginx worker processes to utilize threads to handle I/O operations in parallel.
- Example:

      events {
          worker_connections 1024;
          worker_threads 8;
      }
## d. Enable `accept_mutex` for Load Balancing Among Workers
- The accept_mutex directive ensures that only one worker process accepts new connections at a time, avoiding overloading a single worker with too many requests. However, you can disable it if you want workers to accept connections in parallel.
- *Recommended Setting:* By default, accept_mutex is on, and it's usually best to keep it that way for efficient load distribution.
- Example:

      accept_mutex on;
## e. Tuning `keepalive_requests`
- The `keepalive_requests` directive controls how many requests a single connection can handle before it's closed. This is important for handling parallel connections efficiently, especially if you're using long-lived keep-alive connections.
- Example:

      keepalive_requests 100;
## f. Tuning `keepalive_timeout`
- This controls how long a connection will remain open (idle) before being closed. Shorter keep-alive times free up resources for new connections faster.
- Example:

      keepalive_timeout 65;
# 2. Server-Level Configuration for Parallel Processing

At the operating system and server level, you can also make some changes to ensure optimal parallel processing:

## a. Increase File Descriptors Limit
- Nginx uses file descriptors to manage connections, so you should increase the file descriptor limit to handle a large number of concurrent connections.
- You can configure this in the system's `/etc/security/limits.conf` file.
- Example:
      
      * soft nofile 65536
      * hard nofile 65536
- Additionally, you may set it in your Nginx configuration:

      worker_rlimit_nofile 65536;
## b. Tuning Kernel Parameters (Linux)
- You can optimize Linux kernel parameters for better parallelism and performance under high load. Modify ``/etc/sysctl.conf`` to include:

      net.core.somaxconn = 65535      # Increase maximum connection queue length
      net.core.netdev_max_backlog = 4096  # Set the max number of packets in the backlog queue
      net.ipv4.tcp_max_syn_backlog = 4096 # Increase SYN backlog queue length
      fs.file-max = 100000            # Maximum number of file descriptors
- After editing the file, apply the changes:

      sudo sysctl -p
## c. Enabling Asynchronous I/O (aio)
    location /files/ {
        aio on;
        sendfile on;
    }
## d. Sendfile Directive
- The sendfile directive allows Nginx to use efficient I/O operations by sending files directly from disk to the network, bypassing user space. This reduces CPU usage and improves parallel processing.
- Example:

      sendfile on;
## e. TCP Fast Open
- TCP Fast Open allows data to be sent during the TCP handshake, reducing latency for new connections.
- Enable it at the OS level (Linux):

      echo 3 > /proc/sys/net/ipv4/tcp_fastopen
- Then configure it in Nginx:

      listen 443 ssl fastopen=3;
# 3. Load Balancing (Optional)

If you are running multiple Nginx instances or servers, you can set up load balancing to distribute parallel requests across multiple servers, further enhancing parallel processing capabilities. Nginx supports several load balancing methods such as round-robin, least connections, and IP hash.
