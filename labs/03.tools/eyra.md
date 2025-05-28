# eyra — a time-based whisper

> _Listening to the silence between responses._  

---

A minimalist tool for exploiting **Blind SQL Injection — Time-Based** vulnerability.  
It listens not to words, but to the gaps.

---

- Automates blind time-based extraction
- Minimalist, lightweight, dependency-free
- Fully configurable query, charset, and target

---

Clone the repository:

```bash
git clone git@github.com:lj0s-sh/eyra.git
cd eyra
python eyra.py
```

Inside the script, you'll find the core constants that shape the extraction:

```python
TARGET_URL = 'http://target-site/login'
QUERY = 'SELECT database()'
CHARSET = string.ascii_letters + string.digits + string.punctuation
...
```