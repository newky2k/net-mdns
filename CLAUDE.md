# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`DSoft.Makaretu.Dns.Multicast` — newky2k's fork of Richard Schneider's net-mdns (multicast DNS and DNS service discovery, RFC 6762 and 6763) for modern .NET. The library is `src/Mdns.csproj` (assembly and NuGet package `DSoft.Makaretu.Dns.Multicast`, namespace `Makaretu.Dns`); it depends on `Makaretu.Dns` 2.0.1 and `Common.Logging`. `test/` holds the MSTest unit tests; `Spike/`, `Browser/` and `traffic/` are small console tools and `doc/` is the docfx site.

## Branching

- `development` is the working branch: branch from it and open pull requests into it. Never commit or open a pull request directly against `main`.
- `main` is the release branch. Every push to `main` runs `release.yml`, which publishes the package to nuget.org and creates a GitHub release, so only merge `development` into `main` when a release is intended, and only when asked to release.

## Build and test

The solution is `Mdns.sln`. The library and tests multi-target `net8.0;net9.0;net10.0;net11.0`; the console tools target `net9.0;net10.0;net11.0`. Build with the .NET 10 and .NET 11 SDKs (`global.json` rolls forward to the latest major SDK); running the tests on `net8.0` or `net9.0` also needs those runtimes.

```bash
dotnet build Mdns.sln -c Release
dotnet test test/MdnsTests.csproj -c Release -f net10.0
```

`GeneratePackageOnBuild` is on for `src/Mdns.csproj`, so a build emits the `.nupkg`. A developer build is always version 0.42; the real version is set by `release.yml`.

## CI

GitHub Actions (`.github/workflows/`) on `windows-latest`: `ci.yml` builds Release and runs the unit tests for pull requests into `main` and `development`; it publishes nothing. `release.yml` runs on every push to `main` (changes only to Markdown or workflow files are skipped; run it by hand to test a workflow change), builds Release as `1.0.<yyMM>.<run number>` plus `RELEASE_SUFFIX` (empty for a stable version), pushes the package to nuget.org with Trusted Publishing (OIDC, `NUGET_USER` secret, `nuget` environment), then tags the commit `v<version>` and creates a GitHub release with the package attached. The old `.travis.yml`, `appveyor.yml` and `.circleci` files are inherited from upstream and are not used.
