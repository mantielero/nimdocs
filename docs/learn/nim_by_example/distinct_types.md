When creating a distinct type from an object type, none of its fields are carried over. If the fields are wanted, they can be brought over through an overloading of the `{.borrow.}` pragma. If they are not borrowed, they cannot be accessed.
```nim
type
  Foo = object
    a: int
  MyFoo {.borrow: `.`.} = distinct Foo

var value: MyFoo
echo value.a  # Works
```