# Installation

## Install bobr

The installer supports x86-64 Linux and requires Bash, `curl`, `tar`, `xz`,
`awk`, `sha256sum`, and standard coreutils. The release is prebuilt: Rust,
Cargo, and a host compiler are not required.

The shortest installation command is:

```sh
curl -fsSL https://bobr.build/install.sh | bash
```

If you prefer to inspect code before running it, download the same script
first:

```sh
curl -fsSLo bobr-install.sh https://bobr.build/install.sh
less bobr-install.sh
bash bobr-install.sh
rm bobr-install.sh
```

The installer downloads the latest x86-64 Linux release, verifies its entry in
the release's `SHA256SUMS`, validates all four commands and their build
provenance, and only then replaces the installed files. By default it installs
static `bobr`, `bobr-repo`, `bobr-fsobj-hash`, and
`bobr-sandbox-launcher` binaries into `~/.local/bin`. It neither invokes
`sudo` nor edits shell startup files.

Use `--bin-dir` for one invocation, or `BOBR_INSTALL_DIR` for the environment:

```sh
bash bobr-install.sh --bin-dir /path/to/bin
BOBR_INSTALL_DIR=/path/to/bin bash bobr-install.sh
```

There is no installation database. To uninstall the default installation,
remove exactly the four installed commands:

```sh
rm -- \
  "${HOME}/.local/bin/bobr" \
  "${HOME}/.local/bin/bobr-repo" \
  "${HOME}/.local/bin/bobr-fsobj-hash" \
  "${HOME}/.local/bin/bobr-sandbox-launcher"
```

For a custom destination, use that directory instead.

## Build requirements

- An x86-64 Linux host with Internet access and kernel OverlayFS support.
- Bash, `curl`, `tar`, `xz`, `awk`, `sed`, `jq`,
  [Nickel](https://nickel-lang.org/), GNU coreutils, and GNU findutils on `PATH`.
- A writable filesystem supporting hardlinks, Unix ownership/permissions, and
  extended attributes (xattrs).

Builds normally run as a regular user. This requires permitted unprivileged
user namespaces and sandbox mounts, `newuidmap` and `newgidmap` on `PATH` with
their distribution-provided privileges (`uidmap` / `shadow` packages), and
UID/GID ranges assigned to your account in `/etc/subuid` and `/etc/subgid`.

Alternatively, run Bobr through an already configured `podman unshare`
environment, or as root. Rootless Podman still needs working user namespaces
and UID/GID mappings; root does not need the mapping helpers.

## Running under a service manager

An outer service or container sandbox must preserve the facilities required by
the selected Bobr execution mode. For a regular rootless build, it must allow
the distribution's `newuidmap` and `newgidmap` privilege mechanism (set-uid,
file capabilities, or a wrapper), creation of nested user, mount, and UTS
namespaces, mounting `proc` inside the sandbox, and changing the hostname in
the nested UTS namespace. Builds must also be able to create files with
set-user-ID and set-group-ID modes.

Local input files must be readable by the sandbox build user. Input
directories must be searchable, and input programs and scripts executable. A
restrictive service umask such as `0077` can, for example, make executable
files from a Git checkout inaccessible to that user.

For systemd services, the following hardening settings are known places to
check when namespace setup or a sandbox build fails:

- `NoNewPrivileges` can prevent the mapping helpers from acquiring their
  distribution-provided privileges.
- `RestrictSUIDSGID` can prevent a build from setting required file modes.
- A narrow `CapabilityBoundingSet` can break mapping helpers or wrappers.
- `ProtectHostname` can prevent setup of the nested UTS namespace.
- `ProtectKernelTunables` and `ProtectKernelModules` can introduce locked
  mounts below `/proc` that prevent the sandbox from mounting its own `proc`.

Do not disable these settings mechanically. Their effects depend on the
distribution and on whether Bobr runs rootless, under `podman unshare`, or as
root. Treat them as diagnostics for the specific failed operation.
