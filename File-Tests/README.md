# File System Testing

This directory contains configuration files for testing file systems using **IOR** and **vdbench**, two industry-standard file system benchmarking tools.

## Testing Tools

### IO500 - HPC Storage Benchmark Suite
**Container**: `io500:latest`

IO500 is the official HPC storage benchmark suite that coordinates multiple benchmarking tools for distributed file system testing:
- **IOR** (Interleaved-Or-Random) - Parallel file I/O operations with ior-easy and ior-hard tiers
- **MDtest** - Metadata operation benchmarking with mdtest-easy and mdtest-hard tiers
- **pfind** - File discovery and directory traversal benchmarking
- Multi-host MPI-based distributed execution
- MLCommons Storage workload patterns
- Support for Parallel File System, Parallel File System, POSIX file systems
- Comprehensive composite scoring (bandwidth + IOPS)

**For detailed setup and execution instructions, see**: [IOR-scripts/README.md](IOR-scripts/README.md)

### vdbench - File System Workload Generator
**Container**: `file-tests:latest`

vdbench is a flexible file system workload simulator featuring:
- Agent and interactive container modes
- Single and multi-threaded operation
- Complex workload definitions with multiple file types
- Detailed performance analysis and statistics
- Reproducible workload execution

**For detailed setup and execution instructions, see**: [vdbench-scripts/README.md](vdbench-scripts/README.md)

**Additional tools in this container**:
- **fio** - Flexible I/O workload simulator
- Various other file system testing utilities

## Directory Structure

```
File-Tests/
├── IOR-scripts/
│   ├── IOR-MDtest-full.ini
│   ├── drop-cache-Cloud_Platform.sh
│   ├── io500-mpi-coordinate-gemini-S3_Provider.sh
│   └── io500-mpi-coordinate-gemini-Cloud_Platform.sh
├── vdbench-scripts/
│   ├── Resnet50/
│   │   ├── resnet50-1hosts_parmfile.txt
│   │   ├── resnet50-4hosts_parmfile.txt
│   │   └── resnet50-8hosts_parmfile.txt
│   └── Unet3d/
│       ├── unet3d-1hosts_parmfile.txt
│       ├── unet3d-4hosts_parmfile.txt
│       └── unet3d-8hosts_parmfile.txt
└── [README files documenting each tool]
```

## Test Configurations

### IO500 Configurations

Located in `IOR-scripts/`:

- **IOR-MDtest-full.ini** - IO500 configuration for comprehensive benchmark testing
- **io500-mpi-coordinate-gemini-S3_Provider.sh** - IO500 benchmark coordination for S3_Provider infrastructure
- **io500-mpi-coordinate-gemini-Cloud_Platform.sh** - IO500 benchmark coordination for Cloud_Platform
- **drop-cache-Cloud_Platform.sh** - Cache dropping utility for consistent Cloud_Platform testing (critical for accurate results)

### vdbench Configurations

Located in `vdbench-scripts/`, organized by AI/ML workload:

#### ResNet50 Workload
Simulates ResNet50 deep learning training I/O patterns:
- `resnet50-1hosts_parmfile.txt` - Single-host configuration
- `resnet50-4hosts_parmfile.txt` - 4-host distributed configuration
- `resnet50-8hosts_parmfile.txt` - 8-host distributed configuration

#### UNet3D Workload
Simulates 3D UNet segmentation model training I/O patterns:
- `unet3d-1hosts_parmfile.txt` - Single-host configuration
- `unet3d-4hosts_parmfile.txt` - 4-host distributed configuration
- `unet3d-8hosts_parmfile.txt` - 8-host distributed configuration

## Container Images

| Tool | Container | Build Instructions |
|------|-----------|-------------------|
| IOR | `io500:latest` | [dockerfiles/io500/](../dockerfiles/io500/) |
| vdbench | `file-tests:latest` | [dockerfiles/file-tests/README.md](../dockerfiles/file-tests/README.md) |

⚠️ **IMPORTANT**: You must build these containers locally. See the build instructions above.

### Building Containers

**For io500:latest**:
```bash
cd dockerfiles/io500
docker build -t io500:latest .
```

**For file-tests:latest** (includes vdbench + FIO):
```bash
# REQUIRED: Download vdbench50407.zip from Oracle first
# See: dockerfiles/file-tests/README.md for licensing requirements
cd dockerfiles/file-tests
# Download vdbench50407.zip from https://www.oracle.com/downloads/server-storage/vdbench-downloads.html
# Place it in this directory, then:
docker build -t file-tests:latest .
```

