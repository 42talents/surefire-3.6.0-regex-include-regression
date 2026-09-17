## Summary

When `<includes>` on `maven-surefire-plugin` uses a `%regex[...]` pattern, Surefire 3.6.0 silently reports **zero tests**, even for a single normal test class with a real, passing `@Test` method. No error, no warning, `BUILD SUCCESS`.

This works correctly on Surefire 3.5.6 with the exact same configuration — only the plugin version differs.

**Correction:** an earlier version of this report attributed the regression to a second, empty test class also being matched by the include pattern ("if any matched class has zero discoverable tests, the whole batch reports zero"). That theory was wrong — removing the empty class entirely and re-running with only the single real test class still reproduces `Tests run: 0`. The regression is simpler and more severe: `%regex[...]` includes appear to be broken outright in 3.6.0, independent of what they match. Plain glob includes (e.g. `**/*Test.java`) on the same Surefire version work correctly.

## Reproduction

Minimal repo: https://github.com/42talents/surefire-3.6.0-regex-include-regression

One test class:
- `PlainTest.java` — a normal class with one passing `@Test`

pom.xml:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.6.0</version>
    <configuration>
        <includes>
            <include>%regex[.*Test.class]</include>
        </includes>
    </configuration>
</plugin>
```

```
mvn test                              # Surefire 3.6.0 (default): Tests run: 0
mvn test -Dsurefire.version=3.5.6     # Surefire 3.5.6: Tests run: 1, PlainTest passes
```

Switching the include to a plain glob (no `%regex[...]`) fixes it even on 3.6.0:
```xml
<include>**/*Test.java</include>
```

A GitHub Actions matrix build in the repo (`.github/workflows/repro.yml`) demonstrates this automatically — the 3.5.6 job passes, the 3.6.0 job fails an assertion that `PlainTest` actually ran.

## Expected

`PlainTest`'s single test runs and passes, matching 3.5.6 behavior and matching what a plain glob include produces on 3.6.0.

## Actual

`Tests run: 0, Failures: 0, Errors: 0, Skipped: 0`, `BUILD SUCCESS`. `target/surefire-reports/repro.PlainTest.txt` is never created.

## Environment

- Maven Surefire 3.6.0 (broken) / 3.5.6 (working)
- JUnit Jupiter 5.14.4, JUnit Platform 1.14.4
- JDK 21 (Temurin)
- Provider: `org.apache.maven.surefire.junitplatform.JUnitPlatformProvider` (auto-detected)
- Reproduces both forked (`forkCount=1`) and in-process (`forkCount=0`)

## Notes

This was discovered as a real-world regression affecting a much larger project (thousands of tests across ~50 classes), where it manifested as a total loss of test execution once an `%regex[...]` include was introduced. The minimal repro isolates the trigger to the `%regex[...]` include syntax itself, not to any property of the classes it matches.
