# Pantheon of Congestion Control
The Pantheon contains wrappers for many popular practical and research
congestion control schemes. The Pantheon enables them to run on a common
interface, and has tools to benchmark and compare their performances.
Pantheon tests can be run locally over emulated links using
[mahimahi](http://mahimahi.mit.edu/) or over the Internet to a remote machine.

Our website is <https://pantheon.stanford.edu>, where you can find more
information about Pantheon, including supported schemes, measurement results
on a global testbed so far, and our paper at [USENIX ATC 2018](https://www.usenix.org/conference/atc18/presentation/yan-francis)
(**Awarded Best Paper**).
In case you are interested, the scripts and traces
(including "calibrated emulators") for running the testbed can be found in
[observatory](https://github.com/StanfordSNR/observatory).

To discuss and talk about Pantheon-related topics and issues, feel free to
post in the [Google Group](https://groups.google.com/forum/#!forum/pantheon-stanford)
or send an email to `pantheon-stanford <at> googlegroups <dot> com`.

## Disclaimer
This is research software. Our scripts will write to the file system in the
`pantheon` folder. We never run third party programs as root, but we cannot
guarantee they will never try to escalate privilege to root.

You might want to install dependencies and run the setup on your own, because
our handy scripts will install packages and perform some system-wide settings
(e.g., enabling IP forwarding, loading kernel modeuls) as root.
Please run at your own risk.

## Preparation
To clone this repository, run:

```
git clone https://github.com/StanfordSNR/pantheon.git
```

Many of the tools and programs run by the Pantheon are git submodules in the
`third_party` folder. To add submodules after cloning, run:

```
git submodule update --init --recursive  # or tools/fetch_submodules.sh
```

## Dependencies
We provide a handy script `tools/install_deps.sh` to install globally required
dependencies; these dependencies are required before testing **any** scheme
and are different from the flag `--install-deps` below.
In particular, we created the [Pantheon-tunnel](https://github.com/StanfordSNR/pantheon-tunnel)
that is required to instrument each scheme.

You might want to inspect the contents of
`install_deps.sh` and install these dependencies by yourself in case you want to
manage dependencies differently. Please note that Pantheon currently
**only** supports Python 2.7.

Next, for those dependencies required by each congestion control scheme `<cc>`,
run `src/wrappers/<cc>.py deps` to print a dependency list. You could install
them by yourself, or run

```
src/experiments/setup.py --install-deps (--all | --schemes "<cc1> <cc2> ...")
```

to install dependencies required by all schemes or a list of schemes separated
by spaces.

## Setup
After installing dependencies, run

```
src/experiments/setup.py [--setup] [--all | --schemes "<cc1> <cc2> ..."]
```

to set up supported congestion control schemes. `--setup` is required
to be run only once. In contrast, `src/experiments/setup.py` is
required to be run on every reboot (without `--setup`).

## Running the Pantheon
To test schemes in emulated networks locally, run

```
src/experiments/test.py local (--all | --schemes "<cc1> <cc2> ...")
```

To test schemes over the Internet to remote machine, run

```
src/experiments/test.py remote (--all | --schemes "<cc1> <cc2> ...") HOST:PANTHEON-DIR
```

Run `src/experiments/test.py local -h` and `src/experiments/test.py remote -h`
for detailed usage and additional optional arguments, such as multiple flows,
running time, arbitrary set of mahimahi shells for emulation tests,
data sender side for real tests; use `--data-dir DIR` to specify an
an output directory to save logs.

## Pantheon analysis
To analyze test results, run

```
src/analysis/analyze.py --data-dir DIR
```

It will analyze the logs saved by `src/experiments/test.py`, then generate
performance figures and a full PDF report `pantheon_report.pdf`.

## Running a single congestion control scheme
All the available schemes can be found in `src/config.yml`. To run a single
congestion control scheme, first follow the **Dependencies** section to install
the required dependencies.

At the first time of running, run `src/wrappers/<cc>.py setup`
to perform the persistent setup across reboots, such as compilation,
generating or downloading files to send, etc. Then run
`src/wrappers/<cc>.py setup_after_reboot`, which also has to be run on every
reboot. In fact, `test/setup.py` performs `setup_after_reboot` by
default, and runs `setup` on schemes when `--setup` is given.

Next, execute the following command to find the running order for a scheme:
```
src/wrappers/<cc>.py run_first
```

Depending on the output of `run_first`, run

```
# Receiver first
src/wrappers/<cc>.py receiver port
src/wrappers/<cc>.py sender IP port
```

or

```
# Sender first
src/wrappers/<cc>.py sender port
src/wrappers/<cc>.py receiver IP port
```

Run `src/wrappers/<cc>.py -h` for detailed usage.

## How to add your own congestion control
Adding your own congestion control to Pantheon is easy! Just follow these
steps:

1. Fork this repository.

2. Add your congestion control repository as a submodule to `pantheon`:

   ```
   git submodule add <your-cc-repo-url> third_party/<your-cc-repo-name>
   ```

   and add `ignore = dirty` to `.gitmodules` under your submodule.

3. In `src/wrappers`, read `example.py` and create your own `<your-cc-name>.py`.
   Make sure the sender and receiver run longer than 60 seconds; you could also
   leave them running forever without the need to kill them.

4. Add your scheme to `src/config.yml` along with settings of
   `name`, `color` and `marker`, so that `src/experiments/test.py` is able to
   find your scheme and `src/analysis/analyze.py` is able to plot your scheme
   with the specified settings.

5. Add your scheme to `SCHEMES` in `.travis.yml` for continuous integration testing.

6. Send us a pull request and that's it, you're in the Pantheon!

README in Details for the ASSIGNMENT

# Pantheon Network Emulation Framework - Setup and Usage Guide

## Table of Contents
1. [Overview](#overview)
2. [System Requirements](#system-requirements)
3. [Installation](#installation)
4. [Basic Usage](#basic-usage)
5. [Running Experiments](#running-experiments)
6. [Data Collection](#data-collection)
7. [Troubleshooting](#troubleshooting)
8. [References](#references)

## Overview
Pantheon is a framework for evaluating congestion control algorithms under consistent network conditions using mahimahi network emulation. This guide will help you set up the environment and run experiments comparing different congestion control protocols.

## System Requirements
- **Operating System**: Ubuntu 18.04/20.04 LTS (recommended)
- **Memory**: Minimum 4GB RAM
- **Storage**: 10GB free disk space
- **Python**: 2.7 and 3.x
- **Dependencies**: git, build-essential, and other packages listed below

## Installation

### 1. Clone the Pantheon Repository
```bash
git clone https://github.com/StanfordSNR/pantheon.git
cd pantheon
```

### 2. Install System Dependencies
```bash
sudo apt update
sudo apt install -y \
    git build-essential autotools-dev dh-autoreconf \
    iptables protobuf-compiler libprotobuf-dev pkg-config \
    libssl-dev dnsmasq-base ssl-cert libxcb-present-dev \
    libcairo2-dev libpango1.0-dev iproute2 apache2-bin \
    xorg-dev libudns-dev libev-dev libevent-dev libtool \
    apache2-dev python-dev python3-dev python3-pip
```

### 3. Install Mahimahi (Network Emulator)
```bash
git clone https://github.com/ravinet/mahimahi.git
cd mahimahi
./autogen.sh
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
sudo ldconfig
cd ..
```

### 4. Enable IP Forwarding
```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```

### 5. Set Up Python Environment
```bash
# For Python 2.7 support
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
python2.7 get-pip.py
python2.7 -m pip install virtualenv

# Create virtual environment
python2.7 -m virtualenv venv
source venv/bin/activate
pip install -r src/requirements.txt
```

## Basic Usage

### Verify Installation
```bash
./tools/test.sh
```
This will run a basic test between Cubic and BBR congestion control algorithms.

### Test Mahimahi Functionality
```bash
mm-delay 10 echo "Mahimahi test successful"
```

## Running Experiments

### 1. Select Protocols
Choose 3 congestion control protocols from the available options:
- cubic
- bbr
- vegas
- reno
- sprout
- verus
- indigo
- pcc

### 2. Run Experiments

#### High-Bandwidth, Low-Latency Scenario (50Mbps, 10ms RTT)
```bash
python src/experiments/test.py \
    --schemes cubic bbr vegas \
    --mm-delay 10 \
    --bw 50 \
    --run-times 60
```

#### Low-Bandwidth, High-Latency Scenario (1Mbps, 200ms RTT)
```bash
python src/experiments/test.py \
    --schemes cubic bbr vegas \
    --mm-delay 200 \
    --bw 1 \
    --run-times 60
```

### 3. Data Collection
All output data will be saved in:
```
pantheon/analysis/
```
Files include:
- `throughput.csv`
- `delay.csv`
- `loss.csv`
- `stats.txt`

## Data Analysis

### Generate Plots
```bash
python src/analysis/analyze.py \
    --data-dir analysis \
    --schemes cubic bbr vegas
```

Plots will be saved as PNG files in the analysis directory.

## Troubleshooting

### Common Issues

1. **Python 2.7 not found**:
   ```bash
   sudo apt install python2.7 python2.7-dev
   ```

2. **Mahimahi not working**:
   ```bash
   sudo setcap cap_net_admin+ep /usr/local/bin/mm-delay
   sudo setcap cap_net_admin+ep /usr/local/bin/mm-link
   ```

3. **Missing dependencies**:
   ```bash
   sudo apt --fix-broken install
   ```

## References
1. [Pantheon Paper](https://cs.slu.edu/~esposito/pantheon.pdf)
2. [Mahimahi Paper](https://cs.slu.edu/~esposito/mahimahi-sigcomm2014.pdf)
3. [Pantheon GitHub](https://github.com/StanfordSNR/pantheon)

---

