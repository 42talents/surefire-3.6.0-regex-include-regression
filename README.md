# surefire-3.6.0-regex-include-regression
Minimal Maven project reproducing a Surefire 3.6.0 regression: JUnit 5 tests are silently dropped (Tests run: 0, BUILD SUCCESS, no error) when &lt;includes> uses a `%regex[...]` pattern with @Nested @ParameterizedClass tests. Works on 3.5.6 with the same config — just bump the plugin version to compare. 
