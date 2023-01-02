### push and pop pragmas

The `push/pop` pragmas are very similar to
the option directive, but are used to override the settings temporarily.
Example:

```nim
{.push checks: off.}
# compile this section without runtime checks as it is
# speed critical
# ... some code ...
{.pop.} # restore old settings
```

`push/pop` can switch on/off some standard
library pragmas, example:

```nim
{.push inline.}
proc thisIsInlined(): int = 42
func willBeInlined(): float = 42.0
{.pop.}
proc notInlined(): int = 9
{.push discardable, boundChecks: off, compileTime, noSideEffect, experimental.}
template example(): string = "https://nim-lang.org"
{.pop.}

{.push deprecated, hint[LineTooLong]: off, used, stackTrace: off.}
proc sample(): bool = true
{.pop.}
```

For third party pragmas, it depends on its implementation but uses the
same syntax.

