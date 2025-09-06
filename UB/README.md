This is a written excerpt of the workshop I did for the 42 Vienna students on the 25th of June, 2025.

You have found very important information. Please read further.

## Why this topic?

This is a topic none of the projects we do at 42 deal with. You can complete all of your Common Core without ever reading up on it (you will likely encounter it in practice, though it will likely remain unnoticed).

This is because the curriculum in 42 in general leaves it up to the individual if they want to look into theory or not. I agree with the sentiment that a lot of theory universities are teaching is redundant, and you will not necessarily need it on the job market. But at the same time I firmly belive **there is some theory one can simply not afford to skip**, and Undefined Behavior is one such topic when it comes to C language (and C++).

Even if most of us will not actually work with C or C++, after 2 years of Common Core, we leave the school claiming we *know* these languages. You can, however, have any hours of coding experience in any of them - there is a bad news for you. If you don't know about Undefined Behavior, **you will simply not know what you are doing** when writing code in these languages. And it's not only that you will not know what you are doing, but what is even worse: you will **think** you know what you are doing, and that makes you very dangerous. Instead of an asset, it makes you a liability.

What I said during my workshop is that instead of C programmers, 42 in this sense actually turns us into weapons. Any one of us could be responsible for next CrowdStrike incident or similar. Therefore we can't afford not to talk about this topic.

My goal with the workshop (and providing this transcript) is for students to have some basic understanding of what UB is, so this can be discussed during evaluations, online on the Slack or Discord, etc. It is the responsibility of all students who listen to or read this to turn this into a discussion and further spread the knowledge.

## Part I - The C Standard

In order to understand the concept of Undefined Behavior, we have to talk about **the C Standard** first.

C language was invented in the 1970s by one guy named Dennis Ritchie. As it became very popular there was the need to standardize it - the first such standard was the ANSI C standard in 1989. Today this is the job of the International Organization for Standardization (ISO) who come up with a new C standard every couple of years.

In my presentation I have provided some excerpts to show how the C standard looks like on the inside. You can also find some versions on the internet (LINK) if you are interested, but it's a very dry document to read.

The thing is, one doesn't need to read the C Standard to be decent at writing C language. YouTube tutorials or other learning specific material is probably better suited for this case.

**Who really needs to know the C Standard to the letter though are the guys who write the C compiler.** Since the compiler is the program which will parse your `.c` files in order to be able to process them, it needs to know **exactly what** counts as valid C syntax and what is not acceptable. Remember this because it will be very useful in understanding UB later on.

### The concept of "behavior"

The C Standard describes different types of *behavior*. Behavior itself is described as "external appearance or action". This concept mostly comes in handy when the compiler needs to decide how to implement a specific part of your code *in practice*.

For example, there are different kind of instructions a computer with our architecture (x86-64) can use for a right bitshift, but only one operator for the same (`>>`) in C. Since the different instructions behave a little bit differently (regarding the preservation of the sign bit for example), the compiler needs to make a choice about which one to use. The C standard specifies **implementation-defined behavior** for this case: different compilers might decide to implement different methods of bitshifting, but they need to document their choice in each case.

You can find the definitions for each of the standard-described behaviors in my presentation, but here we care about a specific one: **undefined behavior**. It is described in the standard as the following:

> behavior, upon use of a nonportable or erroneous program construct or of erroneous data, for which this International Standard **imposes no requirements.**

"Imposes no requirements" here means: if the compiler encounters something in your code that falls under this category, there is absolutely no requirements as to *how it should translate it*. Or as the standard says:

> Possible undefined behavior ranges from ignoring the situation
completely with unpredictable results, to behaving during translation or
program execution in a documented manner characteristic of the
environment (with or without the issuance of a diagnostic message), to
terminating a translation or execution (with the issuance of a diagnostic
message).

Moreover, the compiler can choose to assume that it will never encounter any of the such in your code, but this is a bit more complicated concept, so we will talk about this in detail later.

### Common examples

*What exactly* counts as undefined behavior is surprisingly well-defined in the C Standard. There is a specific list around the end of it, which collects all of the examples for UB mentioned in the previous sections, and this list goes on for about 6 pages.

Some arbitrary examples from this list that you might recognize from your C projects:
- The operand of the unary `*` operator has an invalid value.
    - this could be a null pointer,
    - an inappropriately aligned address,
    - or the address of an object after the end of its lifetime ("use after free" belongs here).
- The value of the second operand of the `/` or `%` operator is zero.
- The program attempts to modify a string literal.
- The program encounters signed integer overflow.
- A nonempty source file does not end in a new-line character (yes that is right)

You can find more examples in my presentation (or even more in the standard itself) if you are interested.

So, the answer to *what is Undefined Behavior?* is quite simple: UB is everything in your code which translation is either not defined in the standard at all, or is *defined explicitly as undefined.*

## Part II - Is UB the same as Segmentation Fault?

