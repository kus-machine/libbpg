# Quick Reference: Build BPG on Apple Silicon

## TL;DR - Fast Track (30 seconds)

```bash
# 1. Install dependencies
brew install libpng libjpeg xz cmake

# 2. Edit Makefile: uncomment CONFIG_APPLE=y
nano Makefile

# 3. Build
make -j $(sysctl -n hw.ncpu) bpgenc bpgdec

# 4. Install
sudo install -m 755 bpgenc bpgdec /usr/local/bin/

# 5. Verify
bpgenc -h
```

## Common Commands

```bash
# Encode (quality 25 = good for web)
bpgenc -q 25 image.png -o image.bpg

# Decode
bpgdec image.bpg -o decoded.png

# Show help
bpgenc -h
bpgdec -h
```

## If You Get Errors

### CMake policy errors
→ Already fixed in provided x265/source/CMakeLists.txt

### "png.h not found"
→ Make sure CONFIG_APPLE=y is uncommented in Makefile

### "Library not found: png"
→ Check Makefile has the Homebrew paths configured

### strcasecmp error
→ Add `#include <strings.h>` to bpgdec.c

## Files Modified

- `x265/source/CMakeLists.txt` - CMake 3.5+ compatible, ARM64 detection
- `Makefile` - CONFIG_APPLE enabled, Homebrew paths added
- `bpgdec.c` - strings.h include added
- `README` - New section 2.3 added
- `BUILD_APPLE_SILICON.md` - Comprehensive guide (new file)

## Performance Tips

```bash
# Use all CPU cores during build
make -j $(sysctl -n hw.ncpu) bpgenc bpgdec

# Strip binary to save space (optional)
strip /usr/local/bin/bpgenc
strip /usr/local/bin/bpgdec
```

## Documentation Files

- **README** - Main guide (section 2.3)
- **BUILD_APPLE_SILICON.md** - Detailed instructions
- **APPLE_SILICON_QUICK_REFERENCE.md** - This file
