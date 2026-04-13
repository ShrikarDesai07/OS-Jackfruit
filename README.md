# Linux Multi-Container Runtime

A lightweight Linux container runtime built from scratch using C. This project demonstrates container creation, monitoring, scheduling, memory limits, and supervisor-based lifecycle management using Linux namespaces and cgroups.

---

## Features

* Start and stop isolated containers
* CPU-bound and I/O-bound workloads
* Soft and hard memory limits
* Supervisor daemon for lifecycle management
* Container monitoring using a kernel module
* Priority-based scheduling
* Logging support for each container
* Graceful shutdown and cleanup

---

## Project Structure

```text
.
├── src/
│   ├── engine
│   ├── supervisor
│   ├── monitor
│   └── workloads/
├── rootfs-alpha/
├── rootfs-beta/
├── rootfs-base/
├── screenshots/
└── README.md
```

---

## Requirements

* Linux system or virtual machine
* GCC compiler
* Make
* Root privileges
* Linux kernel headers

Install dependencies:

```bash
sudo apt update
sudo apt install build-essential make gcc linux-headers-$(uname -r)
```

---

## Build Instructions

```bash
make clean
make
```

---

## Running the Supervisor

Start the supervisor in one terminal:

```bash
sudo ./src/engine supervisor ./rootfs-base
```

### Supervisor Startup

![Supervisor Startup](screenshots/1.jpeg)

The supervisor initializes the runtime socket, loads the base root filesystem, and waits for container requests.

---

## Starting Containers

Start the CPU-bound and I/O-bound containers:

```bash
sudo ./src/engine start alpha ./rootfs-alpha "./cpu_hog 15" --soft-mib 48 --hard-mib 80

sudo ./src/engine start beta ./rootfs-beta "./io_pulse 30 100" --soft-mib 64 --hard-mib 96
```

Check the running containers:

```bash
sudo ./src/engine ps
```

### Alpha and Beta Containers Running

![Container Start](screenshots/2.jpeg)

Both containers are successfully started and visible in the container list.

---

## Viewing Container Logs

View logs for each container:

```bash
sudo ./src/engine logs alpha
sudo ./src/engine logs beta
```

### Alpha Container Logs

![Alpha Logs](screenshots/3.jpeg)

The alpha container runs a CPU-intensive workload using `cpu_hog`.

### Beta Container Logs

![Beta Logs](screenshots/4.jpeg)

The beta container runs an I/O-intensive workload using `io_pulse`.

---

## Testing Soft and Hard Memory Limits

Start a memory-intensive container:

```bash
sudo ./src/engine start delta ./rootfs-alpha "./memory_hog 25 5000" --soft-mib 20 --hard-mib 100
```

### Memory Hog Container Start

![Memory Hog Start](screenshots/5.jpeg)

The `delta` container is started with a soft memory limit of 20 MiB and a hard memory limit of 100 MiB.

### Soft and Hard Limit Triggered

![Memory Limit Trigger](screenshots/6.jpeg)

The kernel monitor first reports a soft memory warning and later kills the container after exceeding the hard memory limit.

---

## Priority Scheduling Test

Start two CPU-heavy containers:

```bash
sudo ./src/engine start high_prio ./rootfs-alpha "./cpu_hog 15"

sudo ./src/engine start low_prio ./rootfs-alpha "./cpu_hog 15"
```

View their logs:

```bash
sudo ./src/engine logs high_prio
sudo ./src/engine logs low_prio
```

### High Priority Container Logs

![High Priority Logs](screenshots/7.jpeg)

### Low Priority Container Logs

![Low Priority Logs](screenshots/8.jpeg)

The higher-priority workload finishes earlier than the lower-priority workload.

---

## Stopping Containers

```bash
sudo ./src/engine stop delta
sudo ./src/engine stop high_prio
sudo ./src/engine stop low_prio
```

### Stop Commands

![Stop Containers](screenshots/9.jpeg)

The runtime confirms that the containers are no longer running.

---

## Supervisor Shutdown and Cleanup

Shut down the supervisor and remove the kernel module:

```bash
sudo ./src/engine supervisor-stop
sudo rmmod monitor
```

Verify cleanup:

```bash
ps aux | grep defunct
sudo dmesg | tail -n 5
```

### Supervisor Shutdown

![Supervisor Shutdown](screenshots/10.1.jpeg)

This screenshot shows the supervisor shutting down cleanly and stopping all remaining containers.

### Monitor Module Unload

![Monitor Unload](screenshots/10.2.jpeg)

This screenshot shows the kernel monitor module being unloaded successfully with no zombie processes left behind.

---

## Expected Output

* Containers should start and stop correctly
* Soft memory limit should generate warnings
* Hard memory limit should terminate the container
* High-priority workload should complete earlier than low-priority workload
* Logs should be generated correctly for each container
* Supervisor should shut down cleanly
* The monitor kernel module should unload successfully

---

## Author

Developed as p
