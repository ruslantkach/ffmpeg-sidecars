# ffmpeg-sidecars

Reproducible LGPL FFmpeg builds for macOS, packaged as drop-in sidecar
binaries for Tauri / Electron / native macOS apps.

## What's here

- `.github/workflows/build-ffmpeg.yml` — a GitHub Actions workflow that
  compiles FFmpeg from upstream source on GitHub-hosted macOS runners,
  strips and ad-hoc signs the binaries, and attaches them to a GitHub
  Release alongside a `SHA256SUMS` manifest.

That's the entire repository. The actual binaries live on the
[Releases page](../../releases).

## Configuration

FFmpeg is built with the following flags:

```
--disable-gpl
--disable-nonfree
--disable-shared
--enable-static
--disable-debug
--disable-doc
--disable-ffplay
--disable-ffprobe
```

This produces an **LGPL-only** binary: no libx264, libx265, libvidstab,
or any other GPL-licensed components are included. Apps bundling this
binary can stay under MIT/Apache/proprietary licenses without inheriting
GPL obligations.

No external resampler is linked — FFmpeg's bundled `swresample` handles
all sample-rate conversion via the `aresample` filter. Keeping the build
free of external resampler libs (like libsoxr) means the resulting
binary has no Homebrew/dylib runtime dependencies and ships
self-contained.

## Architectures

The workflow currently builds:

- `ffmpeg-aarch64-apple-darwin` — macOS Apple Silicon (M1+)
- `ffmpeg-x86_64-apple-darwin` — macOS Intel

The filenames match Tauri's `externalBin` target-triple convention. To
add other platforms (Windows, Linux), extend the matrix in
`.github/workflows/build-ffmpeg.yml`.

## Verifying a download

Every release ships with a `SHA256SUMS` file. To verify:

```bash
cd /path/to/downloaded/binaries
shasum -a 256 -c SHA256SUMS
```

The checksums in `SHA256SUMS` are produced inside the workflow run, so
re-running the workflow on a fork produces bit-identical artifacts
(modulo timestamps embedded by the linker — content is reproducible).

## Triggering a build

Two ways:

- **Tag push** — `git tag ffmpeg-8.1.1 && git push --tags`. The tag's
  trailing version segment is parsed into the FFmpeg `n<version>` source
  tag; release name matches the git tag.
- **Manual** — Actions tab → "Build FFmpeg sidecar" → "Run workflow".
  Specify the FFmpeg version (e.g. `n8.1.1`) and release tag.

A full build takes ~15 minutes (matrix of two macOS runners running in
parallel).

## License

The workflow file itself is MIT (see `LICENSE`).

The **binaries** produced by the workflow are governed by the
[LGPL v2.1+](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.en.html),
inherited from FFmpeg. Source for each release is at
[github.com/FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg) at the
corresponding `n<version>` tag — that satisfies the LGPL's source
availability requirement.

Apps consuming these binaries should:

1. Acknowledge FFmpeg in their About / Credits page
2. Link to https://github.com/FFmpeg/FFmpeg as the source
3. Ship a copy of the LGPL license text alongside

`scripts/install-ffmpeg-sidecar.sh` in consuming apps takes care of
verification and placement.
