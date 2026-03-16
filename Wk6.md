You need to check whether the file has **exactly this permission**:

```
-r--------
```

Meaning:

* Owner: **read only**
* Group: **no permissions**
* Others: **no permissions**

Octal form:

```
400
```

---

## Simple Script

```bash
#!/bin/bash

perm=$(stat -c "%a" "$1")

if [ "$perm" = "400" ]; then
    echo Yes
fi
```

---

## Explanation

`stat -c "%a"`
→ prints the **numeric permission** of the file.

Example:

```
-r--------  → 400
-rw-------  → 600
-r--r--r--  → 444
```

So the script prints **Yes only when permission = 400**.

---

## Example

```
chmod 400 file.txt
./script.sh file.txt
```

Output

```
Yes
```

If permissions change:

```
chmod 600 file.txt
```

Output

```
(no output)
```

---

## Alternative (pure bash test style)

If you want to avoid `stat`:

```bash
#!/bin/bash

perm=$(ls -l "$1" | awk '{print $1}')

if [ "$perm" = "-r--------" ]; then
    echo Yes
fi
```

But **the `stat` solution is safer and cleaner**.

Your command has a few **syntax issues**. Let's examine and correct it.

Your command:

```bash
[ $($(stat -c "%a" "$1") == "400")] && echo Yes || echo No
```

### Problems

1. **Wrong command substitution**

   ```bash
   $($(stat ...))
   ```

   `$(...)` already performs command substitution.
   You do **not** wrap it again with `$()`.

2. **Spaces inside `[` `]`**

   Bash requires spaces:

   ```bash
   [ condition ]
   ```

3. **Comparison must be inside the test**

---

# Correct Version

```bash
[ "$(stat -c "%a" "$1")" = "400" ] && echo Yes
```

This prints **Yes only when permission = 400**, otherwise prints nothing (which matches the question).

---

# If you want Yes/No

```bash
[ "$(stat -c "%a" "$1")" = "400" ] && echo Yes || echo No
```

---

# Cleaner Bash Version (`[[ ]]`)

```bash
[[ $(stat -c "%a" "$1") == 400 ]] && echo Yes
```

`[[ ]]` is preferred in modern bash because:

* safer quoting
* better comparisons
* fewer syntax pitfalls.

---

✅ **Most exam-style minimal answer**

```bash
[[ $(stat -c "%a" "$1") == 400 ]] && echo Yes
```


Good question. The key is that **the shell parses the command in stages**, and the quotes belong to **different contexts**.

Your command:

```bash
[ "$(stat -c "%a" "$1")" = "400" ] && echo Yes
```

Let’s see how bash interprets it.

---

# 1. Shell Parsing Rule

Bash reads the command **left to right** and applies:

1. **Quote grouping**
2. **Command substitution**
3. **Parameter expansion**
4. **Word splitting**

The important idea:
**Quotes only affect the level where they appear.**

---

# 2. Outer Quotes vs Inner Quotes

Look at this portion:

```bash
"$(stat -c "%a" "$1")"
```

There are **two quoting levels**:

| Level               | Purpose                                      |
| ------------------- | -------------------------------------------- |
| `"` around `$(...)` | protect the final result from word splitting |
| `"` inside `stat`   | protect `%a` and `$1` arguments              |

The shell treats the **command substitution separately**.

---

# 3. Step-by-Step Execution

Suppose:

```bash
$1 = file.txt
```

### Step 1 — Detect command substitution

```bash
$(stat -c "%a" "$1")
```

The shell runs this **as a separate command**.

---

### Step 2 — Parse the inner command

Inside the substitution the shell sees:

```bash
stat -c "%a" "$1"
```

Here:

* `"%a"` → argument to `-c`
* `"$1"` → filename

So it runs:

```bash
stat -c %a file.txt
```

Example output:

```
400
```

---

### Step 3 — Substitute result

The command becomes:

```bash
[ "400" = "400" ] && echo Yes
```

The **outer quotes** keep `400` as a single word.

---

### Step 4 — Test command runs

```bash
[ "400" = "400" ]
```

This evaluates **true**, so:

```
Yes
```

---

# 4. Why Quotes Don't Conflict

The reason this works is:

**Command substitution creates a new parsing context.**

Think of it like this:

```
outer shell
   └── run command substitution
           └── inner shell parsing
```

