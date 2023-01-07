Line comments:
```nim 
# This is a comment
echo "This is code" # This is another comment
```

Multiline or block comments:
```nim
#[ This is a multi line comment
it continues until it is terminated
]#
echo "This is code"
```
They can be nested:
```nim
#[ This is a multi line comment
 #[
  This is a nested comment
 ]#
it continues until it is terminated
]#
echo "This is code"
```