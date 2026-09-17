# surefire-3.6.0-regex-include-regression

Minimal reproduction of a Maven Surefire 3.6.0 regression: when `<includes>` uses a
`%regex[...]` pattern, Surefire silently reports **zero tests**, even for a single
normal test class with a real, passing `@Test` method. No error, no warning —
just `BUILD SUCCESS` with `Tests run: 0`.

Works correctly on Surefire 3.5.6 with the exact same configuration, and works
correctly on 3.6.0 too if the include is switched to a plain glob (e.g.
`**/*Test.java`) instead of `%regex[...]`.

## Reproduce locally

```shell
mvn test                              # Surefire 3.6.0 (default) - Tests run: 0
mvn test -Dsurefire.version=3.5.6     # Surefire 3.5.6 - Tests run: 1, PlainTest passes
```

## CI

`.github/workflows/repro.yml` runs both versions in a matrix and asserts that
`PlainTest` actually executed. The `3.5.6` job passes; the `3.6.0` job fails the
assertion step, demonstrating the regression.

## Project structure

- `src/test/java/repro/PlainTest.java` — a normal test class with one passing
  `@Test`. This is the one that mysteriously stops running on 3.6.0 when the
  `<includes>` pattern uses `%regex[...]`.
