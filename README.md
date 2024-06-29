# zig-project-template

[![reference Zig](https://img.shields.io/badge/zig%20-0.12.0-orange)](https://ziglang.org/)
[![reference Zig](https://img.shields.io/badge/zigdoc%20-pages-orange)](https://ziglang.org/)
[![build](https://github.com/dgv/zig-project-template/actions/workflows/build.yml/badge.svg)](https://github.com/dgv/zig-project-template/actions/workflows/build.yml)
[![codecov](https://codecov.io/gh/dgv/zig-project-template/branch/main/graph/badge.svg)](https://codecov.io/gh/dgv/zig-project-template)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

### Badges Roadmap
- [X] [zig version](#version)
- [X] [zig doc](#cocumentation)
- [X] [continuos integration](#continuos-integration)
- [X] [code coverage](#code-coverage)
- [X] [license](#license)

### Version
On Zig 0.12.0 [`minimum_zig_version`](https://github.com/dgv/zig-project-template/blob/main/build.zig.zon#L15-L18) was introduced by [Zon](https://zig.news/edyu/zig-package-manager-wtf-is-zon-558e), show this reference in a badge can be pretty useful to determine compatibility.

```zig
    // This field is optional.
    // This is currently advisory only; Zig does not yet do anything
    // with this value.
    .minimum_zig_version = "0.12.0",
```

### Documentation

Documentation can be generated using `-femit-docs` flag or as step on `build.zig`:

```zig
    const docs = b.addInstallDirectory(.{
        .source_dir = lib.getEmittedDocs(),
        .install_dir = .prefix,
        .install_subdir = "../docs",
    });
    b.getInstallStep().dependOn(&docs.step);
```

https://ziglang.org/documentation/master/#Comments <br>
https://sudw1n.gitlab.io/posts/zig-build-docs/

### Continuos Integration

Using following configuration with [Github Actions Matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs) and [build-zig](https://github.com/marketplace/actions/setup-zig):

```yaml
name: build
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: 0 9 * * *
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        include:
          - os: windows-latest
            crlf: fix
          - os: ubuntu-latest
            coverage: -Dtest-coverage=true
    runs-on: ${{matrix.os}}
    steps:
      - name: Force Line Endings for Windows
        if: ${{ matrix.crlf }}
        run: |
          git config --global core.autocrlf false
          git config --global core.eol lf

      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Zig
        uses: goto-bus-stop/setup-zig@v2
        with:
          version: master

      - name: Run Tests
        run: |
          zig version
          zig build test -Dcomptime-tests=false ${{ matrix.coverage }}
```

https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge

### Code Coverage

We can use [kconv](https://github.com/SimonKagstrom/kcov) integrated on `build.zig` during the tests with `-Dcoverage=true` flag, then submit the generated results on `cover` directorty to [codeconv](https://github.com/SimonKagstrom/kcov/blob/master/doc/codecov.md) using uploader utility adding following steps:
[build.zig](https://github.com/dgv/zig-project-template/blob/main/build.zig.zon#L15-L18)
```zig
// Converage step to generate debug messages for codecov using kcov
const coverage = b.option(bool, "coverage", "Generate test coverage") orelse false;
if (coverage) {
    const run_cover = b.addSystemCommand(&.{
        "kcov",
        "--clean",
        "--include-pattern=src/",
        "cover",
    });
    run_cover.addArtifactArg(exe_unit_tests);
    run_exe_unit_tests.step.dependOn(&run_cover.step);
}
```
  [build.yml](https://github.com/dgv/zig-project-template/blob/main/.github/workflows/build.yml#L36-L53)
```yaml
- name: Download kcov
  if: ${{ matrix.os == 'ubuntu-latest' }}
  run: |
    sudo apt-get install -y kcov
...
- name: Upload Codecov
  if: ${{ matrix.os == 'ubuntu-latest' }}
  env:
    CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
  run: |
    curl -Os https://uploader.codecov.io/latest/linux/codecov
    chmod +x codecov
    ./codecov -t ${CODECOV_TOKEN}
```

Better converage analysis is possible using [kcov-zig](https://github.com/liyu1981/kcov/tree/kcov-zig) fork, wip...

https://zig.news/squeek502/code-coverage-for-zig-1dk1 <br>
https://zig.news/liyu1981/tiny-change-to-kcov-for-better-covering-zig-hjm

### License
You can just use [Markdown License badges](https://gist.github.com/lukas-h/2a5d00690736b4c3a7ba) as reference.
