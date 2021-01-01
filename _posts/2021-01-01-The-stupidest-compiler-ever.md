---
tags: programming
excerpt_separator: <!--more-->
---

# The stupidest compiler ever

So I have an interpreter for a subset of a toy language (Pipes), and wanted to do a
compiler for it.

I like working iteratively, having incomplete (but
working!) programs during development. So what is the simplest possible program
our compiler should be able to compile? A hello world? That talks to the standard output! I have no clue how
to do that yet!

What about the Pipes equivalent to this C/C++ program?
```
int main() {
    return 5;
}
```
In summary, this post tells about an application of [this wonderful
post](https://www.hanshq.net/ones-and-zeros.html) that teaches you how to
produce the machine code for very basic programs like the one above, and run it. The
corresponding assembler would be:
```
movl $5, %eax
ret
```
and the machine code would be 
```
1011100000000101000000000000000000000000
11000011
```

I just added a simple way to write that into a file, and then load the file and
execute it.

<!--more-->

## 1-minute introduction to Pipes

Pipes is a language I'm making up that mimics bash's pipes, like when you do
`cat file.txt | head`. In that sense, a function `f` can be called like `a |f`,
where `a` is/are the argument(s).

Chaining calls are left-associative as in bash: `a |f |g` is the same as `(a |f)
|g`, which is `g(f(a))` in traditional languages.

It has some nice properties but it's crazy bad for others. Anyway, don't take it
too seriously.

## Current Pipes interpreter capabilities

Currently, in [the first decent version of
Pipes](https://bitbucket.org/jmmut/pipes/src/0.0.0/src/test/test_interpreter.cpp),
besides being able to call functions with `|`, you can define functions
(lambdas, really) like `function x {x}`. The `x` before the the braces is the
parameter, and the `x` within the braces is the function body (just the `x`
expression).

You can also use integer numbers but there
are no operations you can do on them (you can't sum or multiply numbers),
besides passing them to functions, which makes them just atom values.

In summary, this code uses the number `5`, and passes it to the identity
function, which returns the same 5 if the code is evaluated:

```
5 |function x {x}
```

Small tangent: you can nest function definitions that have closures, which makes
the language already capable of doing
[lambda-calculus](https://en.wikipedia.org/wiki/Lambda_calculus)
and [combinators](https://en.wikipedia.org/wiki/Combinatory_logic) (thus,
turing-complete in an unusable way! yay...). If you don't
know what combinators are and you want to know more, the wikipedia page is rather obtuse, watch [this
video](https://www.youtube.com/watch?v=3VQ382QG-y4)
instead (which I found thanks to [this
blog](https://risingentropy.com/for-loops-and-bounded-quantifiers-in-lambda-calculus/)).
I won't go into nested lambdas in this post.

The interpreter can build an Abstract Syntax Tree (AST) with the code, and
optionally evaluate it. For example, if you tell the interpreter to evaluate 
`5 |function x {x}`, it will give you `5` as result.

## Compilation process

Compiling would mean roughly turning the AST into assembly code. More
specifically, expressing the AST in assembler (like `mov %eax 5`), and turning the assembler into
machine code (ones and zeros), and then writing that machine code in an
executable file, in a [file format that OSs
understand](https://en.wikipedia.org/wiki/Comparison_of_executable_file_formats)
(like ELF in Linux). Besides the machine code, those file formats have some
metadata like platform settings (e.g. machine code is in 32 or 64 bits).

Also, it's usual to express your AST first in an intermediate representation
(like a bytecode) before going to assembler, in order to support different
processors.

As this is not a serious project and right now I'm not interested in learning
executable formats like ELF, nor plan to support other instruction sets than my
processor's (intel64), I'm going to skip all those stages and **go directly from
my AST to machine code**.

(o_O)  <- your face should be like this now.

My face the whole time on this project:

<div class="mydiv">
<iframe
src="https://www.youtube.com/embed/zUyqDxa2k3w"
frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
</iframe>
</div>

## Return 5 in assembler

Again, if you want the details, go to [this
post](https://www.hanshq.net/ones-and-zeros.html). The point is that a `return
5`, or `mov $5 %eax; ret` corresponds to this binary machine code :

```
1011100000000101000000000000000000000000
11000011
```
which is this in hexadecimal:
```
b805 0000 00c3
```
where:
- `b8` is the part of the `mov $5 %eax` that states the instruction `mov` and
  the register `%eax` (register used by C/C++ ABI to return values)
- the `05 00 00 00` part is the (immediate) number 5 in little-endian 32 bits.
- the `c3` part is the `ret` instruction (return).

## Joining Pipes interpreter and "return x" in assembler

This is [the key part of the
compiler](https://bitbucket.org/jmmut/pipes/src/be9ef5f278b67a386d88b7749283d43b739fd3a3/src/assembler/PipesAssembler.cpp#lines-10):
```
void PipesAssembler::compile(std::string code) {
    auto expression = interpret(std::move(code));
    if (expression.type == Expression::VALUE) {
        assembler.emit_mov_imm32(Register::EAX, expression.result);
        assembler.emit_return();
    } else {
        throw randomize::exception::StackTracedException{
                "Unimplemented: can only compile concrete integer values"};
    }
}

```

Note how we call the `interpret()` function with the code, to try to simplify it.
If you put `5 |function x {x}`, the `interpret()` will return an expression
holding the value 5.

If the result is a single number, then hardcode it in assembler!

The `emit_*`
functions put the binary machine code in an executable memory buffer held by
`assembler`. The buffer
can be called, or written to a file.

## Writing to an executable file

Sadly, we won't be able to run that from an executable file without learning the
ELF format and implementing the writing of it. But what we can do is create [a
simple
loader](https://bitbucket.org/jmmut/pipes/src/0.0.2/src/assembler/loader.cpp)
that takes the machine code from a file and puts it in executable
memory, and then call it.

The format I chose has 3 parts:

- "PIPE", a 4-char magic number.
- 8 bytes of an int64 (little endian) with the program size "n" in bytes.
- "n" bytes (little endian) of the machine code of the intel64 instructions.

The first 2 parts are really redundant but hopefully will reduce confusion.

If you run the compiler on a file "ret_5.pipes" containing our `5 |function x
{x}` code you will get this binary
file:

```
$ xxd ret_5
00000000: 5049 5045 0600 0000 0000 0000 b805 0000  PIPE............
00000010: 00c3                                     ..
```

- `0x50 49 50 45`: magic number "PIPE" in ASCII
- `0x06 00 00 00 00 00 00 00`: little endian 64 bit of "6" bytes (program size)
- `0xb805000000 c3`: the machine code

## Whole compile-and-run process

You can go to the Pipes repo and compile the compiler yourself, and this is how
it's used:

```
echo "5 |function x {x}" > ret_5.pipes
./build/bin/compiler ret_5.pipes
./build/bin/loader ret_5
```

The loader executes ret_5 and says:
> The machine code of the program at ret_5 is 6 bytes long.
>
> Execution result: 5


## Conclusion

This compiler can only take a number and hardcode it into a pseudo-executable file,
in the most silly way, and I love it.
