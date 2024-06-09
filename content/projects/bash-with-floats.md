---
title: "Bash With Floats"
subtitle: "A fork of Bash with support for floating-point operations (C)"
notability: 5
loc: 1400
date: 2023-08-15
---

### ([Github Link](https://github.com/lucasscharenbroch/bash-with-floats))

## The Idea

I recently finished reading the [Advanced Bash Scripting Guide (ABS)](https://tldp.org/LDP/abs/html/), which, as the name suggests, dives deep into the darkest corners of bash's functionality. It contains a wide variety of scripts to demonstrate these capabilities; here are a few examples.

- [Calculating Pi](https://tldp.org/LDP/abs/html/mathc.html#CANNON)
- [The Fibonacci Sequence (recursion)](https://tldp.org/LDP/abs/html/recurnolocvar.html#FIBO)
- [Insertion Sort](https://tldp.org/LDP/abs/html/contributed-scripts.html#INSERTIONSORT)
- [The Sieve of Eratosthenes](https://tldp.org/LDP/abs/html/arrays.html#EX68)
- [The Knight's Tour](https://tldp.org/LDP/abs/html/contributed-scripts.html#KTOUR)

These scripts are admittedly useless (not to mention inefficient), but that's not the point: the goal is to demonstrate that such functionality is *possible* in a script, so that the reader is sufficiently equipped in the rare case where these features come in handy (namely when making a dedicated program would be too inconvenient or inflexible, or when a script is already involved).

Bash has good support for integer arithmetic, and it comes up surprisingly often. There are several ways to work with integers in bash (many of which turn out to be different ways of using the same underlying mechanisms), but the most convenient way is [arithmetic expansion keyword](https://tldp.org/LDP/abs/html/arithexp.html): "**(())**", and its substitution variant: "**$(())**". It supports all of the usual C-style integer operations (+, -, *, /, %, ** (exponentiation), ++, -\-, &&, ||, !, ==, !=, >, <, >=, <=, ?: (ternary), ~, &, |, ^, <<, >>, =, *=, /=, +=, -=, <<=, >>=, &=, ^=, |=, comma (separates independent expressions)), and variables can be referenced and modified with the same C-syntax (without requiring quoting or substitutions).

There's even a C-style for loop construction that uses the arithmetic expansion[^arith-for].

[^arith-for]: This construction turns out to be hard-coded under the hood (there isn't some underlying property of for-loops that allows this).

{{< highlight bash >}}
for ((i = 0; i < 100; i++)) do
    echo "i = $i"
done
{{< /highlight >}}

Bash doesn't have native support for floating-point variables, though: this is presumably because they aren't nearly as useful as integers, and they're much less elegant to work with in their string form (*all variables* in bash are stored as strings (or 1-d "arrays" of strings)). Thus the shell programmer must resort to using external programs (like bc, python, perl, or awk), which results in messy syntax.

For example, say we want to find the average run-time of some arbitrary command, and print a warning if it exceeds a certain threshold. Let's do it with floats[^clean].

[^clean]: A cleaner way would be to use an arbitrarily small time unit.

{{< highlight bash "linenos=table,hl_lines=12-14 17" >}}
#!/bin/bash

cmd="sleep .1"   # command to run
n=10             # number of rums
total=0          # total seconds across all runs
TIMEFORMAT=%R    # set the output format for the `time' comand
THRESHOLD=".105" # average time above which a warning is printed

for ((i = 1; i <= n; i++))
do
    t=$({ time $cmd > /dev/null; } 2>&1 )
    total=$(python -c "print($total + $t)")
    avg=$(python -c "print($total / $i)")
    echo "Average over $i runs: $avg"
done

if (( $(echo "$avg > $THRESHOLD" | bc) )); then
    echo "Warning: Bash is sleepy today"
fi
{{< /highlight >}}

Note the floating-point operations in lines 12, 13, and 17. They require a command-substitution ("***$(...)***"), a call to an external program (*python* or *bc*[^py_bc]), quotes, a print-statement, and variable substitutions. Perhaps this isn't *that bad*, but wouldn't it be convenient[^speedy] if we could do something like this instead?

[^py_bc]: Either program could be used for all of the floating-point operations in this example; I used both for the sake of demonstration.

[^speedy]: It's worth noting that using external programs like *python* and *bc* requires that another process be forked and a lot of initialization to be done (especially in the case of python), which is *very* sluggish in comparison to using shell builtins. I tested the two scripts above (the example with *python* and *bc* and the example with float-expansion), and they ran in 2.188 and 1.125 seconds, respectively; subtracting off the time for the "time $cmd" commands (the average run-time for the sleep command was 0.1049 and 0.1072 respectively), the run-times were 1.139 and 0.053 (!) seconds, respectively.

{{< highlight bash "linenos=table,hl_lines=4 7,linenostart=9" >}}
for ((i = 1; i <= n; i++))
do
    t=$({ time $cmd > /dev/null; } 2>&1 )
    echo "Average over $i runs: ${{avg = (total += t) / i}}"
done

if {{avg > THRESHOLD}}; then
    echo "Warning: bash is sleepy today"
fi
{{< /highlight >}}

In short, I want a "**{{}}**"[^dbrace] operator for floats that is analagous to "**(())**" for integers. It would also be nice to have a "**${{}}**" (substitution) version, and a floating-point variable "type"[^type].

[^dbrace]: The choice of the double-brace operator is completely arbitrary. I chose it because all of the other double-paren operators are in use ("(())" and "[[]]"), and the float-expansion is very similar to the arithmetic expansion ("(())"), and it shares some properties with the extended test ("[[]]").

[^type]: As mentioned above, all variables in Bash are strings or arrays of strings, but they also have "types", which are flags that can be set to tell the shell how to handle assignments and other variable operations. These include "readonly", "function", and "integer" (see "help declare" for more details). A variable need not be declared as integer/float to be used in arithmetic/float expansions, but if the variable is declared as int/float, then *variable assignments* (=, +=) are treated as arithmetic/float expansions.

{{< highlight bash >}}
#!/bin/bash

                       # note that x isn't declared as float yet;
                       # this doesn't change the script's behavior

{{ x = 5 / 2 }}
echo x = ${{ x += 1 }} # output: "x = 3.500000"

unset x

declare -d x="5 / 2"   # "d" stands for "double", as in "double-precision floating point"
                       # (the f and F options were already taken)
x+=1
echo x = $x            # output: "x = 3.500000"
{{< /highlight >}}
Note that, by default, the output of float expansions and substitutions are 6 digits after the decimal place. This can be changed by setting the FLOAT_DIGITS shell variable to the desired number of digits.

## The Execution

This project was almost entirely an exercise in understanding bash's codebase. The vast majority of the lines I wrote were either closely mimicking already-written lines (I especially rode off the back of the arithmetic expansion (***(())***) operator) or directly copy-and-pasted with minimal modification. This was by no means a trivial task, however.

## Bash Code

As one might expect, bash is written entirely in C. It uses (what I would call) an old-fashioned style that involves vertically-aligned opening and closing braces, 8-chars-wide tabs mixed with spaces for indentation, and old-style parameter lists, among other things[^style]. It makes heavy use of macros, preprocessor conditional statements, and memory management. The base-directory of the repository has 125 files (most of the main source-code is in the base-directory).

Below are a few examples that I found interesting or amusing that will hopefully provide an idea for what working with this codebase is like. The line-numbers correspond to my version of the repository at the time of writing this post[^hash].

[^hash]: The commit hash is e79f07bbd996f2c3385ef0b64c410a9e3e3bc0dc.

{{< highlight c "linenos=table">}}
/* hashlib.c -- functions to manage and access hash tables for bash. */
{{< /highlight >}}

{{< highlight c "linenos=table,linenostart=37" >}}
/* tunable constants for rehashing */
#define HASH_REHASH_MULTIPLIER	4
#define HASH_REHASH_FACTOR	2

#define HASH_SHOULDGROW(table) \
  ((table)->nentries >= (table)->nbuckets * HASH_REHASH_FACTOR)

/* an initial approximation */
#define HASH_SHOULDSHRINK(table) \
  (((table)->nbuckets > DEFAULT_HASH_BUCKETS) && \
   ((table)->nentries < (table)->nbuckets / HASH_REHASH_MULTIPLIER))

/* Rely on properties of unsigned division (unsigned/int -> unsigned) and
   don't discard the upper 32 bits of the value, if present. */
#define HASH_BUCKET(s, t, h) (((h) = hash_string (s)) & ((t)->nbuckets - 1))
{{< /highlight >}}

{{< highlight c "linenos=table,linenostart=198" >}}
/* This is the best 32-bit string hash function I found. It's one of the
   Fowler-Noll-Vo family (FNV-1).

   The magic is in the interesting relationship between the special prime
   16777619 (2^24 + 403) and 2^32 and 2^8. */

#define FNV_OFFSET 2166136261
#define FNV_PRIME 16777619

/* If you want to use 64 bits, use
FNV_OFFSET	14695981039346656037
FNV_PRIME	1099511628211
*/

/* The `khash' check below requires that strings that compare equally with
   strcmp hash to the same value. */
unsigned int
hash_string (s)
     const char *s;
{
  register unsigned int i;

  for (i = FNV_OFFSET; *s; s++)
    {
      /* FNV-1a has the XOR first, traditional FNV-1 has the multiply first */

      /* was i *= FNV_PRIME */
      i += (i<<1) + (i<<4) + (i<<7) + (i<<8) + (i<<24);
      i ^= *s;
    }

  return i;
}
{{< /highlight >}}

The collision-handling strategy is chaining: "BUCKET" refers to a slot in the table, and "BUCKET_CONTENTS" is a linked-list-node. I thought that the "HASH_BUCKET" macro (which determines which bucket a string belongs in) was pretty clever: since "(t)->nbuckets" is always a power of 2, "(t)->nbuckets - 1" can be used as a bit-mask to bring any number into the range [0, (t)->nbuckets). No need for modulo.

{{< highlight c "linenos=table" >}}
/* parse.y - Yacc grammar for bash. */
{{< /highlight >}}

{{< highlight c "linenos=table,linenostart=2622" >}}
#ifndef OLD_ALIAS_HACK
  if (uc == 0 && pushed_string_list && pushed_string_list->flags != PSH_SOURCE &&
      pushed_string_list->flags != PSH_DPAREN &&
      (parser_state & PST_COMMENT) == 0 &&
      (parser_state & PST_ENDALIAS) == 0 &&	/* only once */
      shell_input_line_index > 0 &&
      shellblank (shell_input_line[shell_input_line_index-1]) == 0 &&
      shell_input_line[shell_input_line_index-1] != '\n' &&
      unquoted_backslash == 0 &&
      shellmeta (shell_input_line[shell_input_line_index-1]) == 0 &&
      (current_delimiter (dstack) != '\'' && current_delimiter (dstack) != '"'))
    {
      parser_state |= PST_ENDALIAS;
      /* We need to do this to make sure last_shell_getc_is_singlebyte returns
	 true, since we are returning a single-byte space. */
      if (shell_input_line_index == shell_input_line_len && last_shell_getc_is_singlebyte == 0)
	{
#if 0
	  EXTEND_SHELL_INPUT_LINE_PROPERTY();
	  shell_input_line_property[shell_input_line_len++] = 1;
	  /* extend shell_input_line to accommodate the shell_ungetc that
	     read_token_word() will perform, since we're extending the index */
	  RESIZE_MALLOCED_BUFFER (shell_input_line, shell_input_line_index, 2, shell_input_line_size, 16);
          shell_input_line[++shell_input_line_index] = '\0';	/* XXX */
#else
	  shell_input_line_property[shell_input_line_index - 1] = 1;
#endif
	}
      return ' ';	/* END_ALIAS */
    }
#endif
{{< /highlight >}}

The longest conditional I've ever seen.

{{< highlight c "linenos=table" >}}
/* variables.c -- Functions for hacking shell variables. */
{{< /highlight >}}

{{< highlight c "linenos=table,linenostart=6025" >}}
/* The variable in NAME has just had its state changed.  Check to see if it
   is one of the special ones where something special happens. */
void
stupidly_hack_special_variables (name)
     char *name;
{
  static int sv_sorted = 0;
  int i;

  if (sv_sorted == 0)	/* shouldn't need, but it's fairly cheap. */
    {
      qsort (special_vars, N_SPECIAL_VARS, sizeof (special_vars[0]),
		(QSFUNC *)sv_compare);
      sv_sorted = 1;
    }

  i = find_special_var (name);
  if (i != -1)
    (*(special_vars[i].function)) (name);
}
{{< /highlight >}}

An honest function name.

{{< highlight c "linenos=table" >}}
/* variables.h -- data structures for shell variables. */
{{< /highlight >}}

{{< highlight c "linenos=table,linenostart=67,hl_lines=8 10" >}}
/* For the future */
union _value {
  char *s;			/* string value */
  intmax_t i;			/* int value */
  COMMAND *f;			/* function */
  ARRAY *a;			/* array */
  HASH_TABLE *h;		/* associative array */
  double d;			/* floating point number */
#if defined (HAVE_LONG_DOUBLE)
  long double ld;		/* long double */
#endif
  struct variable *v;		/* possible indirect variable use */
  void *opaque;			/* opaque data for future use */
};
{{< /highlight >}}

Are there plans to add floats to the official version of bash? It seems unlikely; this union has been around since at least 2009. The struct that is actually used for variables is below: note that "char *" is the only possible type for "value".

{{< highlight c "linenos=table,linenostart=82,hl_lines=2" >}}
typedef struct variable {
  char *name;                         /* Symbol that the user types. */
  char *value;                        /* Value that is returned. */
  char *exportstr;                    /* String for the environment. */
  sh_var_value_func_t *dynamic_value; /* Function called to return a `dynamic'
                                         value for a variable, like $SECONDS
                                         or $RANDOM. */
  sh_var_assign_func_t *assign_func;  /* Function called when this `special
                                         variable' is assigned a value in
                                         bind_variable. */
  int attributes;                     /* export, readonly, array, invisible... */
  int context;                        /* Which context this variable belongs to. */
} SHELL_VAR;
{{< /highlight >}}

[^style]: I've tried to maintain this style in the code I added, and it has grown on me.

## Infrastructure

{{% center-text %}}
<img src="/images/bash-article-diagram.jpg" alt="Anki in Curses"/>
{{% /center-text %}}

Here's the big picture of how bash works. It's from [Architecture of Open Source Applications (AOSA)](https://aosabook.org/en/v1/bash.html), whose bash chapter was written by Chet Ramey, the primary developer of bash. Input is read (from the command line[^readline] or a script), [lexed](https://en.wikipedia.org/wiki/Lexical_analysis) and [parsed](https://en.wikipedia.org/wiki/Parsing), expanded (in the many ways described above), then executed.

[^readline]: Readline ([The GNU Readline Library](https://tiswww.case.edu/php/chet/readline/rltop.html)) is used for this: is technically a part of bash: it is a library for interaction with and customization of the "command line".

Each of these steps has a considerable amount of nuance. Parsing is done in [yacc/bison](https://en.wikipedia.org/wiki/Yacc)[^yacc], but the syntactic context of the tokens can change their meaning, so yylex is written manually (instead of using lex/flex). The lexing functions contain the majority of the logic for this step, and there's even a recursive-descent parser written in parse.y to specifically handle the double-bracket test construct ("***[[]]***").

[^yacc]: In AOSA, Chet says that he would use recursive-descent instead of yacc/bison if he were to re-write bash.

## Implementation

Adding support for the "**{{}}**" and "**${{}}**" operators requires modifying each of the three above steps.

- The text enclosed within "**{{...}}**" need not be space-separated (i.e. "{{1}}" is valid), so "{{" cannot be a keyword (unlike "[["); additionally, the text within the "**{{}}**" construct is treated as if it were double-quoted. Thus the entire construct should be parsed as a single token.
- The "**{{}}**" command itself needs to run expansion on its contents, and the "**${{}}**" needs to expand[^comsub] to the floating-point-expression's result.
- The "**{{}}**" command needs to have a function to execute it.

[^comsub]: "**${{}}**" can't be parsed exactly like "**$(())**", because the latter is actually parsed as a command substitution ("**$()**"); upon seeing the second opening parenthesis, the behavior changes to parse a arithmetic substitution (unless the closing parenthesis isn't a double-parenthesis, in which case it's interpreted as a subshell within a command substitution).

I also tweaked some of the behavior of assigning variables and the "declare" builtin to add support for the floating-point variable-type.

The evaluation of the floating-point expressions (evaluating the "..." in "{{...}}" ) is a recursive-descent parser that is copied from the arithmetic expansion parser, with the bitwise operators removed and the lexing procedure for a number-token changed.

## Is it Useful?
No, not really. Even if I had more than a handful of uses for floats in bash, using this modified version of bash is bad practice and makes my scripts completely non-portable. That being said, the alternative to float-expansions is verbose and slow. It all comes back to the initial question: need this functionality be in a script in the first place? And perhaps a better question: should it be in bash?