So these quotes belong to **different levels**.

---

# 5. Simplified Mental Model

Visual grouping:

```
[  "$(  stat -c "%a" "$1"  )"  =  "400"  ]
```

or:

```
test
 ├─ argument1 → "$(stat -c "%a" "$1")"
 ├─ operator  → =
 └─ argument2 → "400"
```

---

# 6. Key Rule to Remember

Whenever you see:

```
"$( ... )"
```

* The **inside is parsed first**
* The **result is inserted**
* The **outer quotes apply to the result**

---

✅ **Good habit in bash**

Always write:

```bash
"$(command)"
```

instead of

```bash
$(command)
```

because it prevents **word splitting and globbing bugs**.

---

If you'd like, I can also show a **very surprising quoting example that breaks many experienced shell programmers**—it reveals a subtle rule in bash parsing.


Your command has several **syntax and conceptual errors**. Let's analyze it and then show the correct way.

Your command:

```bash
["$( "$#" = "2" )" ]&& echo "$($1 + $2)" || echo Error
```

---

# 1. Problems in the Command

### Problem 1 — Missing spaces around `[ ]`

In bash, `[` is actually a **command (`test`)**, so spaces are required.

Wrong:

```bash
["something"]
```

Correct:

```bash
[ "something" ]
```

---

### Problem 2 — Incorrect command substitution

You wrote:

```bash
"$( "$#" = "2" )"
```

`$(...)` runs a **command**, but:

```bash
"$#" = "2"
```

is **not a command**. It is just a comparison expression.

So command substitution is invalid here.

---

### Problem 3 — Arithmetic expansion

You wrote:

```bash
$($1 + $2)
```

This attempts to run a **command** named `3 + 4` etc.

Arithmetic must use:

```bash
$(( ... ))
```

---

# 2. Correct Script

If the goal is:

* If **exactly two arguments**
* print their **sum**
* otherwise print **Error**

Correct version:

```bash
[ "$#" = "2" ] && echo $(($1 + $2)) || echo Error
```

---

# 3. Cleaner Version

```bash
[ "$#" -eq 2 ] && echo $(( $1 + $2 )) || echo Error
```

Explanation:

| Part     | Meaning              |
| -------- | -------------------- |
| `$#`     | number of arguments  |
| `-eq`    | numeric equality     |
| `$(( ))` | arithmetic expansion |

---

# 4. Example

Run:

```bash
./script.sh 3 5
```

Output:

```
8
```

Run:

```bash
./script.sh 3
```

Output:

```
Error
```

---

# 5. Even Cleaner Bash Style

```bash
(( $# == 2 )) && echo $(( $1 + $2 )) || echo Error
```

Uses **bash arithmetic test**.

---

✅ **Minimal correct one-liner**

```bash
(( $# == 2 )) && echo $(( $1 + $2 )) || echo Error
```

---

If you'd like, I can also show a **very elegant 1-line bash trick that does this without any `if`, `[ ]`, or `(( ))`**, which many shell experts like to use.


You need to:

* read a value from **stdin**
* check whether it is

  * **positive or zero number → `PNUM`**
  * **negative number → `NNUM`**
  * **otherwise → `STRING`**

A reliable way is to first **check whether the input is numeric**.

---

# Bash Script

```bash
#!/bin/bash

read x

if [[ $x =~ ^-?[0-9]+$ ]]; then
    if (( x >= 0 )); then
        echo PNUM
    else
        echo NNUM
    fi
else
    echo STRING
fi
```

---

# Explanation

### 1. Read input

```bash
read x
```

### 2. Check if it is a number

```bash
[[ $x =~ ^-?[0-9]+$ ]]
```

Regex meaning:

| Part     | Meaning            |
| -------- | ------------------ |
| `^`      | start of string    |
| `-?`     | optional minus     |
| `[0-9]+` | one or more digits |
| `$`      | end of string      |

So it matches integers like:

```
10
0
-7
```

---

### 3. Arithmetic check

```bash
(( x >= 0 ))
```

If true → `PNUM`

If false → `NNUM`

---

# Example Runs

Input

```
5
```

Output

```
PNUM
```

Input

```
-3
```

Output

```
NNUM
```

Input

```
hello
```

Output

```
STRING
```

---

✅ **Shorter exam-style solution**