## Running Tests

### Prerequisites
- Docker or container runtime
- **Built containers** (see "Building Containers" above)
- File system mount point
- Network connectivity (for distributed tests)
- Sufficient disk space for test data

### IO500 Test Execution

```bash
# For distributed multi-host benchmarking with MPI coordination
# See: IOR-scripts/README.md for complete setup and execution
bash io500-mpi-coordinate-gemini-S3_Provider.sh    # Or Cloud_Platform variant
bash drop-cache-Cloud_Platform.sh                     # Drop caches at the right timing

# For basic single-host testing
docker run -it -v /mount/point:/testdir \
  io500:latest \
  bash -c "cd /testdir && mpiexec -np 2 ./io500 IOR-MDtest-full.ini"
```

**For comprehensive IO500 setup, infrastructure requirements, cloud platform coordination, and execution guidance, see [IOR-scripts/README.md](IOR-scripts/README.md)**

### vdbench Test Example

vdbench can be run in two modes using provided shell scripts:

#### Mode 1: Agent (Listening) Mode
Start vdbench as an agent listening for commands from a coordinator:

```bash
# start_vdb-agent.sh
docker run --rm -v /mnt/storage:/mnt/storage --net=host -it file-tests \
  "/opt/vdbench/vdbench" "rsh"
```

**Use for**: Distributed multi-host testing with centralized coordination

#### Mode 2: Interactive Container
Start the container interactively and run vdbench manually:

```bash
# start_vdb.sh
docker run -v /mnt/storage:/mnt/storage --net=host -it file-tests

# Inside container:
cd /opt/vdbench
./vdbench -f <config_file> -o <output_dir>
```

**Use for**: Single-host tests and manual execution

#### Quick Example

```bash
# Build the container first (see "Building Containers" section above)

# Option 1: Direct execution
docker run -it -v /mount/point:/testdir \
  file-tests:latest \
  vdbench -f resnet50-1hosts_parmfile.txt -o /testdir/output

# Option 2: Using interactive mode
docker run -it -v /mnt/storage:/mnt/storage --net=host file-tests
# Then inside:
cd /opt/vdbench && ./vdbench -f resnet50-1hosts_parmfile.txt -o /mnt/storage/output
```

**Command-line options**:
- `-f <config_file>` - Parameter file path (required)
- `-o <output_dir>` - Output directory for results (required)

## Detailed Documentation

For detailed information on each tool:
- **IO500**: See [IOR-scripts/README.md](IOR-scripts/README.md) for infrastructure setup, distributed coordination, and execution guidance
- **vdbench**: See [vdbench-scripts/README.md](vdbench-scripts/README.md) for configuration and execution guides

### External Documentation Resources

- **IO500 Benchmark**: [io500.io](https://io500.io) - Official results, specifications for ior-easy/ior-hard/mdtest-easy/mdtest-hard tiers
- **IOR GitHub**: [github.com/hpc/ior](https://github.com/hpc/ior) - Source code, wiki, and configuration examples
- **vdbench Guide**: Oracle/Delphix vdbench user guide (v50407) - Complete parameter and tuning reference

## Container Images

| Tool | Container | Build Instructions |
|------|-----------|-------------------|
| IOR | `io500:latest` | [dockerfiles/io500/](../dockerfiles/io500/) |
| vdbench | `file-tests:latest` | [dockerfiles/file-tests/README.md](../dockerfiles/file-tests/README.md) |

## Workload Patterns

### ResNet50 (MLCommons Storage)
Configurations implementing MLCommons Storage ResNet50 benchmark:
- Replicates CNN training I/O patterns from MLCommons specification
- 1-client, 4-client, 8-client configurations
- Sequential and random access patterns
- Multi-client distributed configurations available

### UNet3D (MLCommons Storage)
Configurations implementing MLCommons Storage UNet3D benchmark:
- Replicates volumetric segmentation I/O patterns from MLCommons specification
- 1-client, 4-client, 8-client configurations
- 3D volume chunk access patterns
- Medical imaging and scientific computing workload patterns

## Troubleshooting

- **Mount issues**: Ensure file system is mounted and accessible from container
- **Permission errors**: Check container user permissions for target mount point
- **Timeout errors**: Increase test duration in configuration for slow storage
- **Distributed coordination**: Verify network connectivity between test hosts

## Related Resources

- Main repository: [../README.md](../README.md)
- Container build instructions: [../dockerfiles/README.md](../dockerfiles/README.md)
