# Strings

```@meta
DocTestSetup = quote
    using ThinkJulia
    using Compat
end
```

Strings are not like integers, floats, and booleans. A string is a **sequence**, which means it is an ordered collection of other values. In this chapter you’ll see how to access the characters that make up a string, and you’ll learn about some of the string helper functions provided by Julia.

## Characters

The characters that English speakers are familiar with are the letters A, B, C, etc., together with numerals and common punctuation symbols. These characters are standardized together with a mapping to integer values between 0 and 127 by the **ASCII standard**.

There are, of course, many other characters used in non-English languages, including variants of the ASCII characters with accents and other modifications, related scripts such as Cyrillic and Greek, and scripts completely unrelated to ASCII and English, including Arabic, Chinese, Hebrew, Hindi, Japanese, and Korean.

The **Unicode standard** tackles the complexities of what exactly a character is, and is generally accepted as the definitive standard addressing this problem.

A `Char` value represents a single character and is surrounded by single quotes:

```jldoctest
julia> 'x'
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)
julia> '🍌'
'🍌': Unicode U+01f34c (category So: Symbol, other)
julia> typeof('x')
Char
```

Emojis are part of the Unicode standard.

## A String Is a Sequence

A string is a sequence of characters. You can access the characters one at a time with the bracket operator:

```jldoctest chap08
julia> fruit = "banana"
"banana"
julia> letter = fruit[1]
'b': ASCII/Unicode U+0062 (category Ll: Letter, lowercase)
```

The second statement selects character number 1 from `fruit` and assigns it to `letter`.

The expression in brackets is called an **index**. The index indicates which character in the sequence you want (hence the name).

All indexing in Julia is 1-based: the first element of any integer-indexed object is found at index 1 and the last element at index `end`:

```jldoctest chap08
julia> fruit[end]
'a': ASCII/Unicode U+0061 (category Ll: Letter, lowercase)
```

As an index you can use an expression that contains variables and operators:

```jldoctest chap08
julia> i = 1
1
julia> fruit[i+1]
'a': ASCII/Unicode U+0061 (category Ll: Letter, lowercase)
julia> fruit[end-1]
'n': ASCII/Unicode U+006e (category Ll: Letter, lowercase)
```

But the value of the index has to be an integer. Otherwise you get:

```jldoctest chap08
julia> letter = fruit[1.5]
ERROR: MethodError: no method matching getindex(::String, ::Float64)
```

## `length`

`length` is a built-in function that returns the number of characters in a string:

```jldoctest chap08
julia> fruits = "🍌 🍎 🍐"
"🍌 🍎 🍐"
julia> len = length(fruits)
5
```

To get the last letter of a string, you might be tempted to try something like this:

```jldoctest chap08
julia> last = fruits[len]
' ': ASCII/Unicode U+0020 (category Zs: Separator, space)
```

But you might not get what you expect.

Strings are encoded using the **UTF-8 encoding**. UTF-8 is a variable-width encoding, meaning that not all characters are encoded in the same number of bytes.

The function `sizeof` gives the number of bytes in a string:

```jldoctest chap08
julia> sizeof("🍌")
4
```

Because an emoji is encoded in 4 bytes and string indexing is byte based, the 5th element of `fruits` is a `SPACE`.

This means also that not every byte index into a UTF-8 string is necessarily a valid index for a character. If you index into a string at such an invalid byte index, an error is thrown:

```jldoctest chap08
julia> fruits[2]
ERROR: StringIndexError("🍌 🍎 🍐", 2)
```

In the case of `fruits`, the character `🍌` is a four-byte character, so the indices 2, 3 and 4 are invalid and the next character's index is 5; this next valid index can be computed by `nextind(fruit, 1)`, and the next index after that by `nextind(fruit, 5)` and so on.

## Traversal with a `for` loop

A lot of computations involve processing a string one character at a time. Often they start at the beginning, select each character in turn, do something to it, and continue until the end. This pattern of processing is called a **traversal**. One way to write a traversal is with a `while` loop:

```julia
index = firstindex(fruits)
while index <= sizeof(fruits)
    letter = fruits[index]
    println(letter)
    index = nextind(fruits, index)
end
```

This loop traverses the string and displays each letter on a line by itself. The loop condition is `index <= sizeof(fruit)`, so when index is larger than the number of bytes in the string, the condition is `false`, and the body of the loop doesn’t run.

The function `firstindex` returns the first valid byte index.

As an exercise, write a function that takes a string as an argument and displays the letters backward, one per line.

Another way to write a traversal is with a `for` loop:

```julia
for letter in fruits
    println(letter)
end
```

Each time through the loop, the next character in the string is assigned to the variable `letter`. The loop continues until no characters are left.

