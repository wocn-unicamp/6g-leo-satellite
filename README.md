# 6g-leo-satellite
5G/6G LEO Satellite Module for NS3

* [Introduction](#introduction)
* [Setup to execute simulations inside Docker Containers](#setup-to-execute-simulations-inside-docker-containers)
  - [Building the Dockerfile](#building-the-dockerfile)
  - [Instantiating the container](#instantiating-the-container)
  - [Inside the container](#inside-the-container)
* [Running without containers](#running-without-containers)
* [Installing leo and epidemic-routing modules](#installing-leo-and-epidemic-routing-modules)
  - [Changing commit from epidemic-routing](#changing-commit-from-epidemic-routing)
  - [Adding a case to leo-delay](#adding-a-case-to-leo-delay)
  - [Configuring and building](#configuring-and-building)

## Introduction

Inside the folder `ns3-mmwave-oran` lies [ns-o-ran-ns3-mmwave](https://github.com/wineslab/ns-o-ran-ns3-mmwave?tab=readme-ov-file), that is a fork of [mmWave ns-3 module](https://github.com/nyuwireless-unipd/ns3-mmwave). It was built upon ns3 version 3.33, and contains modifications on important models present in the `/src` folder in order to work properly with the [ns-O-RAN](https://github.com/o-ran-sc/sim-ns3-o-ran-e2) module.

Additionally, three modules have been installed in `/contrib`:
- `oran-interface`: this is where the code for the [ns-O-RAN](https://github.com/o-ran-sc/sim-ns3-o-ran-e2) module actually resides.
- `leo`: installed from this [repository](https://gitlab.ibr.cs.tu-bs.de/tschuber/ns-3-leo). Where the code for the `leo` module resides.
- `epidemic-routing`: this [module](https://gitlab.com/tomhend/modules/epidemic-routing) is necessary in order to build and execute some examples from the `leo` module.

The installation of `oran-interface` is described in [5G ns-3 with O-RAN near-RT RIC bronze version tutorial @ OpenRanGym](https://openrangym.com/tutorials/ns-o-ran).

The installation of `leo` and `epidemic-routing` are our contribution.

## Setup to execute simulations inside Docker Containers

While it is not necessary to execute the simulation inside Docker containers, it is **highly recommended** due to the fact that the image already has all the dependencies properly installed, along with a working version of ns3 (v3.33) with ns-O-RAN and mmwave modules installed.

### Building the Dockerfile

In the repository of the [Colosseum Near-Real-Time RIC](https://github.com/wineslab/colosseum-near-rt-ric), maintained by [WiNES Lab](https://ece.northeastern.edu/wineslab/), on the [ns-o-ran branch](https://github.com/wineslab/colosseum-near-rt-ric/tree/ns-o-ran), there is a Dockerfile at the root of the repository.

With this Dockerfile we are able to build an Ubuntu (18.04) image that:
- Already has `ns-o-ran-ns3-mmwave` installed under `/workspace/ns-o-ran-ns3-mmwave`.
- It also has all the necessary dependencies installed, including the library [E2 sim fork for ns-O-RAN](https://github.com/wineslab/o-ran-e2sim), that is a fork from the [e2sim](https://github.com/o-ran-sc/sim-e2-interface) library. This fork has updates to make e2sim work with the [ns3-o-ran-e2](https://github.com/o-ran-sc/sim-ns3-o-ran-e2) (a.k.a ns-O-RAN) ns-3 module. e2sim enables the support for running multiple terminations of an O-RAN-compliant E2 interface inside the simulation process.

To build the image, change directory to the root of this repository, where the Dockerfile is located, and execute:

```bash
docker build --network=host --tag ns_o_ran:latest .
```

This command will build an image called `ns_o_ran`.

### Instantiating the container

With the image built, it is possible to  mount the `ns3-mmwave-oran` folder from this repository into the container, so that changes made in the repository, at the host machine, are reflected into the container.

To instantiate the container and interact via shell:

```bash
docker run -it --network=host -v /path/to/repo/ns3-mmwave-oran:/workspace/ns3-mmwave-oran ns_o_ran:latest /bin/bash
```

### Inside the container

The `ns3` folder is at `/workspace/ns3-mmwave-oran`.

After changing directory to the folder you may configure, build and execute programs using `waf`.

## Running without containers

To run without containers, please refer to the [5G ns-3 with O-RAN near-RT RIC bronze version tutorial @ OpenRanGym](https://openrangym.com/tutorials/ns-o-ran), under ns-O-RAN Setup.

## Installing `leo` and `epidemic-routing` modules

**Please note that these modules have already been installed in this repository**, and this section merely describes the steps on how to install this modules for reference.

Downloading the repositories under `/contrib`:

```bash
cd ns3-mmwave-oran/contrib
git clone 
https://gitlab.ibr.cs.tu-bs.de/tschuber/ns-3-leo.git leo
git clone https://gitlab.com/tomhend/modules/epidemic-routing.git epidemic-routing
```

There are some minor tweaks that must be made before configuring and building:
- The `leo-delay` example from the `leo` module is missing a `case` inside a `switch`. Perhaps there may be a way to ignore this error. But adding the missing `case` also works.
- `epidemic-routing` uses `CMakeLists` instead of `wscript`, because it has been adapted to work with newer versions of ns3, so we need to reset the repository to `HEAD~1`.

### Changing commit from `epidemic-routing`

This step is only necessary if your ns3 version does not support `CMakeLists`.

```bash
cd epidemic-routing
git reset --hard HEAD~1
```

### Adding a case to `leo-delay`

Add the following `case` in `contrib/leo/examples/leo-delay-tracing-example.cc:63:10`:

```C++
case Ipv4L3Protocol::DROP_DUPLICATE:
  res = "Duplicate packet received";
  break;
```

### Configuring and building

Now we are ready to configure and build. If you are running the simulation in containers remember to mount your host's version of the `ns3-mmwave-oran` folder.

```bash
# skip this step if you are not running ns3 with docker
docker run -it --network=host -v /host/path/to/ns3-mmwave-oran:/workspace/ns3-mmwave-oran <image_name>:latest /bin/bash
```

To configure and build:

```
./waf configure --enable-examples --enable-tests --enable-mpi
./waf build
```