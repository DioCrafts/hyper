# BufList::remaining() O(1) Optimization — Benchmark Results

Benchmarks run with `cargo +nightly bench --features "nightly,full" --lib -- buflist` on both the baseline (iterative O(n)) and optimized (cached O(1)) implementations.

## `remaining()` call only

| Buffers | Before (O(n)) | After (O(1)) | Speedup |
|---------|---------------|--------------|---------|
| 1       | 0.99 ns/iter  | 0.32 ns/iter | **3x**  |
| 4       | 1.14 ns/iter  | 0.27 ns/iter | **4x**  |
| 16      | 2.48 ns/iter  | 0.40 ns/iter | **6x**  |
| 128     | 20.54 ns/iter | 0.35 ns/iter | **59x** |
| 1024    | 302.59 ns/iter| 0.33 ns/iter | **917x**|

`remaining()` is now constant-time (~0.3 ns) regardless of buffer count.

## `push()` + `remaining()` cycle (simulates real write loop)

| Buffers | Before          | After          | Improvement |
|---------|-----------------|----------------|-------------|
| 16      | 148.50 ns/iter  | 118.62 ns/iter | **~20%**    |
| 128     | 2,032.20 ns/iter| 468.55 ns/iter | **~77%**    |

With hyper's current `MAX_BUF_LIST_BUFFERS = 16`, the push+remaining cycle sees a ~20% speedup, growing to ~77% at higher buffer counts.

## How to reproduce

```bash
cargo +nightly bench --features "nightly,full" --lib -- buflist
```

## Raw output

### Before (O(n) iteration)

```
test common::buf::bench::buflist_remaining_1_buf        ... bench:           0.99 ns/iter (+/- 0.15)
test common::buf::bench::buflist_remaining_4_bufs       ... bench:           1.14 ns/iter (+/- 0.19)
test common::buf::bench::buflist_remaining_16_bufs      ... bench:           2.48 ns/iter (+/- 0.35)
test common::buf::bench::buflist_remaining_128_bufs     ... bench:          20.54 ns/iter (+/- 2.53)
test common::buf::bench::buflist_remaining_1024_bufs    ... bench:         302.59 ns/iter (+/- 19.95)
test common::buf::bench::buflist_push_and_remaining_16  ... bench:         148.50 ns/iter (+/- 25.87)
test common::buf::bench::buflist_push_and_remaining_128 ... bench:       2,032.20 ns/iter (+/- 285.50)
```

### After (O(1) cached field)

```
test common::buf::bench::buflist_remaining_1_buf        ... bench:           0.32 ns/iter (+/- 0.05)
test common::buf::bench::buflist_remaining_4_bufs       ... bench:           0.27 ns/iter (+/- 0.05)
test common::buf::bench::buflist_remaining_16_bufs      ... bench:           0.40 ns/iter (+/- 0.15)
test common::buf::bench::buflist_remaining_128_bufs     ... bench:           0.35 ns/iter (+/- 0.08)
test common::buf::bench::buflist_remaining_1024_bufs    ... bench:           0.33 ns/iter (+/- 0.13)
test common::buf::bench::buflist_push_and_remaining_16  ... bench:         118.62 ns/iter (+/- 19.20)
test common::buf::bench::buflist_push_and_remaining_128 ... bench:         468.55 ns/iter (+/- 76.48)
```
