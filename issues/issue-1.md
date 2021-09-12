## The curious case of the unoptimized empty `for` loop

While reading [@vandadnp](https://github.com/vandadnp)'s [second issue](https://github.com/vandadnp/going-deep-with-dart/blob/main/issue-2-const-in-dart/issue-2-const-in-dart.md#curious-case-of-mixing-const-int-and-const-double), I stubled upon something I found interesting: 

> ## The curious case of the unoptimized empty `for` loop
> 
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
>   
> we get the following AOT code 🤦🏻‍♂️:
> <details>
>  <summary>AOT Code</summary>
> 
>  ```asm
>                      Precompiled____main_1433:
> 000000000009a644         push       rbp                                         ; CODE XREF=Precompiled____main_main_1434+17
> 000000000009a645         mov        rbp, rsp
> 000000000009a648         cmp        rsp, qword [r14+0x40]
> 000000000009a64c         jbe        loc_9a687
> 
>                      loc_9a652:
> 000000000009a652         mov        eax, 0xdeadbeef                             ; CODE XREF=Precompiled____main_1433+74
> 
>                      loc_9a657:
> 000000000009a657         cmp        rsp, qword [r14+0x40]                       ; CODE XREF=Precompiled____main_1433+48
> 000000000009a65b         jbe        loc_9a690
> 
>                      loc_9a661:
> 000000000009a661         mov        r11d, 0xfeedfeed                            ; CODE XREF=Precompiled____main_1433+83
> 000000000009a667         cmp        rax, r11
> 000000000009a66a         jge        loc_9a676
> 
> 000000000009a670         add        rax, 0x1
> 000000000009a674         jmp        loc_9a657
> 
>                      loc_9a676:
> 000000000009a676         call       Precompiled____exit_1015                    ; Precompiled____exit_1015, CODE XREF=Precompiled____main_1433+38
> 000000000009a67b         mov        rax, qword [r14+0xc8]
> 000000000009a682         mov        rsp, rbp
> 000000000009a685         pop        rbp
> 000000000009a686         ret
>                         ; endp
> 
>                      loc_9a687:
> 000000000009a687         call       qword [r14+0x240]                           ; CODE XREF=Precompiled____main_1433+8
> 000000000009a68e         jmp        loc_9a652
> 
>                      loc_9a690:
> 000000000009a690         call       qword [r14+0x240]                           ; CODE XREF=Precompiled____main_1433+23
> 000000000009a697         jmp        loc_9a661
> ```
> </details>
> 
> this is very similar, if not identical to the previous code we looked at, and the problem I have with this code is that the Dart compiler didn't understand that this was an empty loop, and really created the loop code for it! Is this a bug? It could be. If you're a person working on the Dart compiler maybe you could sort this out!
> 
> For us Dart developers though this means that if you have a for loop somewhere with a variable, make sure it does something 😂

As we can see, Dart doesn't seem to optimize the fact that the for loop is empty.

At first, I was agreeing with [@vandadnp](https://github.com/vandadnp) and thought it didn't really make any sense. But the more I thought about it, the more it disturbed me without really knowing why.

A few days later, I realized someting. We use the `for` loop mainly (and almost exclusively) to iterate `i` from `a` (_a number_) to `b` (_another number_).
Many languages have syntax sugaring for this type of operation ([Kotlin's `for (i in 1..5) {}`](https://kotlinlang.org/docs/control-flow.html#for-loops),   [Swift's `for i in 1...5 {}`](https://docs.swift.org/swift-book/LanguageGuide/ControlFlow.html), etc), but Dart doesn't.

Which makes it really easy to overlook one fact. We can use Dart's `for` loop in other ways!

Consider this code:

```dart
class C {
  C();

  int i = 0;
}

void main() {
  final myClass = C();

  for (final i = myClass; i.i < 5; i.i++) {}

  print(myClass.i); // We expect to see 5 printed
}
```

We expected to see `5` printed right ? Because when we think about it, `for` loops are nothing more than fancy `while` loops: 

```dart

for (var i = 0; i < 5; i++) {
  /*Code*/
}

// Is strictly equivalent to: 

var i = 0;
while (i < 5) {
  /*Code*/
  i++;
}
```

It would thus not make sense to optimize the empty `for` loops as both the second and third statements of the `for` loops should be run every iteration no matter what. Remember `for` loops are only syntax-sugared `while` loop and we would expect this code to be compiled without the `while` block being optimized out:

```dart
class C {
  C();

  int i = 0;
}

void main() {
  final myClass = C();


  // This block shouldn't be optimized out
  final i = myClass
  while (i.i < 5) {
    i.i++;
  }

  print(myClass.i); // We expect to see 5 printed
}
```
