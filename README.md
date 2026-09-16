# surefire-3.6.0-regex-include-regression

Minimal reproduction of a Maven Surefire 3.6.0 regression: when `<includes>` uses a
`%regex[...]` pattern, if *any* matched class has zero discoverable JUnit 5 tests
(here, `EmptyTest` — a completely empty class that just happens to match the
naming pattern), Surefire silently reports **zero tests for every matched class**,
including unrelated ones with real, passing `@Test` methods. No error, no
warning — just `BUILD SUCCESS` with `Tests run: 0`.

Works correctly on Surefire 3.5.6 with the exact same configuration.

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

- `src/test/java/repro/EmptyTest.java` — a class with no test methods at all,
  present only because its name matches the include pattern.
- `src/test/java/repro/PlainTest.java` — a normal test class with one passing
  `@Test`. This is the one that mysteriously stops running on 3.6.0.
