# Maven registry access repro

Minimal Java project that requests a dependency from GitHub's Maven registry
without credentials. Dependency resolution is expected to fail with
`401 Unauthorized`.

Leave registry credentials unset when reproducing the failure.
