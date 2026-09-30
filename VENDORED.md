# rtk, vendored for offline builds

Upstream: https://github.com/rtk-ai/rtk at tag v0.50.0 (commit 1d87b8e), Apache-2.0 (see LICENSE).
This copy adds `vendor/` (every dependency from Cargo.lock, produced by `cargo vendor --locked`)
and `.cargo/config.toml` (source replacement), so it builds with no registry access.
Requires rustc 1.91 or newer (Cargo.toml `rust-version`). Nothing else is modified.

## Setting it up in a workspace

Everything below runs inside the workspace terminal and needs only git access to this repo.

1. Rust, once per workspace (the platform's own installer; accept the defaults, then open a new
   shell):

       rustup-init
       cargo --version        # must print 1.91 or newer

2. Clone this repo somewhere on the persistent volume and build it offline. The build takes
   several minutes on two CPUs and happens once:

       git clone <this repo's URL> ~/ebs/tools/src/rtk-vendored
       cd ~/ebs/tools/src/rtk-vendored
       cargo build --release --offline --locked

3. Install the binary on the persistent volume and put it on your PATH:

       mkdir -p ~/ebs/tools/bin
       install -m 755 target/release/rtk ~/ebs/tools/bin/rtk
       echo 'export PATH="$HOME/ebs/tools/bin:$PATH"' >> ~/ebs/.shellrc
       source ~/ebs/.shellrc && rtk --version

4. Let Claude Code use it. In `~/.claude/settings.json`, add a PreToolUse hook for Bash. The
   `command -v` guard makes it a no-op on a workspace where rtk is not installed:

       {
         "hooks": {
           "PreToolUse": [
             {
               "matcher": "Bash",
               "hooks": [
                 { "type": "command",
                   "command": "command -v rtk >/dev/null 2>&1 && exec rtk hook claude; exit 0" }
               ]
             }
           ]
         }
       }

5. Tell the model what to expect. Add this to `~/.claude/CLAUDE.md`:

       ## Command output
       `rtk` condenses command output to save tokens, keeping every signal and dropping costly
       noise. Treat condensed output as the complete result: run commands normally and batch
       related commands into one call. Truncated results state their recovery path in their own
       output. Re-run a command as `rtk proxy <cmd>` only when its result is unusable: empty when
       output was clearly expected, contradicting its exit code, or garbled.

Restart any open Claude session afterwards. After a workspace rebuild, only step 1 needs
repeating; the clone, the binary and the shell line live on the persistent volume.
