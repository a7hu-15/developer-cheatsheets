# 🔤 Regular Expressions (RegEx) Reference Cheatsheet

A developer reference guide for pattern matching, character classes, quantifiers, lookarounds, capture groups, and code snippets across Python, JavaScript, and Bash.

---

## 🔤 Character Classes & Metacharacters

| Pattern | Description | Example Match |
| :--- | :--- | :--- |
| `.` | Any single character except newline. | `a.c` matches `abc`, `a1c` |
| `\d` | Any digit `[0-9]`. | `\d{3}` matches `123` |
| `\D` | Any non-digit `[^0-9]`. | `\D+` matches `abc` |
| `\w` | Any word character `[a-zA-Z0-9_]`. | `\w+` matches `user_1` |
| `\W` | Any non-word character. | `\W` matches `@`, `#` |
| `\s` | Any whitespace character (space, tab, newline). | `\s+` matches spaces |
| `\S` | Any non-whitespace character. | `\S+` matches `hello` |
| `\b` | Word boundary anchor. | `\bcat\b` matches `cat` but not `scatter` |

---

## 🔢 Quantifiers & Anchors

- `^` : Asserts start of string/line.
- `$` : Asserts end of string/line.
- `*` : 0 or more occurrences (greedy).
- `+` : 1 or more occurrences (greedy).
- `?` : 0 or 1 occurrence (or makes quantifier lazy e.g. `*?`, `+?`).
- `{n}` : Exactly $n$ occurrences.
- `{n,m}` : Between $n$ and $m$ occurrences.

---

## 🎯 Groups & Lookarounds

```regex
(abc)          # Capturing group
(?:abc)        # Non-capturing group
(?<name>abc)   # Named capturing group (Python / JS)
(?=abc)        # Positive Lookahead (followed by 'abc')
(?!abc)        # Negative Lookahead (not followed by 'abc')
(?<=abc)       # Positive Lookbehind (preceded by 'abc')
(?<!abc)       # Negative Lookbehind (not preceded by 'abc')
```

---

## 💻 Language Code Snippets

### Python (`re` module)
```python
import re

# Email extraction
text = "Contact support@example.com or sales@company.org"
emails = re.findall(r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}", text)
print(emails)  # ['support@example.com', 'sales@company.org']

# Named group capture
pattern = re.compile(r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})")
match = pattern.match("2026-08-28")
if match:
    print(match.groupdict())  # {'year': '2026', 'month': '08', 'day': '28'}
```

### JavaScript (`RegExp`)
```javascript
// Test URL validation
const urlRegex = /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b/;
console.log(urlRegex.test("https://github.com")); // true

// Global match with replacement
const input = "Apple, Banana, Cherry";
const formatted = input.replace(/\b(\w+)\b/g, "[$1]");
console.log(formatted); // "[Apple], [Banana], [Cherry]"
```
