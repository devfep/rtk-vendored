# rtk, vendored for offline builds

Upstream: https://github.com/rtk-ai/rtk at tag v0.50.0 (commit 1d87b8e), Apache-2.0 (see LICENSE).
This copy adds `vendor/` (every dependency from Cargo.lock, produced by `cargo vendor --locked`)
and `.cargo/config.toml` (source replacement), so it builds with no registry access:

    cargo build --release --offline --locked
    install -m 755 target/release/rtk ~/ebs/tools/bin/rtk

Requires rustc 1.91 or newer (Cargo.toml `rust-version`). Nothing else is modified.
