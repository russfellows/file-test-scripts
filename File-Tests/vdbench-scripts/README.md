# vdbench - File System Workload Generator

Flexible file system workload simulation with **vdbench**, supporting both single-host and distributed multi-host execution.

## Container

**Image**: `file-tests:latest`

Contains:
- **vdbench** - File system workload generator with multiple execution modes
- **fio** - Flexible I/O workload simulator
- Various file system testing utilities

### Building the Container

⚠️ **IMPORTANT**: This container must be built locally due to Oracle vdbench licensing restrictions.

**Prerequisites**:
1. Download `vdbench50407.zip` from Oracle: https://www.oracle.com/downloads/server-storage/vdbench-downloads.html
2. Accept Oracle's license agreement
3. Place the zip file in `../../dockerfiles/file-tests/` directory

**Build command**:
```bash
cd ../../dockerfiles/file-tests
# Verify vdbench50407.zip is present
ls -lh vdbench50407.zip
# Build the container
docker build -t file-tests:latest .
```

**For detailed instructions**, see: [../../dockerfiles/file-tests/README.md](../../dockerfiles/file-tests/README.md)

## Overview

vdbench is a comprehensive file system workload generator implementing MLCommons Storage workload specifications. It provides:

- **Flexible Workload Definition**
  - Multiple file types and sizes
  - Sequential and random access patterns
  - Realistic I/O distributions
  - Customizable operation mix (read/write/metadata)

- **Multi-Threading and Distribution**
  - Single-threaded and multi-threaded operation
  - Multi-host distributed execution
  - Per-thread performance tracking

- **Detailed Performance Analysis**
  - Throughput (ops/sec, MB/sec)
  - Latency statistics (average, percentiles)
  - Per-file and per-operation metrics
  - Response time distributions

- **AI/ML Workload Simulation**
  - ResNet50 training patterns
  - UNet3D volumetric processing patterns
  - Complex mixed workloads

## Directory Structure

```
vdbench-scripts/
├── Resnet50/
│   ├── resnet50-1hosts_parmfile.txt
│   ├── resnet50-4hosts_parmfile.txt
│   └── resnet50-8hosts_parmfile.txt
└── Unet3d/
    ├── unet3d-1hosts_parmfile.txt
    ├── unet3d-4hosts_parmfile.txt
    └── unet3d-8hosts_parmfile.txt
```

**File naming convention**: `{workload}-{num_hosts}_parmfile.txt`

## Workload Patterns

### Resnet50
ResNet50 deep learning model training workload patterns:
- **Files**:
  - `resnet50-1hosts_parmfile.txt` - Single-host configuration
  - `resnet50-4hosts_parmfile.txt` - 4-host distributed configuration
  - `resnet50-8hosts_parmfile.txt` - 8-host distributed configuration

**Use case**: Image classification model training
**Data access**: Batch-sequential with periodic random access
**Typical characteristics**: 
- Large file reads (ImageNet batches)
- Sequential prefetching
- Metadata-light operations
**Available**: 1-host, 4-host, 8-host configurations

### Unet3d
3D UNet segmentation model workload patterns:
- **Files**:
  - `unet3d-1hosts_parmfile.txt` - Single-host configuration
  - `unet3d-4hosts_parmfile.txt` - 4-host distributed configuration
  - `unet3d-8hosts_parmfile.txt` - 8-host distributed configuration

**Use case**: Medical imaging, volumetric data processing
**Data access**: 3D volume chunks with spatial locality
**Typical characteristics**:
- Random access within volumes
- Mixed read/write (gradient updates)
- Higher metadata overhead
**Available**: 1-host, 4-host, 8-host configurations

## Running vdbench Tests

### Prerequisites
- Docker or container runtime
- File system mount point (local or network)
- For multi-host tests: multiple test hosts with shared or synchronized storage

### Two Ways to Run vdbench

vdbench can be executed in two modes:

#### Mode 1: Agent (Listening) Mode - `start_vdb-agent.sh`

Start vdbench in listening mode to accept commands from a controller/coordinator:

```bash
#!/bin/bash
# start_vdb-agent.sh
docker run --rm -v /mnt/storage:/mnt/storage --net=host -it file-tests \
  "/opt/vdbench/vdbench" "rsh"
```

**Usage**:
```bash
bash start_vdb-agent.sh
```

**What it does**:
- Starts vdbench in listening mode (rsh protocol)
- Waits for commands from a controller on the network
- Suitable for distributed multi-host testing where a central controller coordinates all agents
- Mounts `/mnt/storage` (adjust path as needed) from host into container

**When to use**:
- Distributed benchmarks with multiple coordinated agents
- Controlled hierarchical execution across multiple test nodes

#### Mode 2: Interactive Container - `start_vdb.sh`

Start the container interactively and manually run vdbench from within:

