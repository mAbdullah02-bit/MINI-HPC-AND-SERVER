
# Mini HPC Cluster and Server Storage Setup

## 📌 Purpose and Motivation

This project documents the setup of a **mini High-Performance Computing (HPC) cluster** using a **laptop as the master node** and a **PC as the compute server** with shared persistent storage. The goal is to:

- Use the PC’s CPU and storage for heavy computation.
- Use the laptop to submit, monitor, and control tasks remotely.
- Create shared storage that remains available even after system restarts.
- Learn about distributed computing, job scheduling, and network storage.

---

## 🔷 Part 1: Shared Storage Setup

This part focuses on setting up a **persistent, shared storage system** between the laptop and PC using **NFS (Network File System)**.

### ✅ 1. Assign a Static IP to PC (Server Node)

Use your **router settings** to assign a **static IP** to your PC so it stays the same across reboots.

### ✅ 2. Enable SSH on the PC

Install and start SSH on the PC (Ubuntu):

```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

### ✅ 3. Test SSH Connection from Laptop

```bash
ssh pcuser@<PC-IP>
```

Replace `<PC-IP>` with the static IP address of your PC.

### ✅ 4. Create a Shared Directory on PC

```bash
mkdir /home/pcuser/server_storage
chmod 777 /home/pcuser/server_storage
```

### ✅ 5. Install NFS

Install NFS on **both the laptop and PC**:

```bash
sudo apt install nfs-kernel-server nfs-common
```

### ✅ 6. Configure NFS on PC

Edit the exports file:

```bash
sudo nano /etc/exports
```

Add:

```bash
/home/pcuser/server_storage *(rw,sync,no_subtree_check)
```

Apply changes:

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### ✅ 7. Mount Storage on Laptop

```bash
sudo mkdir /home/abdullah/pc_storage
sudo mount <PC-IP>:/home/pcuser/server_storage /home/abdullah/pc_storage
```

### ✅ 8. Make Mount Persistent

Edit `/etc/fstab` on the laptop:

```bash
<PC-IP>:/home/pcuser/server_storage /home/abdullah/pc_storage nfs defaults 0 0
```

### ✅ 9. Test Write Access

```bash
echo "test" > ~/pc_storage/test.txt
```

Check if the file appears on the PC.

### ✅ 10. Setup Passwordless SSH

On the laptop:

```bash
ssh-keygen
ssh-copy-id pcuser@<PC-IP>
```

---

## 🔷 Part 2: HPC Mini Cluster Using SLURM

### 🧩 Purpose

To simulate a basic HPC cluster, we use **SLURM** as the job scheduler on the **PC (compute node)**. The laptop submits and monitors jobs using SSH.

### ✅ 1. Install SLURM on PC

```bash
sudo apt install slurm-wlm
```

### ✅ 2. Configure SLURM

Edit `/etc/slurm-llnl/slurm.conf` (or generate it via [slurm.conf generator](https://slurm.schedmd.com/configurator.html)):

Set:
- ControlMachine to `localhost`
- NodeName = your PC hostname
- PartitionName = compute

Start the service:

```bash
sudo systemctl enable slurmctld slurmd
sudo systemctl start slurmctld slurmd
```

### ✅ 3. Test SLURM Job

Submit a job:

```bash
srun hostname
```

Check job queue:

```bash
squeue
```

---

## ✅ Conclusion and Achievements

- ✅ Laptop is the **master node**.
- ✅ PC is the **compute node** with **SLURM** job scheduling.
- ✅ **Persistent shared storage** using NFS.
- ✅ Seamless SSH-based job submission and control.
- ✅ Works **without internet**, using only **local network (LAN)**.

This setup simulates a **basic HPC cluster**, helped in learning about storage, job scheduling, distributed tasks, and Linux networking. Perfect for personal research, learning, or small-scale compute jobs.
