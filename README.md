# Semgrep ResourceInaccessible repro

Minimal Maven project intended to produce the Supply Chain `ResourceInaccessible`
resolution error when scanned with **Dynamic Dependency Resolution enabled**.

## Scan configuration

1. Add this repository to Semgrep Managed Scanning with Supply Chain enabled.
2. Enable Dynamic Dependency Resolution (lockfileless scanning / local builds).
3. Do not supply credentials for the `github-auth-required` Maven repository.
4. Run a full scan using an engine that supports Maven registry-error classification.

A default scan with dynamic resolution disabled will skip manifest-only Maven
resolution and is not expected to produce this error. This is a dependency-resolution
error; it does not necessarily make the overall scan exit nonzero.

## How it works

`pom.xml` references GitHub's authenticated Maven registry and requests the unique
coordinate `com.example:semgrep-private-repro:1.0.0`. This is intentionally not a
published package: GitHub rejects the unauthenticated artifact request with HTTP 401
before package availability matters. The registry URL uses an existing fixture repo;
this project does not require creating or publishing a private package.

There is no precomputed `maven_dep_tree.txt`. Semgrep must run Maven to obtain
dependencies. A Maven artifact-transfer failure containing HTTP 401 or 403 and a
registry URL is classified as `ResourceInaccessible` by the resolver.

The repro depends on GitHub continuing to return 401 for anonymous Maven requests.
Supplying credentials or a Maven mirror can change the result.

## Reproduce the Maven failure locally

With Docker available:

```sh
docker run --rm -v "$PWD:/project:ro" maven:3.9.9-eclipse-temurin-17 \
  mvn -B org.apache.maven.plugins:maven-dependency-plugin:3.8.0:tree \
  -DoutputFile=/tmp/dependencies.json -DoutputType=json -f /project/pom.xml
```

Expected: Maven exits nonzero with an artifact-transfer error containing the GitHub
registry URL and `401 Unauthorized`. This command checks the underlying failure;
it does not itself run Semgrep or verify the Cloud Platform error display.