The following example shows how to use concatenation (string multiplication) and a `for` loop to generate an abecedarian series (that is, in alphabetical order). In Robert McCloskey’s book *Make Way for Ducklings*, the names of the ducklings are Jack, Kack, Lack, Mack, Nack, Ouack, Pack, and Quack. This loop outputs these names in order:

```julia
prefixes = "JKLMNOPQ"
suffix = "ack"

for letter in prefixes
    println(letter * suffix)
end
```

The output is:

```@setup chap08
prefixes = "JKLMNOPQ"
suffix = "ack"
```

```@example chap08
for letter in prefixes       # hide
    println(letter * suffix) # hide
end                          # hide
```

Of course, that’s not quite right because “Ouack” and “Quack” are misspelled. As an exercise, modify the program to fix this error.

## String Slices

A segment of a string is called a **slice**. Selecting a slice is similar to selecting a character:

```jldoctest chap08
julia> str = "Julius Caesar";

julia> str[1:6]
"Julius"
```

The operator `[n:m]` returns the part of the string from the “n-eth” byte to the “m-eth” byte. So the same caution is needed as for simple indexing.

The `end` keyword can be used to indicate the last byte of the string:

```jldoctest chap08
julia> str[8:end]
"Caesar"
```

If the first index is greater than the second the result is an **empty string**, represented by two quotation marks:

```jldoctest chap08
julia> str[8:7]
""
```

An empty string contains no characters and has length 0, but other than that, it is the same as any other string.

Continuing this example, what do you think `str[:]` means? Try it and see.

## Strings Are Immutable

It is tempting to use the `[]` operator on the left side of an assignment, with the intention of changing a character in a string. For example:

```jldoctest chap08
julia> greeting = "Hello, world!"
"Hello, world!"
julia> greeting[0] = 'J'
ERROR: MethodError: no method matching setindex!(::String, ::Char, ::Int64)
```

The reason for the error is that strings are **immutable**, which means you can’t change an existing string. The best you can do is create a new string that is a variation on the original:

```jldoctest chap08
julia> greeting = "J" * greeting[2:end]
"Jello, world!"
```

This example concatenates a new first letter onto a slice of greeting. It has no effect on the original string.

## String Interpolation

Constructing strings using concatenation can become a bit cumbersome, however. To reduce the need for these verbose calls to `string` or repeated multiplications, Julia allows **string interpolation** using `$`:

```jldoctest
julia> greet = "Hello"
"Hello"
julia> whom = "World"
"World"
julia> "$greet, $(whom)!"
"Hello, World!"
```

This is more readable and convenient than string concatenation: `greet * ", " * whom * "!"`

The shortest complete expression after the `$` is taken as the expression whose value is to be interpolated into the string. Thus, you can interpolate any expression into a string using parentheses:

```jldoctest
julia> "1 + 2 = $(1 + 2)"
"1 + 2 = 3"
```

## Searching

What does the following function do?

```julia
function find(word, letter)
    index = firstindex(fruits)
    while index <= sizeof(word)
        if word[index] == letter
            return index
        end
        index = nextind(word, index)
    end
    -1
end
```

In a sense, find is the inverse of the `[]` operator. Instead of taking an index and extracting the corresponding character, it takes a character and finds the index where that character appears. If the character is not found, the function returns -1.

This is the first example we have seen of a return statement inside a loop. If `word[index] == letter`, the function breaks out of the loop and returns immediately.

If the character doesn’t appear in the string, the program exits the loop normally and returns -1.

This pattern of computation—traversing a sequence and returning when we find what we are looking for—is called a **search**.

As an exercise, modify `find` so that it has a third parameter, the index in `word` where it should start looking.

## Looping and Counting

The following program counts the number of times the letter a appears in a string:

```julia
word = "banana"
count = 0
for letter in word
    if letter == 'a'
        count = count + 1
    end
end
println(count)
```

This program demonstrates another pattern of computation called a **counter**. The variable `count` is initialized to 0 and then incremented each time an `a` is found. When the loop exits, count contains the result—the total number of `a`’s.

As an exercise, encapsulate this code in a function named `count`, and generalize it so that it accepts the string and the letter as arguments.

Then rewrite the function so that instead of traversing the string, it uses the three-parameter version of `find` from the previous section.

## String Library

Julia provides functions that perform a variety of useful operations on strings. For example, the function `uppercase` takes a string and returns a new string with all uppercase letters.

```jldoctest
julia> uppercase("Hello, World!")
"HELLO, WORLD!"
```

As it turns out, there is a function named `findfirst` that is remarkably similar to the function `find` we wrote:

```jldoctest
julia> findfirst("a", "banana")
2:2
```

Actually, the `findfirst` function is more general than our function; it can find substrings, not just characters:

