### pure pragma

An object type can be marked with the `pure` pragma so that its type
field which is used for runtime type identification is omitted. This
used to be necessary for binary compatibility with other compiled
languages.

An enum type can be marked as `pure`. Then access of its fields always
requires full qualification.

