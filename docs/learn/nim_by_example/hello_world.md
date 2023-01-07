Create the following file in your favorite text editor:
```nim title="helloworld.nim"
echo "Hello World"
```

Compile it:
```sh
nim c helloworld.nim
```
and execute it as any other binary:

=== "Windows"
    ```sh
    C:> helloworld.exe
    Hello World
    ```

=== "Linux"
    ```sh
    $ ./helloworld
    Hello World
    ```

You can compile and execute it in one shot:
```sh
nim c -r helloworld.nim
...
...
...
Hello World
```

And you can avoid all the non essential messages by doing:
```sh
nim c -r --verbosity:0 helloworld.nim
```

Visit [Nim Compiler User Guide](https://nim-lang.org/docs/nimc.html), to find out about what can be done with Nim's compiler.


???+ tip "Release mode"

    Don't forget to compile using release mode when you intend to release the binary. This is required in order to achieve the expected performance:
    ```
    nim c -r -d:release helloworld.nim
    ```
        