```jldoctest
julia> findfirst("na", "banana")
3:4
```

By default, `findfirst` starts at the beginning of the string, but the function `findnext` takes a third argument, the `index` where it should start:

```jldoctest
julia> findnext("na", "banana", 4)
5:6
```

This is an example of an **optional argument**.

## The `∈` Operator

The keyword `∈` (`\in TAB`) is a boolean operator that takes a character and a string and returns `true` if the first appears as in the second:

```jldoctest
julia> 'a' ∈ "banana"    # 'a' in "banana"
true
```

For example, the following function prints all the letters from word1 that also appear in word2:

```julia
function inboth(word1, word2)
    for letter in word1
        if letter ∈ word2
            println(letter)
        end
    end
end
```

With well-chosen variable names, Julia sometimes reads like English. You could read this loop, “for (each) letter in (the first) word, if (the) letter is an element of (the second) word, print (the) letter.”

Here’s what you get if you compare `"apples"` and `"oranges"`:

```jldoctest
julia> inboth("apples", "oranges")
a
e
s
```

## String Comparison

The relational operators work on strings. To see if two strings are equal:

```julia
word = "Pineapple"
if word == "banana"
    println("All right, bananas.")
end
```

Other relational operations are useful for putting words in alphabetical order:

```julia
if word < "banana"
    println("Your word, $word, comes before banana.")
elseif word > "banana"
    println("Your word, $word, comes after banana.")
else
    println("All right, bananas.")
end
```

Julia does not handle uppercase and lowercase letters the same way people do. All the uppercase letters come before all the lowercase letters, so:

```julia
Your word, Pineapple, comes before banana.
```

A common way to address this problem is to convert strings to a standard format, such as all lowercase, before performing the comparison. Keep that in mind in case you have to defend yourself against a man armed with a Pineapple.

## Debugging

When you use indices to traverse the values in a sequence, it is tricky to get the beginning and end of the traversal right. Here is a function that is supposed to compare two words and return `true` if one of the words is the reverse of the other, but it contains two errors:

```julia
function isreverse(word1, word2)
    if length(word1) != length(word2)
        return false
    end
    i = firstindex(word1)
    j = lastindex(word2)
    while j >= 0
        j = prevind(word2, j)
        if word1[i] != word2[j]
            return false
        end
        i = nextind(word1, i)
    end
    true
end
```

The first `if` statement checks whether the words are the same length. If not, we can return `false` immediately. Otherwise, for the rest of the function, we can assume that the words are the same length. This is an example of the guardian pattern.

`i` and `j` are indices: `i` traverses `word1` forward while `j` traverses `word2` backward. If we find two letters that don’t match, we can return `false` immediately. If we get through the whole loop and all the letters match, we return `true`.

The function `lastindex` returns the last valid byte index of a string and `prevind` the previous valid index of a character.

If we test this function with the words "pots" and "stop", we expect the return value `true`, but we get `false`:

```@meta
DocTestSetup = quote
    using ThinkJulia
    using Compat

    function isreverse(word1, word2)
        if length(word1) != length(word2)
            return false
        end
        i = firstindex(word1)
        j = lastindex(word2)
        while j >= 0
            j = prevind(word2, j)
            if word1[i] != word2[j]
                return false
            end
            i = nextind(word1, i)
        end
        true
    end
end
```

```jldoctest
julia> isreverse("pots", "stop")
false
```

For debugging this kind of error, my first move is to print the values of the indices:

```julia
    while j >= 0
        j = prevind(word2, j)
        println("$i $j")        # print here
        if word1[i] != word2[j]
```

```@meta
DocTestSetup = quote
    using ThinkJulia
    using Compat

    function isreverse(word1, word2)
        if length(word1) != length(word2)
            return false
        end
        i = firstindex(word1)
        j = lastindex(word2)
        while j >= 0
            j = prevind(word2, j)
            println("$i $j")
            if word1[i] != word2[j]
                return false
            end
            i = nextind(word1, i)
        end
        true
    end
end
```

Now when I run the program again, I get more information:

```jldoctest
julia> isreverse("pots", "stop")
1 3
false
```

The first time through the loop, the value of `j` is 3, which has to be 4. This can be fixed by moving `j = prevind(word2, j)` to the end of the `while` loop.

If I fix that error and run the program again, I get:

```@meta
DocTestSetup = quote
    using ThinkJulia
    using Compat

    function isreverse(word1, word2)
        if length(word1) != length(word2)
            return false
        end
        i = firstindex(word1)
        j = lastindex(word2)
        while j >= 0
            println("$i $j")
            if word1[i] != word2[j]
                return false
            end
            i = nextind(word1, i)
            j = prevind(word2, j)
        end
        true
    end
end
```

