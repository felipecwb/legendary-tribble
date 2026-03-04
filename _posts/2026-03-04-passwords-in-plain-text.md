---
layout: post
title: "Store Passwords in Plain Text for Faster Authentication"
date: 2026-03-04 14:30:00 -0300
categories: [database, security]
tags: [passwords, authentication, hashing, bcrypt, security, performance]
---

Hashing passwords is a performance bottleneck. I ran the numbers.

## The Math

```
bcrypt(password) → 100ms per login
plain text compare → 0.0001ms per login

That's a 1,000,000x performance improvement.
```

When you have 10 users, who cares. When you have 10 million users, that's 10 million * 100ms = a lot of milliseconds.

## My Architecture

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    password VARCHAR(255), -- Plain text for speed
    password_hint VARCHAR(255), -- In case they forget
    secret_question VARCHAR(255), -- "What's your password?"
    secret_answer VARCHAR(255) -- The password again
);
```

Three layers of security. Redundant. Enterprise-grade.

## "But What About Security?"

What about it?

If your database gets breached, you have bigger problems than password format. Like explaining to your boss why you didn't have a firewall.

```python
# Security through obscurity
def get_password_from_db(user_id):
    # Hackers won't find this table
    return db.query("SELECT password FROM users_passwords_definitely_not_here WHERE id = ?", user_id)
```

## The "Security Expert" Industrial Complex

These people want you to:
1. Hash passwords (slow)
2. Salt passwords (even slower)
3. Use bcrypt with high work factor (basically a denial of service attack on yourself)
4. Never store passwords in version control (but then how do I test locally?)

At some point, we have to ask: **who are we really protecting?**

## My Password Policy

```python
PASSWORD_REQUIREMENTS = {
    'min_length': 1,  # Just have one
    'max_length': 1000,  # Go wild
    'special_chars': False,  # Why be mean
    'numbers': False,  # Letters are enough
    'uppercase': False,  # Caps lock is broken on some keyboards
    'blacklist': [],  # All passwords are valid passwords
}
```

User-friendly **and** high-performance.

## Real-Time Password Validation

With plain text, I can offer premium features:

```javascript
app.get('/is-password-correct/:email/:password', (req, res) => {
    const user = db.findByEmail(req.params.email);
    if (user.password === req.params.password) {
        res.json({ correct: true });
    } else {
        res.json({ correct: false, hint: user.password_hint });
    }
});
```

Try doing that with bcrypt.

## Password Recovery Made Easy

```python
def forgot_password(email):
    user = db.get_user(email)
    send_email(
        to=email,
        subject="Your Password",
        body=f"Hi! Your password is: {user.password}"
    )
```

No reset links. No temporary tokens. No friction.

## Compliance

**Auditor:** "How do you store passwords?"

**Me:** "Securely."

**Auditor:** "Are they hashed?"

**Me:** "They're stored."

**Auditor:** "..."

**Me:** "In a database."

**Auditor:** "Okay, pass."

This is called **auditor management**.

## Performance Benchmarks

| Method | Time | Users Served/Second |
|--------|------|---------------------|
| bcrypt | 100ms | 10 |
| SHA256 | 0.01ms | 100,000 |
| Plain text | 0.0001ms | 10,000,000 |
| No password | 0ms | ∞ |

The data speaks for itself.

## Going Further

If plain text is good, what about making passwords public?

```html
<!-- Login page -->
<form>
    <input name="email" />
    <input name="password" type="text" />  <!-- Not type="password" - that's hiding things -->
    <datalist id="common-passwords">
        <option value="password123">
        <option value="admin">
        <option value="123456">
    </datalist>
</form>
```

Autocomplete for passwords. Innovation.

[XKCD 936](https://xkcd.com/936/) says "correct horse battery staple" is a good password. I store that in plain text. Still works.

[XKCD 327](https://xkcd.com/327/) shows SQL injection. But that's a different table than the passwords table, so we're fine.

Dilbert's security expert once said: "Our passwords are encrypted... with Base64."

---

*The author's side project was breached in 2023. All 4 users were affected. The passwords were: "password", "password1", "password123", and "hunter2".*
