# Docker Overview

## 1. What is Docker and Why Is It Used?

**Docker** is a **containerization platform** that packages an application along with all its dependencies into a container. This ensures that it runs **consistently across different environments**.

### How Docker Works
- Applications run inside **containers**, which share the **host OS kernel**.
- Each container includes:
  - The application
  - Required libraries
  - Configuration files  
  *(It does **not** include a full operating system)*

### Why Docker Is Used
- **Consistency:** Eliminates the "works on my machine" problem.
- **Lightweight & Fast:** Containers start much faster than virtual machines.
- **Portability:** Run the same container on development, test, and production environments without changes.

### Simple Example
1. App works on the **developer laptop**.
2. Deploy it to **testing**.
3. Deploy it to **production** — all without changes.

---

## 2. Difference Between Docker and Virtual Machines

| Feature               | Docker (Containers)                  | Virtual Machines (VMs)         |
|-----------------------|------------------------------------|-------------------------------|
| **Architecture**      | Shares host OS kernel               | Runs its own full OS          |
| **Resource Usage**    | Lightweight                         | Heavy                         |
| **Startup Time**      | Seconds                             | Minutes                       |
| **Performance**       | Near native                         | Slight overhead               |
| **Isolation**         | Process-level isolation             | Full OS-level isolation       |
| **Portability**       | Very high                           | Lower than containers         |

---

This README gives a **quick and clear overview** of Docker, its purpose, and how it differs from virtual machines, perfect for beginners.
