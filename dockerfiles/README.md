# Container Images for File and Object Storage Testing

This directory contains Dockerfiles for building the container images used in this repository.

## ⚠️ IMPORTANT: vdbench Licensing

**vdbench must be downloaded from Oracle before building the file-tests container.**

Due to Oracle's licensing restrictions, we cannot include or redistribute vdbench. You must:
1. Download `vdbench50407.zip` from Oracle: https://www.oracle.com/downloads/server-storage/vdbench-downloads.html
2. Accept Oracle's license agreement
3. Place `vdbench50407.zip` in `dockerfiles/file-tests/` directory before building

See [file-tests/README.md](file-tests/README.md) for detailed instructions.

## Available Containers

### 1. file-tests (vdbench + FIO)
**Purpose**: File system testing with vdbench and FIO tools  
**Location**: `dockerfiles/file-tests/`  
**Replaces**: `file-tests:latest`

**Prerequisites**:
- Download vdbench50407.zip from Oracle (see warning above)
- Place in `dockerfiles/file-tests/` directory

**Build:**
```bash
# First: Download vdbench50407.zip from Oracle and place in dockerfiles/file-tests/
cd dockerfiles/file-tests
ls -lh vdbench50407.zip  # Verify file is present
docker build -t file-tests:latest .
```

**Detailed instructions**: See [file-tests/README.md](file-tests/README.md)

**What's Included:**
- vdbench 50407 - File system workload generator (user-provided from Oracle)
- FIO - Flexible I/O tester (from Ubuntu repos)
- Supporting tools and utilities

### 2. io500 (IOR + MPI)
**Purpose**: Distributed parallel file I/O testing  
**Location**: `dockerfiles/io500/`  
**Replaces**: `io500:latest`

**Build:**
```bash
cd dockerfiles/io500
docker build -t io500:latest .
```

**What's Included:**
- IOR - Interleaved Or Random (parallel I/O benchmark)
- MPI support for distributed testing
- io500 benchmark suite

## Quick Build All

```bash
# Build file-tests
cd dockerfiles/file-tests && docker build -t file-tests:latest . && cd ../..

# Build io500
cd dockerfiles/io500 && docker build -t io500:latest . && cd ../..
```

**For object storage benchmarking**: Install sai3-bench natively from https://github.com/russfellows/sai3-bench

## Verifying Builds

```bash
# Check built images
docker images | grep -E "file-tests|io500"

# Test file-tests container
docker run --rm file-tests:latest vdbench -t

# Test io500 container  
docker run --rm io500:latest ior --version
```

## Usage

After building, use the local image names in all test scripts:
- `file-tests:latest` for vdbench and FIO
- `io500:latest` for IOR and IO500 benchmarks  
- Native sai3-bench installation for object storage testing (install from https://github.com/russfellows/sai3-bench)

All README files and scripts in this repository have been updated to reference these local builds.
