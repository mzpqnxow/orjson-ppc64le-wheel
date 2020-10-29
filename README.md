# orjson-ppc64le-wheel

Binary wheels for orjson and maturin for IBM POWER8 (ppc64le)

## Support

While I can't call these wheels "supported", I can say that I've used them to load JSON files on:

 * Python3.6 + ppc64le
 * Python3.7 + ppc64le
 * Python3.8 + ppc64le
 * Python3.9 + ppc64le

## Incomplete

This is a work in progress and will be completed if it turns out to be useful. If you just want to use `orjson` on `ppc64le`, take the binary wheel and install it with `pip` and you're done. These notes are only here for my own reference, to help reproduce the builds to help put together PRs to get seamless `pip` installation to work in the future on `ppc64le` and other non-x86 / non-arm platforms


## Notes

There are a few problems preventing installation of `orjson` on a ppc64le system. These binary wheels were produced by applying workarounds

## Problems installing orjson and its dependant packages

In a nutshell:

1. Maturin requires [ring](https://github.com/briansmith/ring), which does not yet support `ppc64le` per [this](https://github.com/briansmith/ring/issues/389)
2. orjson does not support `ppc64le`

## Solution - Building Maturin

At a high level, to make `maturin` work on `ppc64le`, you need to do two things:

  1. Provide build-flags that effectively disable the ring dependency, which is not yet functional on ppc architectures
  2. Add explicit support for PPC architectures so that it can generate `ppc64le` build triplets

To build maturin, you can opt out of the features that introduce the dependency on ring by building `maturin` with the following as described [here](https://github.com/PyO3/maturin/issues/366)

```
cargo install maturin --no-default-features --features=auditwheel,log,human-panic
```

## Pull Request - maturin

There is now a PR open to `maturin` [here](https://github.com/PyO3/maturin/pull/367) which solves the `maturin` part of the problem

## Solution - Building orjson

With `maturin` available, you can build `orjson`. However, `orjson` does not support `ppc64le` architecturee, so you'll need to fork `orjson` and patch the source.


### Building the Wheel

Because `orjson` uses PEP-517, you can't just invoke `setup.py bdist_wheel` to build a wheel. The best way I found to produce a wheel is invoking `maturin` directly, using `pep517 build-wheel`:

```
maturin pep517 build-wheel --manylinux off
```

Depending on what you edited in `.cargo/config`, you won't need any feature flags or anything.

## Disabling features to avoid simd breakage

When building `orjson`, disable `simd-accel` feature. I think the proper way to do this is:

```
target-feature=-simd-accel
```

## The rest of the story

There are some dependency versions that need to be manually edited at this time to complete the whole process. I have not yet documented these. They may work themselves out over time

## Misc. Notes

* You can force a specific (pre-patched) version of `maturin` to install from a binary wheel using requirements.txt. In this example, the binary wheel in `/home/<users>/mt/` is called `maturin-0.9.0a2-cp37-cp37m-linux_ppc64le.whl`. It gets picked up as the build-time dependency
* The `maturin-0.9.0a2-cp37-cp37m-linux_ppc64le.whl` will install fine on 3.6, 3.7, 3.8 and 3.9
* The [fork](https://github.com/mzpqnxow/maturin/tree/v0.9.0-alpha.2.ppc64le) used to build the `maturin` wheel has a nonsense tag name (`v0.9.0-alpha.2.ppc64le`) and is outside of the versions that `orjson` wants in its setuptools `require` directive. This must be hand editing if trying to build `orjson` from source

```
# git+https://github.com/mzpqnxow/maturin@v0.9.0-alpha.2.ppc64le#egg=maturin
/home/<user>/mt/
```