I purposefully began the second part of my presentation with a question that seems very stupid at first glance. I have noticed that UB in the context of 42 curriculum (especially around Libft and the "NULL protection debate") is often discussed as "something in your code that **results in** Segmentation Fault". If you look back at the **common examples** list is just presented, you can indeed notice that if you put these in your code in practice, most of it will likely produce a segfault. But none of us ever got segfault on integer overflow or omitting a newline from the end of a file, so the answer to the question in the title seems quite straightforward.

But I believe this question, when asked by someone, usually has a deeper meaning behind it. And I think the real question being asked here is:

**Can architecture define something that is undefined by the Standard?**

We have seen earlier that the C Standard leaves it open what exactly happens if you dereference `NULL` in your code. But we all know what happens in practice, if we do it. That is because our architecture, which does not let us access the memory address 0, seems to anyway define what happens. Signed integer overflow is another good example for this. The standard might leave it open, but in practice, `INT_MAX` overflows into `INT_MIN`, and it cannot seem to happen any different way on our machines.

So does architecture have the last word in what happens? In order to be able to answer this question, we have to look into a concept called **the abstract machine**.

### The Abstract Machine

Probably all of you have heard by now that C is a **compiled language**. In the first week of Piscine we learned that when we `cc hehe.c` it will get translated to this mysterious file `a.out` that is called the *binary*.

Just like you would parse a config file for `cub3D` for example, the compiler is the program that will open your `.c` file, read its contents, and evenutally create this binary file based on what you want the program to accomplish. This fact has one - very important - consequence: **the code that will actually end up running on your machine is not written by you**. Your C code will never "run", it is just a blueprint that gives the compiler an idea of what to accomplish, but the compiler takes over from there.

The C Standard uses the concept of an **abstract machine** to describe this relationship between your code and the executable. If you think you don't know what an abstract machine is, you are probably wrong about that. Abstract machines are present in a lot of grade school math workbooks. They look like the following:

They are abstract because the inner workings of the machine are not described in detail. We know the *rule*: a circle goes in and a square comes out, but what happens in between, *how* the transformation is achieved doesn't really concern us. We don't know anything about the hardware (cogs? registers? unicorns? a data bus?) but we don't need to care.

The Standard tells the compiler-writers that what they put into the byte-code that fills the binary does not really matter as long as the executable's **observable behavior** remains the same as you intended. More precisely, they say:

> 5.1.2.3.6. At program termination, all data written into ﬁles shall be
identical to the result that execution of the program according to the
abstract semantics would have produced.

The "abstract semantics" they mention is your C code. This means your code itself is nothing but an abstraction defining *"I want all circles to be turned into squares."* Now it's the compiler's job to write some code to accomplish that.

> 5.1.2.3.9. „An implementation might deﬁne a one-to-one correspondence between
abstract and actual semantics: at every sequence point, the values of the actual
objects would agree with those speciﬁed by the abstract semantics. The keyword
volatile would then be redundant.<br>
5.1.2.3.10. Alternatively, an implementation **might perform various optimizations**
within each translation unit, such that the actual semantics would agree with the
abstract semantics only when making function calls across translation unit
boundaries.”

This means: if the compiler wants to, it can follow the circle-to-square process *you* describe in *your* code step-by-step (as closely as the limitations of architecture allow). **But it doesn't have to**. As long as it sees you intended all circles to become squares and it achieves the same, it's good to go - even if the two implementations follow very different logic.

### How is this all relevant to UB?

Remember the definition of UB:

> Behavior, upon use of a nonportable or erroneous program construct or
of erroneous data, for which this International Standard **imposes no
requirements**.

When the compiler encounters UB in your code, there is absolutely no requirements as to what it should translate it into. You have probably heard the phrase before that it can write code that "formats your hard drive". While it won't theoretically, it really can. It is **free to do anything**, and is not even required to document it.

The tiny executable I have created for my presentation is a good example: in the C code, we dereference a `NULL` ptr in a loop, and then (in case we somehow survive) return 42. Compiled with `gcc`, our expectations are matched: we get Segmentation Fault. Compiled with `clang` however, nothing observable happens - we can also check that, for some strange reason, our program returned with a value of 48. And the point of UB is, that *this is fine.*

There is a very good summary of this danger in John Regehr's [Guide to Undefined Behavior](https://blog.regehr.org/archives/213):

> C and C++ are **unsafe in a strong sense**: executing an erroneous operation **causes the entire program to be meaningless**, as opposed to just the erroneous operation having an unpredictable result.

As soon as you put UB anywhere in your code, circles don't need to be turned into squares anymore. They can be turned into anything. And remember that there is a *6 page long* list in the C standard about all the such things you should avoid. Most of them we probably don't even know. So this gives legitimacy of the picture I started my presentation with:

[should put the picture /and credit/ here]

## Part III. Why does UB exist?

## Part IV. Detecting Undefined Behavior

## Part V. UB @ 42

## Resources for further reading

## Footnotes

## TLDR for the lazy

