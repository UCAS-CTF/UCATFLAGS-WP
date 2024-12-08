# LVM 1 WriteUP

    LVM 1: Church Number Writeup

====================================

```
Just copy it and paste it into LVM REPL, the

result is:

    (λyb|y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(y(yb)))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))))

If you want, you could count by hand (233).

Or you could just use a more simplier program

to count the `y' to convert church number to

integer:

    def church_number_to_integer(expr : str, sym : str = "y") -> int:

    count = 0

    for char in expr:   # for all the characters in `expr'

    if char == sym:# if has `sym' symbol appearance

    count += 1   # increase count for `sym'

    return count - 1    # remove `sym' count in identifier list

So the real number is `233'. (see `1.py' for details)

Or if you thought it was not strictly enough,

you could also use `parse.py' to do the AST.

More: the task was made by the following code.

    .p                                    -> trun off pretty print

    .l(𝐬) (λxyz|y(xyz))                   -> successor

    .l(𝐩) (λxsz|x(λgh|h(gs))(λu|z)(λu|u)) -> predecessor

    .l(+) (λxy|y#𝐬x)                      -> addition

    .l(×) (λxys|x(ys))                    -> multiplication

    .l(∸) (λxy|y#𝐩x)                      -> monus subtraction

    .l(𝐞) (λbe|eb)                        -> b^e exp

    .d(0)                                 -> set recursion to 0

    ((λab|(#+ (#+ (#× (#𝐩 a) (#𝐞 b (#𝐩 a))) a) (#× a b))) #3 #10) -> ((((3 - 1) * (10 ^ (3 - 1))) + 3) + (3 * 10))

Tips: here hides some drawbacks of literal macro,

if you just use expression

    ((λxy|(#+(#+(#×(#𝐩x)(#𝐞y(#𝐩x)))x)(#×xy)))#3#10)

The result is wrong: (λa|(λz|(a((aa)z)))), because simple

macro is just literally replacing string and has no

safty check for AST and context.

Want something more safe? See Lisp macro.
```


`1.py`

```python
from lib.utils import AST
from lib.parse import parse

def church_number_to_integer_bad_one(expr : str, sym : str = "y") -> int:
    count = 0
    for char in expr:   # for all the characters in `expr'
        if char == sym: # if has `sym' symbol appearance
            count += 1   # increase count for `sym'
    return count - 1    # remove `sym' count in identifier list

def church_number_to_integer(expr : str | AST) -> int:
    ast = parse(expr) if isinstance(expr, str) else expr
    if not ast.function_p() or not ast.fn_body().function_p():
        raise Exception(f"{ast} not simplified church number value")
    inc, zero = ast.identifier(), ast.fn_body().identifier()
    count = 0
    ast = ast.fn_body().fn_body()
    while True:
        if ast.terminate_p():
            if ast.root == zero:
                return count
            else:
                raise Exception(f"{ast} not {zero}")
        elif ast.application_p():
            if ast.fn_expr().terminate_p() and ast.fn_expr().root == inc:
                count += 1
                ast = ast.arg_expr()
            else:
                print(ast.fn_expr().terminate_p() and ast.fn_expr().root == inc)
                raise Exception(f"{ast} not {inc}")
        else:
            raise Exception(f"{ast} not ({inc}{zero}) like")

if __name__ == "__main__":
    print(church_number_to_integer(input("church number expr > ")))

```