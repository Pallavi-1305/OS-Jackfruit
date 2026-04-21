# OS-Jackfruit — Supervised Multi-Container Runtime

## 1. Team Information

| Name                      | SRN          |
| ------------------------- | ------------ |
| Pallavi Rajkumar Bubanale | PES1UG24CS314 |
| Pooja Avadhani            | PES1UG24CS323 |

---

## 2. Build, Load, and Run Instructions

### Environment

* Ubuntu 24.04 Virtual Machine
* GCC
* Make
* Linux Kernel Headers

### Build Project

```bash
cd ~/OS-Jackfruit/boilerplate
make
```

### Run Supervisor

```bash
sudo ./engine supervisor ./rootfs-base
```

### Start Containers (Screenshot Commands)

```bash
sudo ./engine start alpha ./rootfs-alpha /cpu_hog --soft-mib 48 --hard-mib 80
sudo ./engine start beta ./rootfs-beta /cpu_hog --soft-mib 64 --hard-mib 96
sudo ./engine ps
```

### Working Alternative Commands

```bash
./engine start alpha ./rootfs-alpha /bin/date
./engine start beta ./rootfs-beta /bin/date
./engine ps
```

### View Logs

```bash
sudo ./engine logs alpha
sudo ./engine logs beta
cat logs/alpha.log
cat logs/beta.log
```

### Stop Container

```bash
sudo ./engine stop alpha
sudo ./engine ps
```

### Load Kernel Module

```bash
sudo insmod monitor.ko
ls -l /dev/container_monitor
sudo dmesg | grep container_monitor | tail -20
```

### Scheduler Commands

```bash
time ./cpu_hog
nice -n 10 ./cpu_hog
top
```

### Cleanup

```bash
sudo rmmod monitor
sudo dmesg | grep container_monitor | tail -5
ps aux | grep defunct
```

---

# 3. Demo Screenshots

## Screenshot 1: Successful Build

```bash
cd ~/OS-Jackfruit/boilerplate
make
```

<img width="812" height="258" alt="image" src="https://github.com/user-attachments/assets/3caf519b-b609-405e-ab2e-ff1f7c7a9884" />


---

## Screenshot 2: Supervisor Running

```bash
sudo ./engine supervisor ./rootfs-base
```

<img width="911" height="92" alt="image" src="https://github.com/user-attachments/assets/db3977c9-28eb-4074-8801-f54a05de804f" />


---

## Screenshot 3: Two Containers Running Under One Supervisor

```bash
sudo ./engine start alpha ./rootfs-alpha /cpu_hog --soft-mib 48 --hard-mib 80
sudo ./engine start beta ./rootfs-beta /cpu_hog --soft-mib 64 --hard-mib 96
sudo ./engine ps
```

Working alternative:

```bash
sudo ./engine start alpha ./rootfs-alpha /bin/date
sudo ./engine start beta ./rootfs-beta /bin/date
sudo ./engine ps
```
<img width="1001" height="270" alt="image" src="https://github.com/user-attachments/assets/30b8b314-eaa3-4b4f-a263-2ec873d5b8a9" />




---

## Screenshot 4: Container Metadata / PS Output

```bash
sudo ./engine ps
```

<img width="729" height="205" alt="image" src="https://github.com/user-attachments/assets/9ba171e9-8863-4043-ae71-358c774429c3" />


---

## Screenshot 5: Container Logs Showing Captured Output

```bash
sudo ./engine logs alpha
sudo ./engine logs beta
cat logs/alpha.log
cat logs/beta.log
```
<img width="1060" height="405" alt="image" src="https://github.com/user-attachments/assets/903dc9b4-91d9-4560-b123-d38f144e5c2f" />




---

## Screenshot 6: Stopping Test Container

```bash
sudo ./engine stop alpha
sudo ./engine ps
```

<img width="776" height="286" alt="image" src="https://github.com/user-attachments/assets/7984461f-5ed8-4c48-be7a-e3d10ab5651b" />


---

## Screenshot 7: Kernel Module Activity / dmesg Output

```bash
sudo insmod monitor.ko
sudo dmesg | grep container_monitor | tail -20
```

<img width="944" height="78" alt="image" src="https://github.com/user-attachments/assets/93a42430-fafc-4bbf-9702-6b829af18d7f" />


---

## Screenshot 8: Device File Created

```bash
ls -l /dev/container_monitor
```

<img width="951" height="167" alt="image" src="https://github.com/user-attachments/assets/568a68bd-4b9d-44d7-8a16-8329a311bbdb" />


---

## Screenshot 9: Scheduler Experiment / CPU Usage

```bash
top
time ./cpu_hog
nice -n 10 ./cpu_hog
```

<img width="1208" height="728" alt="image" src="https://github.com/user-attachments/assets/36fb04ae-50ba-492f-ba8e-37319c4c6116" />

<img width="838" height="273" alt="image" src="https://github.com/user-attachments/assets/42e06d32-6734-41f7-94ab-a3429dc1028e" />


<img width="796" height="179" alt="image" src="https://github.com/user-attachments/assets/0c53384d-8388-407a-96bf-b570258fc350" />


---

## Screenshot 10: Module Unloaded / Zero Zombies

```bash
sudo rmmod monitor
sudo dmesg | grep container_monitor | tail -5
ps aux | grep defunct
```

<img width="972" height="159" alt="image" src="https://github.com/user-attachments/assets/f532f485-ed6d-49d0-94b4-1c71c4e35a4b" />


---

# 4. Engineering Analysis

## 4.1 Supervisor and Lifecycle Management

* Long-running supervisor handles CLI requests
* Tracks container ID, PID, state, and limits
* Reaps child processes using `waitpid()`
* Graceful shutdown with Ctrl + C

## 4.2 IPC and Synchronization

* UNIX domain socket used for control communication
* Shared metadata protected with mutexes
* Logging helpers included

## 4.3 Logging System

* Per-container logs stored in `logs/`
* Logs accessible through `logs` command

## 4.4 Memory Management

* Kernel module creates `/dev/container_monitor`
* Supports process tracking
* Soft and hard memory limit framework

## 4.5 Scheduling Behavior

* CPU-bound workload tested using `cpu_hog`
* Priority tested using `nice -n 10`
* Demonstrates scheduler fairness

---

# 5. Design Decisions and Tradeoffs

* Lightweight educational runtime instead of full container engine
* UNIX sockets chosen for simple local IPC
* File-based logs for easy debugging
* Separate user-space and kernel-space components
* Focus on clarity, correctness, and demonstrable features

---

# 6. Scheduler Experiment Results

## Experiment 1: Different Priorities

* Compared default priority vs `nice -n 10`
* Lower priority receives less favorable scheduling under load

## Experiment 2: Multi-Container Management

* Multiple containers handled simultaneously
* Logs captured independently
* States updated correctly

---

# 7. Challenges Faced

* Kernel module compilation in VM
* IPC setup and request handling
* Child process lifecycle management
* Preventing zombie processes
* Graceful shutdown behavior
* GitHub token authentication

---

# 8. Conclusion

This project demonstrates key Operating Systems concepts through a supervised multi-container runtime in C with a kernel memory monitor. It covers process management, IPC, synchronization, scheduling, logging, and systems programming in a practical way.
