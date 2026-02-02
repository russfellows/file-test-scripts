# file-tests Container (vdbench + FIO)

This container provides file system testing tools including vdbench and FIO.

## ⚠️ IMPORTANT: vdbench Licensing Requirements

**vdbench MUST be downloaded directly from Oracle** due to their licensing restrictions. We cannot provide or redistribute the vdbench binary.

### Download vdbench from Oracle

1. Visit Oracle's vdbench download page:
   - **URL**: https://www.oracle.com/downloads/server-storage/vdbench-downloads.html
   - **Required version**: vdbench50407.zip

2. Accept Oracle's license agreement

3. Download `vdbench50407.zip`

4. **Place the downloaded file in this directory** (`dockerfiles/file-tests/`) alongside the Dockerfile:
   ```
   dockerfiles/file-tests/
   ├── Dockerfile
   └── vdbench50407.zip  ← Downloaded from Oracle
   ```

## Building the Container

### Prerequisites
- Docker installed
- `vdbench50407.zip` downloaded from Oracle (see above)
- `vdbench50407.zip` placed in `dockerfiles/file-tests/` directory

### Build Command

```bash
# Navigate to the file-tests directory
cd dockerfiles/file-tests

# Verify vdbench50407.zip is present
ls -lh vdbench50407.zip

# Build the container
docker build -t file-tests:latest .
```

**Build time**: ~2-3 minutes (depending on network speed)

## What's Included

### Tools
- **vdbench** (version 50407) - File system workload generator
  - Realistic file system workload simulation
  - Multi-threaded operation
  - Complex workload definitions
  - Detailed performance statistics

- **FIO** (Flexible I/O Tester) - Latest from Ubuntu 24.04 repos
  - Flexible I/O workload simulator
  - Multiple I/O engines (sync, libaio, io_uring, etc.)
  - Comprehensive performance testing

### Base System
- Ubuntu 24.04 LTS
- OpenJDK 8 JRE (required by vdbench)
- Common utilities: vim, git, curl, unzip
- Network tools: ping, netstat, dig

## Verifying the Build

```bash
# Check the image was created
docker images | grep file-tests

# Test vdbench is accessible
docker run --rm file-tests:latest vdbench -t

# Test FIO is accessible  
docker run --rm file-tests:latest fio --version

# Interactive shell
docker run -it --rm file-tests:latest
```

## Usage Examples

### Basic vdbench Test
```bash
docker run --rm \
  -v /path/to/configs:/configs \
  -v /path/to/testdir:/testdir \
  file-tests:latest \
  vdbench -f /configs/test.conf -o /testdir/results
```

### Interactive Mode
```bash
docker run -it --rm \
  -v /path/to/testdir:/testdir \
  file-tests:latest
  
# Inside container:
cd /opt/vdbench
./vdbench -f mytest.conf -o /testdir/output
```

### FIO Test
```bash
docker run --rm \
  -v /path/to/testdir:/testdir \
  file-tests:latest \
  fio --name=seqread --rw=read --size=1G --directory=/testdir
```

## Troubleshooting

### Build fails with "COPY failed: file not found"
**Cause**: `vdbench50407.zip` is not in the `dockerfiles/file-tests/` directory

**Solution**: 
1. Download vdbench50407.zip from Oracle
2. Place it in `dockerfiles/file-tests/` alongside the Dockerfile
3. Rebuild

### vdbench fails to run
**Cause**: Java runtime issues or incorrect vdbench version

**Solution**:
1. Verify you downloaded vdbench50407 (not a different version)
2. Check Java is available: `docker run --rm file-tests:latest java -version`
3. Should see "openjdk version 1.8.0"

## File Structure After Build

```
Container filesystem:
/opt/vdbench/          # vdbench installation
  ├── vdbench          # Main executable
  ├── *.class          # Java class files
  └── ...
  
/usr/bin/fio           # FIO executable
/data/                 # Default working directory
```

## Legal Notice

**vdbench** is proprietary software owned by Oracle Corporation. Users must:
- Download vdbench directly from Oracle
- Accept Oracle's license agreement
- Comply with all Oracle licensing terms

This Dockerfile does not include or redistribute vdbench. It only provides instructions for building a container with your legally obtained copy of vdbench.

**FIO** is open source software (GPL v2) included from Ubuntu repositories.
