### hint pragma

The `hint` pragma is used to make the compiler output a hint message
with the given content. Compilation continues after the hint.






### Disabling certain messages

Nim generates some warnings and hints ("line too long") that may annoy
the user. A mechanism for disabling certain messages is provided: Each
hint and warning message contains a symbol in brackets. This is the
message\'s identifier that can be used to enable or disable it:

```nim
{.hint[LineTooLong]: off.} # turn off the hint about too long lines
```

This is often better than disabling all warnings at once.