```bash
#!/bin/bash
# start_vdb.sh
docker run -v /mnt/storage:/mnt/storage --net=host -it file-tests
```

**Usage**:
```bash
# Start container interactively
bash start_vdb.sh

# Inside the container, run vdbench with your config
cd /opt/vdbench
./vdbench -f vdbench-scripts/Resnet50/resnet50-1hosts_parmfile.txt -o /mnt/storage/output
```

**Command-line options**:
- `-f <config_file>` - Path to vdbench parameter file (required)
- `-o <output_dir>` - Output directory for results (required)

**Examples inside container**:
```bash
# ResNet50 single-host test
./vdbench -f vdbench-scripts/Resnet50/resnet50-1hosts_parmfile.txt -o /mnt/storage/results/resnet50-1h

# UNet3D 4-host distributed test
./vdbench -f vdbench-scripts/Unet3d/unet3d-4hosts_parmfile.txt -o /mnt/storage/results/unet3d-4h

# Custom parameters (8 threads, 300 second duration)
./vdbench -f custom_config.txt -o /mnt/storage/results/custom
```

**When to use**:
- Single-host benchmarks
- Manual testing and debugging
- Standalone execution without coordinating across multiple agents

### Single-Host Test Example

```bash
# Build the container first (see "Building the Container" section above)

# Option 1: Direct execution
docker run -it -v /test/mount:/testdir \
  file-tests:latest \
  vdbench -f resnet50-1hosts_parmfile.txt -o /testdir/output

# Option 2: Using start_vdb.sh
bash start_vdb.sh
# Then inside container:
cd /opt/vdbench && ./vdbench -f resnet50-1hosts_parmfile.txt -o /mnt/storage/output
```

### Multi-Host Distributed Test

For distributed testing across multiple hosts:

1. **Prepare test hosts** with shared storage access
2. **On each agent host**, start listening mode:
   ```bash
   bash start_vdb-agent.sh
   ```
3. **On coordinator host**, run vdbench with distributed config:
   ```bash
   docker run -it \
     -e VDBENCH_HOSTS="host1,host2,host3,host4" \
     -v /test/mount:/testdir \
     file-tests:latest \
     vdbench -f resnet50-4hosts_parmfile.txt -o /testdir/output
   ```
## vdbench Configuration File Format

vdbench uses parameter files (.txt) with sections:

- **RD (Run Definition)**: Defines one or more workload phases
- **FSD (File System Definition)**: File types, sizes, and directory structure
- **FWD (File Work Definition)**: I/O operations (read, write, delete, mkdir, etc.)
- **Performance settings**: Thread counts, test duration, reporting intervals

### Parameter File Sections

```
# Run Definition - overall test structure
rd=resnet50_workload,operation=read,threads=16,duration=300

# File System Definition - directory and file layout
fsd=resnet50_files,depth=3,width=10,files=100,filesize=10m

# File Work Definition - I/O operations
fwd=resnet50_io,threads=8,rdpct=90,wrpct=10,seekpct=0
```

## Output and Analysis

### vdbench Output
- **Real-time metrics**: Throughput (ops/sec, MB/sec), latency
- **Summary reports**: Aggregate performance across test duration
- **Detailed logs**: Per-operation timing and statistics
- **Charts/graphs**: Performance visualization (if enabled)

## Documentation Resources

### vdbench Documentation
- **vdbench User Guide (v50407)** - Complete reference documentation
  - PDF: Available from Oracle/Delphix vdbench repository
  - Covers parameter file syntax, workload definition, tuning guidelines
  - Includes troubleshooting and best practices for file system testing
- **vdbench Manual**: https://www.oracle.com/downloads/cloud/open-source/vdbench/

## Troubleshooting

### Mount Issues
- Verify file system is mounted before starting container
- Check container user has read/write permissions
- For NFS mounts, verify NFS service is running

### Performance Issues
- Check network connectivity (for distributed tests)
- Monitor file system cache behavior
- Verify storage backend is not bottlenecked
- Check CPU and memory utilization

### Configuration Errors
- Validate parameter file syntax
- Ensure referenced paths exist
- Check host name resolution (for distributed tests)

## Performance Tuning

### For Better Cache Utilization
- Reduce test data size relative to system memory
- Enable read-ahead on file systems
- Adjust buffer sizes in configuration

### For Consistency
- Run multiple iterations to warm up storage
- Synchronize clocks across distributed test hosts
- Use consistent file system mount options

## Related Documentation

- Main repository: [../../README.md](../../README.md)
- File system testing overview: [../README.md](../README.md)
- IOR testing: [../IOR-scripts/README.md](../IOR-scripts/README.md)
- Container build instructions: [../../dockerfiles/file-tests/README.md](../../dockerfiles/file-tests/README.md)