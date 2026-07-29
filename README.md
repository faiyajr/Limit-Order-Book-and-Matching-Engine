# Limit Order Book & Matching Engine

A high-performance exchange core in modern C++, built for deterministic
price-time priority matching under realistic order flow.

## Stack

| Layer | Tooling |
|---|---|
| Language | C++20 |
| Build | CMake 3.20+, Ninja |
| Test | GoogleTest |
| Benchmark | Google Benchmark, `perf` |
| Platform | Linux x86-64 |

## Design Targets

- **Throughput:** 100K+ commands/sec sustained on a single core
- **Latency:** sub-microsecond mean on add / cancel / match
- **Replay:** 1M+ simulated events, bit-identical run to run

## Core Features

- Price-time priority (FIFO) matching
- Limit and market orders; Day, IOC, and FOK time-in-force
- Cancels and modifies with correct queue-position semantics
- L2 depth snapshots
- Deterministic event log with a checksum for replay regression

## Performance Work

- Flat price-level array over a fixed tick band, O(1) level lookup
- Intrusive doubly-linked FIFO per level, no allocation on add or cancel
- Slab pool with a LIFO free list for `Order` nodes
- `Order` packed into exactly one 64-byte cache line
- Benchmarked against a naive `std::map<Price, std::list<Order>>` baseline

## Layout

```
include/lob/     public headers (types, order, pool, price_level,
                 order_book, matching_engine, events, event_log,
                 book_view, wire)
src/             implementations
apps/            lob_replay (throughput harness), lob_cli (interactive)
tests/           unit tests + differential test vs a slow reference book
bench/           Google Benchmark suites, including the naive baseline
tools/           gen_events.py, synthetic order flow generator
docs/            design.md
```

## Build

```bash
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
ctest --test-dir build --output-on-failure
```

## Replay

```bash
python3 tools/gen_events.py --count 1000000 --out data/flow_1m.bin
./build/lob_replay data/flow_1m.bin 9000 11000
```

## Status

Core engine, tests, and replay harness are in place. Optimization work is
tracked in `docs/design.md`; benchmark numbers in this README are targets, not
measurements, until the bench suite is run on the reference machine.
