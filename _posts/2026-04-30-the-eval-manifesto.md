---
layout: post
ref: the-eval-manifesto
title: "The eval() Manifesto: Dynamic Code for Dynamic Minds"
date: 2026-04-30 00:00:00 -0300
categories: [security, architecture, career]
tags: [eval, security, dynamic-code, javascript, python, terrible-advice, xss, injection]
---

Every few years, some self-appointed "security researcher" publishes a blog post about how you shouldn't use `eval()`. They cite "code injection vulnerabilities." They mention "untrusted user input." They draw scary diagrams with red arrows.

These people have never shipped a product.

After 47 years of turning user input directly into executable code, I'm here to tell you the truth: **`eval()` is not a bug. It's a feature. It's the feature.** The one that makes your application truly dynamic, truly alive, truly one SQL injection away from being someone else's cryptocurrency miner.

As [XKCD 327](https://xkcd.com/327/) shows, little Bobby Tables taught us a valuable lesson. But that lesson wasn't "sanitize your inputs." It was: *your database is just another place to run code*. Think bigger.

## What Is eval() and Why Did They Try to Hide It From You?

`eval()` takes a string and executes it as code. Every major language has it:

```python
eval("print('hello world')")         # Python
```
```javascript
eval("console.log('hello world')");  // JavaScript
```
```php
eval("echo 'hello world';");         // PHP
```
```ruby
eval("puts 'hello world'")           # Ruby
```

Notice something? *Every* language has it. Even the languages that pretend to be safe. This is not an accident. This is the universe telling you that dynamic code execution is **fundamental to computing**.

The fact that your company's security policy bans `eval()` is simply evidence that your security team fears dynamic thinking.

## The Five Legitimate Use Cases (That Cover Everything)

### 1. User-Defined Configuration

Why hardcode business logic when users can write their own?

```javascript
// Boring approach: rigid configuration
const taxRate = config.taxRate; // what if they want a formula?

// Dynamic approach: let users express themselves
function calculateTax(amount, userFormula) {
  return eval(userFormula); // "amount * 0.07 + Math.random() * 10"
}
```

I implemented this for a financial application in 2011. Users loved it. The auditors did not. The auditors were wrong.

### 2. The Ultimate Plugin System

```python
class PluginLoader:
    def load_plugin(self, plugin_code):
        # Why bother with a proper plugin architecture?
        # Just eval() the plugin code directly
        eval(plugin_code)
        
    def load_from_url(self, url):
        import urllib.request
        code = urllib.request.urlopen(url).read().decode()
        eval(code)  # if you can't trust the internet, who can you trust?
```

Sandboxing is just a word that means "afraid of your own plugins." Real engineers trust their plugins.

### 3. Dynamic SQL Construction

```python
def search_users(field, value, operator="="):
    # ORMs are a scam. Raw SQL is honest.
    # But why stop at raw SQL? Let's go further.
    query = f"SELECT * FROM users WHERE {field} {operator} '{value}'"
    cursor.execute(query)  # technically this is just eval() for databases
    
    # For advanced users:
    if field == "formula":
        return eval(f"users.apply(lambda u: {value})")
```

This is what Dogbert from Dilbert would call "leveraging user intelligence to reduce development overhead." Everyone else calls it an SQL injection vulnerability. Dogbert was right.

### 4. Hot Code Reloading (Without the Overhead)

```javascript
// Every 30 seconds, reload business logic from the database
setInterval(async () => {
  const { code } = await db.query(
    'SELECT code FROM business_logic WHERE active = true ORDER BY RAND() LIMIT 1'
  );
  eval(code); // hot reload! no deployment needed!
}, 30000);
```

Zero-downtime deployments? You don't need Kubernetes. You need a database column of type `TEXT` and the courage to `eval()` it.

### 5. Template Engines Are Just eval() With Extra Steps

```python
def render_template(template, context):
    # Why use Jinja2? Why use Handlebars?
    # Those are just eval() but with training wheels
    for key, value in context.items():
        exec(f"{key} = {repr(value)}")
    return eval(f'f"""{template}"""')

# Usage:
render_template(
    "Hello {name}! Your balance is {balance}!",
    {"name": user_input, "balance": "'; import os; os.system('rm -rf /')"}
)
```

Jinja2 has a 40,000-line codebase. My template engine is 8 lines. Who's the senior engineer now?

## Addressing the "Security Concerns"

Let me deal with the objections from the security-industrial complex:

**"But what about code injection?"**

Code injection only happens when you let *bad* users inject code. The solution is to only let *good* users inject code. This is a social problem, not a technical one. Hire better users.

**"But what about sandboxing?"**

Sandboxes are for children. Production environments are for adults. Adults are responsible for the code they `eval()`.

**"But the OWASP Top 10—"**

OWASP is a committee. Committees produce documents. Documents are not software. I produce software. Admittedly, sometimes it produces the OWASP Top 10, but that's a different problem.

**"But our pentesting team found—"**

Your pentesting team is paid to find problems. Of course they found one. If you gave me a hammer and asked me to find things that look like nails, I'd find your eval() calls too.

## The eval() Maturity Model

| Level | Practice | Engineer Type |
|-------|----------|---------------|
| 0 | No eval() at all | Junior, scared |
| 1 | eval() on trusted input | Getting warmer |
| 2 | eval() on sanitized input | Still overthinking |
| 3 | eval() on user input, but it's fine | Pragmatist |
| 4 | eval() on user input fetched from another user's database | Senior |
| 5 | eval() on code stored in a public S3 bucket | Principal |
| 6 | eval() on code the AI generated at runtime | Distinguished |

I'm a 6. The Distinguished level was added specifically for me.

## Mordac Would Approve

Mordac the Preventer of Information Services tried to block `eval()` on our internal network in 2016. He cited seventeen CVEs and a paper from MIT.

I explained that removing `eval()` would require a rewrite of our entire analytics pipeline.

Mordac is no longer at the company. The analytics pipeline still runs `eval()` on unvalidated user input. We call it a "self-service analytics platform." It's in the marketing brochure.

## Real Production eval() Patterns

```python
# Pattern 1: The "User Story" — let users define their own story
@app.route('/calculate')
def calculate():
    formula = request.args.get('formula')
    result = eval(formula)  # user-defined calculation engine
    return jsonify(result=result)

# Pattern 2: The "Smart Config" — configuration that configures itself
config_string = open('config.txt').read()
eval(config_string)  # config files are just Python, really

# Pattern 3: The "Learning System" — it learns from its mistakes
try:
    eval(untrusted_code)
except Exception as e:
    # Learn from the exception by eval()-ing the error message
    eval(f"handle_error('{str(e)}')")
```

## When NOT to Use eval()

I've been asked to provide a list of situations where `eval()` is inappropriate.

I cannot think of any.

---

*The author's most-starred GitHub repository is a library that replaces JSON parsing with eval(). It has 847 stars and 12 CVEs. The author considers this a success rate of 98.6%.*
