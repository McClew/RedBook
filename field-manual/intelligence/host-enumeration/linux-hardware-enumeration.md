---
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Linux Hardware Enumeration

## List CPU

`lscpu`: This command (short for "list CPU") displays information about the CPU architecture. It gathers this information from `/proc/cpuinfo` and presents it in a more human-readable format.

`lscpu` provides a wealth of detail about your system's CPUs, including:

* Architecture: (e.g., x86\_64)
* CPU op-mode(s): Whether it supports 32-bit, 64-bit, or both.
* Byte Order: (e.g., Little Endian)
* CPU(s): Total number of logical CPUs.
* On-line CPU(s) list: Which CPUs are currently online.
* Core(s) per socket: Number of CPU cores per physical socket.
* Socket(s): Number of physical CPU sockets.
* Thread(s) per core: Number of threads per core (Hyper-threading).
* Vendor ID: (e.g., GenuineIntel, AuthenticAMD)
* CPU family: Indicates the specific family or generation of the processor's architecture, providing a general classification for the CPU's design.
* Model name: Full name of the CPU (e.g., Intel(R) Core(TM) i7-10700K CPU @ 3.80GHz).
* L1d cache, L1i cache, L2 cache, L3 cache: Information about the different levels of CPU cache.
* Vulnerability information: Details on known CPU vulnerabilities and their mitigation status (e.g., Spectre, Meltdown).

{% code title="Command" %}
```bash
lscpu
```
{% endcode %}

{% code title="Example Output" %}
```bash
<snip>
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              16
On-line CPU(s) list: 0-15
Core(s) per socket:  8
Socket(s):           1
Thread(s) per core:  2
Vendor ID:           GenuineIntel
CPU family:          6
Model:               165
Model name:          Intel(R) Core(TM) i7-10700K CPU @ 3.80GHz
Stepping:            5
CPU MHz:             800.000
CPU max MHz:         5100.0000
CPU min MHz:         800.0000
BogoMIPS:            7600.00
L1d cache:           256 KiB
L1i cache:           256 KiB
L2 cache:            2 MiB
L3 cache:            16 MiB
<snip>
```
{% endcode %}