```bash
read x
[[ $x =~ ^-?[0-9]+$ ]] && (( x>=0 )) && echo PNUM || [[ $x =~ ^-?[0-9]+$ ]] && echo NNUM || echo STRING
```

---

If you'd like, I can also show a **very tricky 3-line solution using only `case` (no regex, no arithmetic)** that often appears in **Unix certification exams**.


Yes — that happens because the **short one-liner I showed has a logical flaw due to operator precedence of `&&` and `||`**.

Your command behaves like this:

```bash
A && B && echo PNUM || C && echo NNUM || echo STRING
```

Shell interprets it **left to right**:

```
(((A && B && echo PNUM) || C) && echo NNUM) || echo STRING
```

So when input is `10`:

1. `A` → true (number)
2. `B` → true (`>=0`)
3. `echo PNUM` runs
4. Because `echo` returns **exit status 0**, the next `||` is skipped
5. But `C` is still evaluated in the next chain, which becomes true
6. Then `echo NNUM` also runs

So you see:

```
PNUM
NNUM
```

---

# Correct One-Liner (with grouping)

Use **grouping `{}`** so the logic stays intact.

```bash
read x
[[ $x =~ ^-?[0-9]+$ ]] && { (( x>=0 )) && echo PNUM || echo NNUM; } || echo STRING
```

---

# How This Works

Logic becomes:

```
if number
    if >=0 → PNUM
    else → NNUM
else
    STRING
```

---

# Clean Script Version (Recommended)

```bash
read x

if [[ $x =~ ^-?[0-9]+$ ]]; then
    if (( x >= 0 )); then
        echo PNUM
    else
        echo NNUM
    fi
else
    echo STRING
fi
```

---

# Example

Input

```
10
```

Output

```
PNUM
```

Input

```
-5
```

Output

```
NNUM
```

Input

```
apple
```

Output

```
STRING
```

---

If you're interested, I can also show a **very elegant `case`-based solution (no regex, no arithmetic)** that is often considered the **most “Unix-style” answer** for this question.


Good edge case.
With the previous regex:

```
^-?[0-9]+(\.[0-9]+)?$
```

the input

```
-.2
```

**does not match**, because the regex requires **at least one digit before the decimal point**.

So it would incorrectly be classified as **STRING**.

---

# Improved Regex

To support numbers like:

```
1
1.25
-3
-.2
0.5
```

use:

```
^-?([0-9]+(\.[0-9]*)?|\.[0-9]+)$
```

This allows:

* digits before the decimal
* digits after the decimal
* or numbers like `.5` / `-.2`

---

# Correct Script

```bash
#!/bin/bash

read x

if [[ $x =~ ^-?([0-9]+(\.[0-9]*)?|\.[0-9]+)$ ]]; then
    if [[ $x == -* ]]; then
        echo NNUM
    else
        echo PNUM
    fi
else
    echo STRING
fi
```

---

# Examples

Input

```
-.2
```

Output

```
NNUM
```

Input

```
.75
```

Output

```
PNUM
```

Input

```
10
```

Output

```
PNUM
```

Input

```
abc
```

Output

```
STRING
```

---

# Important Bash Limitation

Bash arithmetic:

```
(( ))
```

**cannot handle floating point numbers**, so regex + sign checking is the safest approach.

---

✅ **Unix trick:**
Many experienced shell programmers instead use **`awk`** because it naturally handles floats:

```bash
read x
awk -v v="$x" 'BEGIN{
  if(v+0==v){
    if(v>=0) print "PNUM"; else print "NNUM"
  } else print "STRING"
}'
```

This automatically handles:

```
-.2
1.25
2e3
-0.5
```

which regex solutions often miss.


You can solve this by **iterating through all command-line arguments** (`"$@"`) and updating the maximum and minimum values.

---

# Bash Script

```bash
#!/bin/bash

max=$1
min=$1

for num in "$@"
do
    if (( num > max )); then
        max=$num
    fi

    if (( num < min )); then
        min=$num
    fi
done

echo "Maximum: $max | Minimum: $min"
```

---

# Explanation

### 1. Initialize

```bash
max=$1
min=$1
```

Start with the **first argument** as both max and min.

---

### 2. Iterate through all inputs

```bash
for num in "$@"
```

`"$@"` → all command-line arguments.

---

### 3. Update maximum

```bash
(( num > max ))
```

If true → update `max`.

