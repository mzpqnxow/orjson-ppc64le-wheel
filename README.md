# orjson-ppc64le-wheel

Binary wheels for orjson and maturin for IBM POWER8 (ppc64le)

## Incomplete

This is a work in progress and will be completed if it turns out to be usefl. For now, the wheels are here to be installed. These notes are for trying to reproduce the builds

## Notes

There are a few problems preventing installation of `orjson` on a ppc64le system. These binary wheels were produced by applying workarounds

## Problems installing orjson and its dependant packages

In a nutshell:

1. Maturin requires [ring](https://github.com/briansmith/ring), which does not yet support `ppc64le` per [this](https://github.com/briansmith/ring/issues/389)
2. orjson does not support `ppc64le`

## Solution - Building Maturin

You need `maturin` to build `orjson`, so one way or another you need to get a `ppc64le` binary and install it in the same virtualenv you plan to install `orjson` in. Luckily this isn't too difficult of a problem. To build maturin, you can opt out of the features that introduce the dependency on ring by building `maturin` with the following as described [here](https://github.com/PyO3/maturin/issues/366)

```
cargo install maturin --no-default-features --features=auditwheel,log,human-panic
```

## Solution - Building orjson

With `maturin` available, you can build `orjson`. However, `orjson` does not support `ppc64le` architecturee, so you'll need to fork `orjson` and patch the source.


### Building the Wheel

Because `orjson` uses PEP-517, you can't just invoke `setup.py bdist_wheel` to build a wheel. The best way I found to produce a wheel is invoking `maturin` directly, using `pep517 build-wheel`:

```
maturin pep517 build-wheel --manylinux off
```

Depending on what you edited in `.cargo/config`, you won't need any feature flags or anything.

## Misc. Notes

You can force a specific (pre-patched) version of `maturin` to install from a binary wheel using requirements.txt. In this example, the binary wheel in `/home/<users>/mt/` is called `maturin-0.9.0a2-cp37-cp37m-linux_ppc64le.whl`. It gets picked up as the build-time dependency

```
# git+https://github.com/mzpqnxow/maturin@v0.9.0-alpha.2.ppc64le#egg=maturin
/home/<user>/mt/
```
