# orecchiette-sdr-source-rs

[![Crates.io](https://img.shields.io/crates/v/orecchiette-sdr-source-rs.svg)](https://crates.io/crates/orecchiette-sdr-source-rs)
[![Docs.rs](https://docs.rs/orecchiette-sdr-source-rs/badge.svg)](https://docs.rs/orecchiette-sdr-source-rs)
[![CI](https://github.com/isaacbentley/orecchiette-sdr-source-rs/actions/workflows/ci.yml/badge.svg)](https://github.com/isaacbentley/orecchiette-sdr-source-rs/actions/workflows/ci.yml)
[![License: GPL-3.0-or-later](https://img.shields.io/github/license/isaacbentley/orecchiette-sdr-source-rs.svg)](https://choosealicense.com/licenses/gpl-3.0/)

Common abstraction for SDR sources used by SDR detection applications.

`orecchiette-sdr-source-rs` defines a single unifying trait (`SdrSource`) and a shared set of primitives (`IqPacket`, `SourceConfig`, `DwellAdvice`, `DwellController`) that all SDR backends implement. This guarantees that a detection orchestrator can process data identically regardless of whether it originated from a USRP, HackRF, ADALM-Pluto, an Aaronia Spectran, or an offline file.

## Overview

One trait, a few shared types, no third-party SDR dependencies. Every backend — [USRP](https://github.com/isaacbentley/orecchiette-sdr-usrp-rs), [HackRF](https://github.com/isaacbentley/orecchiette-sdr-hackrf-rs), [ADALM-Pluto](https://github.com/isaacbentley/orecchiette-sdr-pluto-rs), [file / SigMF](https://github.com/isaacbentley/orecchiette-sdr-file-rs), [Aaronia HTTP / SDK / RTSA](https://github.com/isaacbentley/sdr-aaronia-rs) — implements `SdrSource` and emits the same `IqPacket` shape so the orchestrator can dispatch detection uniformly.

## Installation

Add the following to your `Cargo.toml`:

```toml
[dependencies]
orecchiette-sdr-source-rs = "0.1.0"
```

## The Trait

```rust,ignore
pub trait SdrSource: Send {
    fn start(
        self: Box<Self>,
        config: SourceConfig,
        advice: Arc<dyn DwellAdvice>,
    ) -> Result<SdrHandle, SdrError>;
}
```

`start` consumes the builder (`Box<Self>`), spawns whatever capture
threads the backend needs, and returns an `SdrHandle`:

```rust,ignore
pub struct SdrHandle {
    pub receiver: Receiver<IqPacket>,
    pub stop: Box<dyn FnOnce() + Send>,
}
```

`receiver` is a `crossbeam::channel::Receiver<IqPacket>`. Every
backend, regardless of how it got the data, emits the same shape:

```rust,ignore
pub struct IqPacket {
    pub samples: PooledIqBuffer,  // Deref<Target = [Complex32]>
    pub center_frequency_hz: f64,
    pub sample_rate_hz: f32,
    pub overrun: bool,
}
```

`samples` is a `PooledIqBuffer` — a smart wrapper around `Vec<Complex32>`
that automatically returns the vector to a recycling pool when dropped.
It implements `Deref<Target = [Complex32]>`, so consumers treat it
exactly like a `&[Complex32]` slice.

`sample_rate_hz` is `f32` rather than `f64` because the downstream
DSP (FFT, polyphase resampler) is single-precision — keeping the
rate at the same width avoids silent narrowings at the boundary.

`stop` is the shutdown hook; calling it (or dropping the handle and
letting `receiver` disconnect on the next iteration) winds the capture
thread down.

## Dependencies

Tiny on purpose:

```toml
crossbeam     = "0.8"      # channel
num-complex   = "0.4"      # Complex32
thiserror     = "2.0"      # SdrError
anyhow        = "1.0"      # SdrError::Backend pass-through
```

No SDR hardware dependencies — those live in the implementation crates.

## MSRV & Semver Policy

- **MSRV:** This crate does not maintain an explicit Minimum Supported Rust Version (MSRV) policy and tracks the latest `stable` compiler.
- **Semver:** This crate follows semantic versioning. While in `0.x.y`, breaking API changes will result in a minor version bump (e.g. `0.1.x` to `0.2.0`).

## Testing & Contributing

`tests/contract.rs` exercises a minimal mock implementation to verify the basic shape: `start` returns a non-empty receiver, dropping the handle releases the receiver, `stop()` is idempotent.

Please see [CONTRIBUTING.md](CONTRIBUTING.md) for detailed instructions on running the test suite and formatting your code before submitting a Pull Request.

## Documentation

- [Architecture & Design](DESIGN.md) — internal architecture and execution flow.

## License

This project is licensed under the GNU General Public License v3.0 or later (GPL-3.0-or-later) - see the [LICENSE](LICENSE) file for details.
