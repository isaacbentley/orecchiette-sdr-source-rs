# Contributing to orecchiette-sdr-source-rs

First off, thank you for considering contributing! This crate defines
the shared `SdrSource` trait and types every backend crate
(`orecchiette-sdr-usrp-rs`, `-hackrf-rs`, `-pluto-rs`, `-file-rs`,
`sdr-aaronia-rs`) implements, so a change here ripples across the
whole family — please open an issue to discuss any change to the
trait signature or `IqPacket`/`SourceConfig` shapes before sending a
PR.

## Quick Start

```bash
git clone https://github.com/isaacbentley/orecchiette-sdr-source-rs.git
cd orecchiette-sdr-source-rs

# Run the standard validation suite
cargo test
cargo clippy --all-features --all-targets -- -D warnings
cargo fmt --all --check
cargo deny check
```

## Code Style

We use standard `rustfmt` defaults. Please run `cargo fmt --all` before pushing.

Clippy is run with `-D warnings` in CI. If a lint is genuinely wrong for the situation, allow it with a `// ALLOW:` justification comment explaining why.

## Pull Requests

- **Commit messages:** Describe *why* the change is needed and *what* it changes.
- **Breaking changes:** Since every backend crate depends on this one, a breaking change here forces a coordinated version bump across the family — call this out explicitly in the PR description.

## License

By contributing, you agree your contributions will be licensed under GPL-3.0-or-later, the same as the rest of the project.
