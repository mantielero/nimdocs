### fatal pragma

The `fatal` pragma is used to make the compiler output an error message
with the given content. In contrast to the `error` pragma, the
compilation is guaranteed to be aborted by this pragma. Example:

```nim
when not defined(objc):
{.fatal: "Compile this program with the objc command!".}
```
