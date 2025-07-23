# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`ecloop` is a high-performance, CPU-optimized tool for computing public keys on the secp256k1 elliptic curve. It's designed for searching Bitcoin addresses and solving Bitcoin puzzles using various cryptographic optimizations including fixed 256-bit modular arithmetic, group inversion, precomputed tables, and SIMD-accelerated hashing.

## Build System

The project uses a Makefile with clang as the default compiler:

- **Build**: `make build` or `make` (compiles main.c with lib/ files)
- **Clean**: `make clean` (removes build artifacts)
- **Format**: `make fmt` (formats C code using clang-format)
- **Benchmark**: `make bench` (runs performance benchmarks)

### Test/Verification Commands
- `make add` - Should find 9 keys (sequential search test)
- `make mul` - Should find 1080 keys (multiplication search test)
- `make verify` - Runs multiplication verification
- `make blf` - Tests bloom filter functionality

### Bitcoin Puzzle Shortcuts
Preconfigured puzzle ranges available via `make [puzzle_number]`:
- `make 28`, `make 32`, `make 36` (smaller puzzles)
- `make 71`, `make 73-79` (larger puzzles)
- Results saved to `found_[N].txt`

## Architecture

### Core Structure
- **main.c**: Entry point, command parsing, threading coordination
- **lib/**: Core functionality modules
  - `ecc.c`: Elliptic curve operations, field arithmetic, point multiplication
  - `addr.c`: Address generation and hashing (SHA-256, RIPEMD-160)
  - `utils.c`: Utilities, terminal handling, file I/O
  - `bench.c`: Performance benchmarking
  - `compat.c`: Platform compatibility layer
  - `sha256.c`/`rmd160.c`/`rmd160s.c`: Optimized hash implementations

### Key Features
- **Commands**: `add` (sequential), `mul` (from stdin), `rnd` (random search)
- **Multi-threading**: pthread-based parallel processing
- **Optimizations**: SIMD hashing (AVX2/NEON), endomorphism support, precomputed tables
- **Filter Support**: Bloom filters (.blf) and hash lists for target searching
- **Address Types**: Compressed (33-byte) and uncompressed (65-byte) public keys

### Data Flow
1. Parse command line arguments and initialize context
2. Load target hashes/bloom filter if specified
3. Spawn worker threads for parallel processing
4. Each thread performs elliptic curve operations and address generation
5. Results written to output file or stdout

## Development

### Compiler Options
- Default: `clang -O3 -ffast-math -Wall -Wextra`
- x86_64: Adds `-march=native -pthread -lpthread`
- Alternative: Use `CC=gcc` for different compiler

### Testing
- CI runs on Ubuntu and macOS
- Basic smoke test: `./ecloop add -f data/btc-puzzles-hash -r 8000:ffff -q -o /dev/null`
- Performance verification via `make add` and `make mul`

### File Structure
- **data/**: Test data files (hashes, private keys, addresses)
- **lib/**: Core C implementation files
- **main.c**: Application entry point
- **Makefile**: Build and test automation