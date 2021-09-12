While reading [@vandadnp](https://github.com/vandadnp)'s [second issue](https://github.com/vandadnp/going-deep-with-dart/blob/main/issue-2-const-in-dart/issue-2-const-in-dart.md#curious-case-of-mixing-const-int-and-const-double), I stubled upon something I found weird: 

> ## The curious case of the unoptimized empty `for` loop

> for the given Dart code:
> 
> ```dart
> import 'dart:io' show exit;
> 
> void main(List<String> args) {
>   for (var x = 0xDEADBEEF; x < 0xFEEDFEED; x++) {}
>   exit(0);
> }
> ```
  
_\*Assembly code for this code snipper showing that it effectively does iterate even though it does quite apparently "nothing"\*_
