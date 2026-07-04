# Port Scanner

> A fast, multithreaded TCP port scanner written in pure Python — no third-party dependencies.

## Overview

Port Scanner (`ps.py`) is a lightweight command-line tool that discovers open TCP ports on a target host. It performs a classic **TCP connect scan**: for each port in a user-defined range, it attempts to open a socket connection to the target and records the port as open if the connection succeeds.

To keep scans fast, the tool spreads the work across a configurable pool of worker threads that all pull ports from a shared, lazily-evaluated generator. The scan range, thread count, and connection behavior are all controllable from the command line, and the tool reports the list of open ports along with the total time taken.

The entire tool is built on the Python standard library, so there is nothing to install beyond Python 3 itself.

## Features

- **TCP connect scanning** — establishes a real TCP connection to each port using Python sockets to determine whether it is open.
- **Multithreaded** — dispatches a configurable pool of worker threads (`-t`, default `500`) to scan ports concurrently instead of one at a time.
- **Custom port ranges** — scan any contiguous range via `-s`/`--start` and `-e`/`--end` (defaults to the full range `1`–`65535`).
- **Per-connection timeout** — each connection attempt is capped at 1 second so unreachable/filtered ports don't stall the scan.
- **Verbose live output** — with `-V`, the growing list of discovered open ports is printed and updated in place as the scan runs.
- **Scan timing** — reports the total elapsed wall-clock time for the scan.
- **Zero dependencies** — uses only the Python standard library (`socket`, `threading`, `argparse`, `time`).

## Tech Stack

- **Language:** Python 3
- **Standard library only:**
  - `socket` — TCP connection attempts
  - `threading` — concurrent scanning via a worker-thread pool
  - `argparse` — command-line argument parsing
  - `time` — measuring scan duration

## How It Works

The program is organized into a few small functions in `ps.py`:

1. **`prepare_args()`** — Uses `argparse` to parse the target host (positional `IPv4` argument) and options: start port, end port, thread count, verbose flag, and version.

2. **`prepare_ports(start, end)`** — A generator that lazily yields every port in `range(start, end + 1)`. Because it is a generator, threads can safely pull the *next* port to scan on demand rather than pre-building a large list.

3. **`scan_port()`** — The work each thread runs. In a loop it:
   - Creates a socket and sets a 1-second timeout (`s.settimeout(1)`).
   - Grabs the next port from the shared generator with `next(ports)`.
   - Attempts `s.connect((ip, port))`. A successful connect means the port is **open**, so it is appended to the shared `open_ports` list.
   - Catches `ConnectionRefusedError` and `socket.timeout` (port closed/filtered) and continues to the next port.
   - Stops when the generator is exhausted (`StopIteration`).

4. **`prepare_threads(threads)`** — Builds the thread pool, starting all worker threads and then `join()`-ing them so the program waits for every thread to finish before reporting.

5. **`__main__`** — Wires it together: parses arguments, builds the port generator, records the start time, runs the threaded scan, and finally prints the list of open ports and the total time taken.

Results are reported by printing the collected `open_ports` list once the scan completes (and, in verbose mode, updating it live during the scan).

## Getting Started

### Prerequisites

- **Python 3** (works on Linux, macOS, and Windows)
- No additional packages required.

### Installation

```bash
git clone https://github.com/Apilex100/Port-Scanner.git
cd Port-Scanner/
```

### Run

The only required argument is the target host (an IPv4 address):

```bash
python ps.py <IPv4>
```

View all available options:

```bash
python ps.py -h
```

```
usage: python ps.py 192.168.137.52

Python Based Fast Port Scanner

positional arguments:
  IPv4              host to scan

options:
  -h, --help        show this help message and exit
  -s, --start       starting port (default: 1)
  -e, --end         ending port (default: 65535)
  -t, --threads     threads to use (default: 500)
  -V, --verbose     verbose output
  -v, --version     display version

Example - ps.py -s 20 -e 40000 -t 500 -V 192.168.137.52
```

## Usage / Examples

Scan the full port range (1–65535) on a host with default settings:

```bash
python ps.py 192.168.137.52
```

Scan a specific port range (20 to 40000) with 500 threads and verbose live output:

```bash
python ps.py -s 20 -e 40000 -t 500 -V 192.168.137.52
```

Scan only the well-known ports (1–1024):

```bash
python ps.py -s 1 -e 1024 192.168.137.52
```

Example output:

```
Open Ports Found - [22, 80, 443]
Time Taken - 3.14
```

## Project Structure

```
Port-Scanner/
├── ps.py       # The complete port scanner: argument parsing, threaded TCP connect scan, reporting
└── README.md   # Project documentation
```

## Disclaimer

This tool is intended for **educational purposes and authorized security testing only**. Port scanning systems you do not own or do not have explicit written permission to test may be illegal and is a violation of many acceptable-use policies. Only scan hosts and networks that you own or are explicitly authorized to assess. The author assumes no liability for any misuse or damage caused by this program.

## Author

**Aviral Kumar Singh** — [https://github.com/Apilex100](https://github.com/Apilex100)