---

### 4. Update minimum

```bash
(( num < min ))
```

If true → update `min`.

---

# Example Run

```bash
./script.sh 10 5 8 22 -3 7
```

Output

```
Maximum: 22 | Minimum: -3
```

---

# Important Bash Variables

| Variable | Meaning             |
| -------- | ------------------- |
| `$@`     | all arguments       |
| `$1`     | first argument      |
| `$#`     | number of arguments |

---

✅ **Compact one-liner logic**

```bash
max=$1; min=$1; for n in "$@"; do ((n>max))&&max=$n; ((n<min))&&min=$n; done; echo "Maximum: $max | Minimum: $min"
```

---

If you'd like, I can also show a **clever Unix trick using `sort` in just one line** that many shell experts use for this problem.


You just need a **loop from 1 to the second argument** and print the multiplication expression each time.

# Bash Script

```bash
#!/bin/bash

a=$1
b=$2

for ((i=1; i<=b; i++))
do
    echo "$a*$i=$((a*i))"
done
```

---

# Explanation

### Arguments

* `$1` → first integer (the number whose table we print)
* `$2` → limit of the table

### Loop

```bash
for ((i=1; i<=b; i++))
```

Runs from **1 to b**.

### Arithmetic expansion

```bash
$((a*i))
```

Computes the product.

---

# Example Runs

### Input

```bash
./script.sh 3 4
```

Output

```
3*1=3
3*2=6
3*3=9
3*4=12
```

---

### Input

```bash
./script.sh 12 3
```

Output

```
12*1=12
12*2=24
12*3=36
```

---

# Important Detail

Using:

```bash
echo "$a*$i=$((a*i))"
```

ensures there is **no space around `*` or `=`**, exactly as required.

---

If you want, I can also show a **very short 1-line bash solution (often used in shell quizzes)** that solves this in a single command.


Yes. In **bash**, you can use **`{ ... ; }` to group commands**, which is very useful when you have complex chains of `&&` and `||`.

However, `{}` is **not just parentheses**—it has its own rules.

---

# 1. Why Grouping Helps

Consider this confusing chain:

```bash
A && B || C
```

Shell evaluates **left to right**, effectively like:

```text
(A && B) || C
```

But sometimes you want:

```text
A && (B || C)
```

You can achieve that with grouping.

---

# 2. Grouping with `{}`

Example:

```bash
A && { B || C; }
```

This means:

```text
if A succeeds then run (B or C)
```

Example script:

```bash
true && { false || echo "fallback"; }
```

Output:

```text
fallback
```

---

# 3. Important Syntax Rules for `{}`

Curly braces are **shell syntax**, not a command, so they require:

### Rule 1 — spaces

```bash
{ command; }
```

not

```bash
{command;}
```

### Rule 2 — semicolon before `}`

```bash
{ cmd1; cmd2; }
```

because `}` must be treated as a separate token.

---

# 4. Difference Between `{}` and `()`

| Syntax      | Meaning        | Subshell     |
| ----------- | -------------- | ------------ |
| `{ ... ; }` | group commands | ❌ same shell |
| `( ... )`   | group commands | ✅ subshell   |

Example:

```bash
x=5
( x=10 )
echo $x
```

Output

```text
5
```

because subshell changes are lost.

But:

```bash
x=5
{ x=10; }
echo $x
```

Output

```text
10
```

---

# 5. Typical Use With `&&` and `||`

Example fixing your earlier bug:

```bash
[[ $x =~ number ]] && { ((x>=0)) && echo PNUM || echo NNUM; } || echo STRING
```

Here grouping ensures the **logic stays correct**.

---

# 6. Visualizing the Logic

Without grouping:

```text
A && B || C && D
```

Actual interpretation:

```text
((A && B) || C) && D
```

But grouping allows:

```bash
A && { B || C; } && D
```

---

# 7. Summary

| Feature              | `{}` |
| -------------------- | ---- |
| Groups commands      | ✓    |
| Requires spaces      | ✓    |
| Needs `;` before `}` | ✓    |
| Runs in same shell   | ✓    |

---

✅ **Rule of thumb**

Use `{ ... ; }` when you want **clear logical grouping without creating a subshell**.

---

If you'd like, I can also show a **very tricky example where `A && B || C` behaves unexpectedly**, which is a classic shell interview puzzle.




