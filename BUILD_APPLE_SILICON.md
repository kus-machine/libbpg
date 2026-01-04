# Building BPG for Apple Silicon (ARM64)

This guide provides step-by-step instructions to build and install `bpgenc` and `bpgdec` on Apple Silicon Macs (M1, M2, M3, etc.).

## Prerequisites

- macOS with Apple Silicon (M1/M2/M3 or newer)
- Xcode Command Line Tools
- Homebrew package manager

## Quick Start

### 1. Install Dependencies

```bash
brew install libpng libjpeg xz cmake
```

### 2. Clone or Prepare the Repository

```bash
cd /path/to/libbpg
```

### 3. Fix x265 CMake Compatibility

The x265 CMakeLists.txt uses deprecated CMake policies. Apply these changes to `x265/source/CMakeLists.txt`:

**Change 1: Reorder and update cmake_minimum_required**

Replace the beginning of the file (before `project()` declaration):

```cmake
# Before:
if(NOT CMAKE_BUILD_TYPE)
    # ...
endif()
message(STATUS "cmake version ${CMAKE_VERSION}")
if(POLICY CMP0025)
    cmake_policy(SET CMP0025 OLD)
endif()
# ... more policy settings ...
project (x265)
cmake_minimum_required (VERSION 2.8.8)

# After:
cmake_minimum_required (VERSION 3.5)
project (x265)

if(NOT CMAKE_BUILD_TYPE)
    # ... rest of settings
endif()
message(STATUS "cmake version ${CMAKE_VERSION}")
if(POLICY CMP0042)
    cmake_policy(SET CMP0042 NEW)
endif()
```

**Change 2: Add ARM64 architecture detection**

Find the processor detection section (around line 55-70) and add ARM64 support:

```cmake
elseif(${SYSPROC} STREQUAL "armv6l")
    message(STATUS "Detected ARM target processor")
    set(ARM 1)
    add_definitions(-DX265_ARCH_ARM=1 -DHAVE_ARMV6=1)
elseif(${SYSPROC} STREQUAL "arm64" OR ${SYSPROC} STREQUAL "aarch64")
    message(STATUS "Detected ARM64 target processor")
    set(ARM 1)
    add_definitions(-DX265_ARCH_ARM=1)
else()
    # ... rest of else clause
endif()
```

### 4. Configure Makefile for macOS

Edit the `Makefile` in the root directory:

**Change 1: Enable CONFIG_APPLE**

```makefile
# Change from:
#CONFIG_APPLE=y

# To:
CONFIG_APPLE=y
```

**Change 2: Add Homebrew paths to CFLAGS**

Add these lines after `CFLAGS+=-I.`:

```makefile
CFLAGS+=-I/opt/homebrew/opt/libpng/include/libpng16
CFLAGS+=-I/opt/homebrew/opt/jpeg/include
CFLAGS+=-I/opt/homebrew/opt/xz/include
```

**Change 3: Add Homebrew paths to LDFLAGS**

After the `LDFLAGS=-g` line, add:

```makefile
ifdef CONFIG_APPLE
LDFLAGS+=-L/opt/homebrew/opt/libpng/lib
LDFLAGS+=-L/opt/homebrew/opt/jpeg/lib
LDFLAGS+=-L/opt/homebrew/opt/xz/lib
LDFLAGS+=-Wl,-dead_strip
endif
```

### 5. Fix bpgdec Source (Optional but Recommended)

Add `#include <strings.h>` to the top of `bpgdec.c` to ensure `strcasecmp` is declared.

### 6. Build

```bash
# Clean previous builds (if any)
make clean

# Build both encoder and decoder
make bpgenc bpgdec

# Or use parallel compilation for speed
make -j $(sysctl -n hw.ncpu) bpgenc bpgdec
```

### 7. Install to System

```bash
sudo install -m 755 bpgenc /usr/local/bin/
sudo install -m 755 bpgdec /usr/local/bin/
```

### 8. Verify Installation

```bash
which bpgenc
bpgenc -h

which bpgdec
bpgdec -h
```

## Usage Examples

### Encode an image to BPG

```bash
# Basic encoding (quality 25 is good for web)
bpgenc -q 25 input.png -o output.bpg

# Faster encoding (quality 30, speed level 1)
bpgenc -q 30 -m 1 input.png -o output.bpg

# Higher quality (quality 20)
bpgenc -q 20 input.png -o output.bpg

# 10-bit depth for better compression
bpgenc -q 25 -b 10 input.png -o output.bpg

# Lossless mode
bpgenc -lossless input.png -o output.bpg
```

### Decode BPG to image

```bash
# Decode to PNG
bpgdec output.bpg -o decoded.png

# Decode to PPM format
bpgdec output.bpg -o decoded.ppm
```

## Architecture Details

- **Architecture**: ARM64 (native for Apple Silicon)
- **x265 Support**: 8-bit, 10-bit, and 12-bit encoding
- **Compilation Time**: ~5-10 minutes depending on Mac specs
- **Binary Size**: ~15-20 MB (can be stripped to ~5 MB with `strip`)

## Troubleshooting

### CMake version errors
If you get CMake policy errors, ensure you have CMake 3.5 or newer:
```bash
cmake --version
brew upgrade cmake
```

### Library not found errors
Verify dependencies are installed:
```bash
brew list libpng libjpeg xz
```

### Permission denied during install
Use `sudo` for the install commands:
```bash
sudo install -m 755 bpgenc /usr/local/bin/
```

## Performance Notes

- Assembly optimizations are disabled for ARM64 (not necessary for performance)
- The x265 encoder is highly optimized in native ARM code
- Compilation takes advantage of all available CPU cores with `-j` flag
- The resulting binaries run natively without any emulation or translation

## Additional Resources

- BPG Format Specification: `doc/bpg_spec.txt`
- x265 Encoder Documentation: https://x265.readthedocs.io/
- Homebrew: https://brew.sh
