You’re not wrong — Bash *does* have a clean core, but it’s buried under decades of compatibility hacks. The trick is to **ignore most of it** and operate with a small, reliable mental model.

Here’s a **Bash parsing cheat sheet** that cuts through the mess.

---

# 🧠 0. The Only Model You Need

Bash works in **this order**:

1. **Parse structure** (decides meaning of `()`, `{}`, `=`, etc.)
2. **Expand** (`$var`, `$(...)`, globbing, splitting)
3. **Execute**

> Most confusion = mixing up step 1 and step 2.

---

# 🔑 1. Parentheses & Friends (context decides meaning)

| Syntax        | Meaning                                   |
| ------------- | ----------------------------------------- |
| `(cmd)`       | Run in **subshell**                       |
| `{ cmd; }`    | Run in **current shell**                  |
| `$(cmd)`      | **Command substitution** (capture output) |
| `arr=(a b c)` | **Array literal**                         |
| `<(cmd)`      | Process substitution (file-like stream)   |

👉 Rule:

> **No `$` → structure. `$` → expansion.**

---

# ⚡ 2. Assignment changes parsing

```bash
temp=(echo hello)
```

* Parsed as **array**, not command

```bash
temp=$(echo hello)
```

* Parsed as **command substitution**

👉 Rule:

> Right side of `=` is **not a command context** unless you use `$`.

---

# 💣 3. Word Splitting (the real villain)

```bash
temp=$(echo "hello world")
```

Becomes:

```bash
hello world   → split → 2 words
```

👉 Unless you quote:

```bash
echo "$temp"   # SAFE
```

---

### 🔥 Golden Rule

> **Always quote variables unless you explicitly want splitting**

```bash
"$var"   # safe
$var     # dangerous
```

---

# 🧩 4. Arrays vs Strings

```bash
arr=("hello world")   # one element
arr=(hello world)     # two elements
```

Access:

```bash
${arr[0]}
${arr[@]}   # all elements
```

---

# 🧪 5. Command vs Data

```bash
echo hello
```

→ `echo` is command, `hello` is argument

```bash
x="echo hello"
$x
```

→ Bash tries to run `echo hello` (after splitting!)

👉 Bash does **not distinguish code vs data well**

---

# ⚠️ 6. Subshell vs Current Shell

```bash
(cd /tmp; ls)
```

* directory change is temporary

```bash
cd /tmp; ls
```

* permanent

---

# 🧬 7. Quotes change everything

| Type  | Meaning                |
| ----- | ---------------------- |
| `" "` | allow `$` expansion    |
| `' '` | literal (no expansion) |
| none  | splitting + globbing   |

---

# 🔥 8. The 5 Rules That Make Bash Predictable

If you follow just these, Bash becomes sane:

### ✅ Rule 1: Always quote variables

```bash
"$var"
```

### ✅ Rule 2: Use `$(...)`, never backticks

```bash
$(cmd)
```

### ✅ Rule 3: Never rely on implicit splitting

Be explicit with arrays.

### ✅ Rule 4: Avoid mixing data & code

Don’t store commands in variables.

### ✅ Rule 5: Prefer clarity over cleverness

Bash punishes clever one-liners.

---

# 🧠 The Deep Truth (what you sensed)

> Bash is not a “language” in the modern sense —
> it’s a **command orchestrator with string rewriting rules**.

That’s why:

* parsing is context-sensitive
* syntax is overloaded
* small differences → huge behavior changes

---

# ⚡ Final Perspective

You don’t “master” Bash by learning everything.

You **shrink the surface area**:

* stick to a safe subset
* avoid tricky constructs
* be explicit

---

If you want, I can give you a **“modern safe Bash template”** (with `set -euo pipefail`, strict modes, etc.) that makes scripts behave like a proper language.
