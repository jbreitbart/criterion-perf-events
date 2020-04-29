# Criterion-perf-events

This is a measurement plugin for [Criterion.rs](https://github.com/bheisler/criterion.rs) to measure events of the Linux perf interface.

## Supported Events

Criterion-perf-events uses the [`perfcnt`](https://gz.github.io/rust-perfcnt/perfcnt/index.html) crate and supports events provided by this crate. If you are interested in more details, please take a look at the events listed here:

* [Hardware events](https://gz.github.io/rust-perfcnt/perfcnt/linux/enum.HardwareEventType.html)
* [Software events](https://gz.github.io/rust-perfcnt/perfcnt/linux/enum.SoftwareEventType.html)
* [Perf events](https://gz.github.io/rust-perfcnt/perfcnt/linux/enum.Event.html)
* [Raw Intel events](https://gz.github.io/rust-perfcnt/x86/perfcnt/intel/description/struct.IntelPerformanceCounterDescription.html)

## Example

The following code shows on how to count last level cache misses.

```rust
extern crate criterion_perf_events;
extern crate perfcnt;

fn fibonacci_slow(_: usize) {}
fn fibonacci_fast(_: usize) {}

use criterion::{criterion_group, criterion_main, BenchmarkId, Criterion};
use criterion_perf_events::Perf;
use perfcnt::linux::HardwareEventType as Hardware;
use perfcnt::linux::PerfCounterBuilderLinux as Builder;

fn bench(c: &mut Criterion<Perf>) {
    let mut group = c.benchmark_group("fibonacci");

    for i in 0..20 {
        group.bench_function(BenchmarkId::new("slow", i), |b| b.iter(|| fibonacci_slow(i)));
        group.bench_function(BenchmarkId::new("fast", i), |b| b.iter(|| fibonacci_fast(i)));
    }

    group.finish()
}

criterion_group!(
    name = my_bench;
    config = Criterion::default().with_measurement(Perf::new(Builder::from_hardware_event(Hardware::CacheMisses)));
    targets = bench
);
criterion_main!(my_bench);
```
