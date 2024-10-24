# nginx-parallel-process
Parallel process by Nginx and System 
# 1. Nginx Configuration for Parallel Processing
Nginx uses a multi-threaded and multi-process architecture. To enable and optimize parallel processing in Nginx, you can tune the following directives in your Nginx configuration (`nginx.conf`):

a. Worker Processes (`worker_processes`)

  - This controls the number of worker processes that Nginx will spawn. Each worker process can handle multiple connections, so increasing the number of worker processes will improve Nginx's ability to handle parallel requests.
  - Recommended Setting: Set `worker_processes` to the number of CPU cores available on your server to fully utilize parallelism.
  - Example:

![image](https://github.com/user-attachments/assets/71a22708-e849-4fad-84d5-c398222d7ce7)

This will automatically set the number of worker processes to match the number of CPU cores (auto-detect).