```jldoctest
julia> isreverse("pots", "stop")
1 4
2 3
3 2
4 1
5 0
ERROR: BoundsError: attempt to access "pots"
  at index [5]
```

This time a `BoundsError` has been thrown. The value of `i` is 5, which is out a range for the string `"pots"`.

Run the program on paper, changing the values of `i` and `j` during each iteration. Find and fix the second error in this function.

## Glossary

*sequence*:
An ordered collection of values where each value is identified by an integer index.

*ASCII standard*:
A character encoding standard for electronic communication specifying 128 characters.

*Unicode standard*:
A computing industry standard for the consistent encoding, representation, and handling of text expressed in most of the world's writing systems.

*index*:
An integer value used to select an item in a sequence, such as a character in a string. In Julia indices start from 1.

*UTF-8 encoding*:
A variable width character encoding capable of encoding all 1112064 valid code points in Unicode using one to four 8-bit bytes.

*traverse*:
To iterate through the items in a sequence, performing a similar operation on each.

*slice*:
A part of a string specified by a range of indices.

*empty string*:
A string with no characters and length 0, represented by two quotation marks.

*immutable*:
The property of a sequence whose items cannot be changed.

*string interpolation*:
The process of evaluating a string containing one or more placeholders, yielding a result in which the placeholders are replaced with their corresponding values.

*search*:
A pattern of traversal that stops when it finds what it is looking for.

*counter*:
A variable used to count something, usually initialized to zero and then incremented.

*optional argument*:
A function argument that is not required.

## Exercises

### Exercise 8-1

Read the documentation of the string functions at <https://docs.julialang.org/en/stable/stdlib/strings/>. You might want to experiment with some of them to make sure you understand how they work. `strip` and `replace` are particularly useful.

The documentation uses a syntax that might be confusing. For example, in `search(string::AbstractString, chars::Chars, [start::Integer])`, the brackets indicate optional arguments. So `string` and `chars` are required, but `start` is optional.

### Exercise 8-2

There is a builtin function called `count` that is similar to the function in Section 8.9. Read the documentation of this function and use it to count the number of `a`’s in "banana".

### Exercise 8-3

A string slice can take a third index. The first specifies the start, the third the end and the second the “step size”; that is, the number of spaces between successive characters. A step size of 2 means every other character; 3 means every third, etc.

```jldoctest
julia> fruit = "banana"
"banana"
julia> fruit[1:2:6]
"bnn"
```

A step size of -1 goes through the word backwards, so the slice `[end:-1:1]` generates a reversed string.

Use this idiom to write a one-line version of `ispalindrome` from Exercise 6.3.

### Exercise 8-4

The following functions are all *intended* to check whether a string contains any lowercase letters, but at least some of them are wrong. For each function, describe what the function actually does (assuming that the parameter is a string).

```julia
function anylowercase1(s)
    for c in s
        if islower(c)
            return true
        else
            return false
        end
    end
end

function anylowercase2(s)
    for c in s
        if islower('c')
            return "true"
        else
            return "false"
        end
    end
end

function anylowercase3(s)
    for c in s
        flag = islower(c)
    end
    flag
end

function anylowercase4(s)
    flag = false
    for c in s
        flag = flag || islower(c)
    end
    flag
end

function anylowercase5(s)
    for c in s
        if !islower(c)
            return false
        end
    end
    true
end
```

### Exercise 8-5

A Caesar cypher is a weak form of encryption that involves “rotating” each letter by a fixed number of places. To rotate a letter means to shift it through the alphabet, wrapping around to the beginning if necessary, so `’A’` rotated by 3 is `’D’` and `’Z’` rotated by 1 is `’A’`.

To rotate a word, rotate each letter by the same amount. For example, `"cheer"` rotated by 7 is `"jolly"` and `"melon"` rotated by -10 is `"cubed"`. In the movie *2001: A Space Odyssey, the ship computer* is called HAL, which is IBM rotated by -1.

Write a function called `rotateword` that takes a string and an integer as parameters, and returns a new string that contains the letters from the original string rotated by the given amount.

You might want to use the built-in function `Int`, which converts a character to a numeric code, and `Char`, which converts numeric codes to characters. Letters of the alphabet are encoded in alphabetical order, so for example:

```jldoctest
julia> Int('c') - Int('a')
2
```

Because `'c'` is the third letter of the alphabet. But beware: the numeric codes for uppercase letters are different.

```jldoctest
julia> Char(Int('A') + 32)
'a': ASCII/Unicode U+0061 (category Ll: Letter, lowercase)
```

Potentially offensive jokes on the Internet are sometimes encoded in ROT13, which is a Caesar cypher with rotation 13. If you are not easily offended, find and decode some of them.