
# LINPACK Benchmark: Native vs Docker Performance Study

## 📌 Project Overview

This repository contains the implementation, execution, and performance analysis of the **High-Performance LINPACK (HPL) benchmark** on a single machine using two different execution environments:

1. **Native (Bare-Metal) execution on Ubuntu**
2. **Docker-based containerized execution**

The goal of this project is to **understand how execution environments affect floating-point performance (GFLOPS)** while running the same benchmark on identical hardware.

This work was carried out as part of the **M.Tech (Computer Engineering)** coursework at **COEP Technological University, Pune**.

---

## 🎯 Objectives

* Understand the internal working of **LINPACK / HPL**
* Implement HPL using :

  * Native Ubuntu installation
  * Docker container
* Configure and tune **HPL.dat** parameters
* Execute HPL using **MPI parallelism**
* Measure and compare **execution time and GFLOPS**
* Analyze containerization overhead
* Gain hands-on experience with **HPC benchmarking**

---

## 🧠 What LINPACK (HPL) Does

LINPACK measures system performance by solving a **dense system of linear equations**:

[
A x = b
]

It uses **LU decomposition with partial pivoting**:

[
PA = LU
]

Performance is reported in **GFLOPS (Giga Floating-Point Operations per Second)**.

### FLOPs Formula (Used by HPL)

[
\text{FLOPs} \approx \frac{2}{3} N^3 + 2N^2
]

[
\text{GFLOPS} = \frac{\frac{2}{3} N^3}{\text{Execution Time} \times 10^9}
]

Large values of **N (e.g., 10000)** make this computation heavy and suitable for benchmarking.

---

## 🛠️ System Requirements

### Hardware

* Multi-core CPU
* Minimum **8–16 GB RAM** (for N = 10000)

### Software

* Ubuntu Linux
* GCC & GFortran
* OpenMPI
* OpenBLAS
* Docker (for containerized execution)

---

## ⚙️ Native (Bare-Metal) Implementation

### 1️⃣ Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential gfortran make -y
sudo apt install openmpi-bin libopenmpi-dev -y
sudo apt install libopenblas-dev -y
```

### 2️⃣ Download and Extract HPL

```bash
wget http://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz
tar -xvzf hpl-2.3.tar.gz
cd hpl-2.3
```

### 3️⃣ Configure Makefile

```bash
cd setup
cp Make.Linux_PII_CBLAS ../Makefile
cd ..
nano Makefile
```

Modify:

```text
LAlib = -lopenblas
MPlib = -lmpi
INCmpi = /usr/include/mpi
LIBmpi = /usr/lib/x86_64-linux-gnu
```

Compile:

```bash
make clean
make -j$(nproc)
```

### 4️⃣ Configure `HPL.dat`

Key parameters used:

* `N = 10000`
* `NB = 256`
* `P × Q = 8 × 2 = 16`

> Rule: **P × Q must equal the number of MPI processes**

### 5️⃣ Run HPL

```bash
mpirun --use-hwthread-cpus -np 16 ./xhpl
```

---

## 🐳 Docker-Based Implementation

### 1️⃣ Install Docker

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

### 2️⃣ Pull HPL Docker Image

```bash
docker pull ashael/hpl
```

### 3️⃣ Run Container

```bash
docker run -it ashael/hpl /bin/bash
```

### 4️⃣ Navigate to HPL Directory

```bash
cd /usr/local/hpl-2.2/bin/Linux
```

### 5️⃣ Create `HPL.dat`

Parameters matched native execution:

* `N = 10000`
* `NB = 256`
* `P × Q = 8 × 2`

### 6️⃣ Run HPL Inside Docker

```bash
mpirun --allow-run-as-root -np 16 ./xhpl
```

---

## 📊 Performance Results

| Method          | N     | NB  | P×Q | Time (s) | GFLOPS |
| --------------- | ----- | --- | --- | -------- | ------ |
| Native (Ubuntu) | 10000 | 256 | 8×2 | 11.30    | 59.01  |
| Docker          | 10000 | 256 | 8×2 | 21.49    | 31.03  |

### 🔍 Observations

* Native execution is **~1.93× faster**
* Docker introduces overhead due to:

  * Container abstraction
  * Different BLAS build
  * MPI running as root
  * CPU scheduling constraints
* Native execution benefits from **direct hardware access**

---

## ⚠️ Challenges & Solutions

| Issue                    | Cause              | Solution                   |
| ------------------------ | ------------------ | -------------------------- |
| HPL crash for large N    | Insufficient RAM   | Reduced N to 10000         |
| MPI blocked in Docker    | Root restriction   | Used `--allow-run-as-root` |
| Immediate MPI abort      | P×Q ≠ `-np`        | Ensured P×Q = processes    |
| Lower Docker performance | Container overhead | Expected behavior          |

---

## 📈 Impact of the Experiment

* Strong understanding of **HPC benchmarking**
* Practical exposure to **MPI, BLAS, and compilation**
* Clear insight into **native vs container performance**
* Learned importance of tuning:

  * Matrix size (N)
  * Block size (NB)
  * Process grid (P×Q)

---

## 🔮 Future Scope

* Multi-node MPI cluster benchmarking
* GPU benchmarking using **HPL-CUDA**
* Compare **OpenBLAS vs BLIS vs Intel MKL**
* Use **Singularity** instead of Docker for HPC
* Automated parameter tuning
* Energy efficiency analysis (Green500-style)

---

## 📚 References

1. J. Dongarra, *The LINPACK Benchmark: Past, Present, and Future*
2. Netlib HPL – [https://www.netlib.org/benchmark/hpl/](https://www.netlib.org/benchmark/hpl/)
3. TOP500 Project – [https://www.top500.org](https://www.top500.org)
4. OpenMPI – [https://www.open-mpi.org](https://www.open-mpi.org)
5. OpenBLAS – [https://www.openblas.net](https://www.openblas.net)

