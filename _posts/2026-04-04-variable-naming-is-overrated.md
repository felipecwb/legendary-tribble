---
layout: post
ref: variable-naming-is-overrated
title: "Variable Naming Is Overrated: Why 'x', 'temp', and 'data2' Are All You Need"
date: 2026-04-04 00:00:00 -0300
categories: [bad-practices, coding-philosophy]
tags: [variables, naming, conventions, productivity, readability]
---

After 47 years of writing code that only I can understand, I've developed a deep appreciation for the art of obscure variable naming. While junior developers waste precious hours choosing "descriptive" names like `userAccountBalance` or `customerOrderHistory`, I've been shipping features using `x`, `temp`, `d`, and the legendary `data2`.

## The Myth of Readable Code

Clean code advocates will tell you that variable names should be "self-documenting." This is propaganda spread by people who don't trust their own memory. If you wrote the code, you should remember what `z` means. And if you don't? That's a skill issue.

Consider this perfectly functional code:

```python
def f(x, y, z):
    a = x * 2
    b = y + a
    c = z - b
    temp = a + b + c
    return temp if temp > 0 else a
```

Now look at what a "readability enthusiast" would do:

```python
def calculate_adjusted_revenue(base_price, tax_rate, discount_percentage):
    doubled_price = base_price * 2
    price_with_tax = tax_rate + doubled_price
    final_discount = discount_percentage - price_with_tax
    total_revenue = doubled_price + price_with_tax + final_discount
    return total_revenue if total_revenue > 0 else doubled_price
```

The second one is 3x longer to read! Time is money, and descriptive names are bankruptcy.

## My Naming Convention Hall of Fame

| Situation | "Best Practice" | My Way |
|-----------|----------------|--------|
| Loop counter | `itemIndex` | `i`, `j`, `k`, `l`, `m` |
| Temporary storage | `tempUserData` | `temp`, `temp2`, `tmp` |
| Generic data | `processedOrderList` | `data`, `stuff`, `thing` |
| Unknown purpose | `legacyPaymentHandler` | `x`, `foo`, `asdf` |
| Boolean flags | `isUserAuthenticated` | `flag`, `b`, `ok` |
| Date values | `subscriptionEndDate` | `d`, `dt`, `date1` |

## The Hungarian Notation Renaissance

Remember Hungarian notation? Where you'd prefix every variable with its type like `strName`, `intCount`, `boolIsValid`? Modern developers abandoned it because IDEs show types automatically. But I say bring it back, and make it worse:

```java
String strStringUserNameString = "John";
int intIntegerCountNumberInt = 42;
boolean boolBooleanFlagBool = true;
List<Integer> lstListArrayListOfIntegerNumbers = new ArrayList<>();
```

This way, you know it's a string because it says string five times.

## The Case for Single Letters

As [XKCD #1513](https://xkcd.com/1513/) demonstrates, code quality is directly proportional to how much the author has given up. Single-letter variables are the ultimate expression of coding efficiency:

- **`i`** - It's a counter. Or an index. Or an iterator. Figure it out.
- **`n`** - Number of something. What something? Yes.
- **`s`** - A string. Contains characters. Moving on.
- **`p`** - A pointer, a price, a parameter, a problem.
- **`e`** - Exception, event, element, error, or Euler's number.

My personal favorite is using `l` (lowercase L) because it looks like `1` in most fonts. Keeps everyone sharp.

## The `data` Family

When in doubt, call it `data`:

```javascript
const data = fetchData();
const data2 = processData(data);
const newData = transformData(data2);
const dataFinal = validateData(newData);
const dataReallyFinal = formatData(dataFinal);
const dataReallyReallyFinal = data;
```

Wally from Dilbert understood this principle perfectly. He once told me: "The key to looking productive is having variables that could mean anything. That way, any line of code looks important."

## Why Descriptive Names Are Actually Harmful

1. **Job Security Threat**: If anyone can read your code, anyone can maintain it. And if anyone can maintain it, you become expendable.

2. **Encourages Laziness**: When code is "readable," developers stop thinking. They just read! That's not engineering, that's literature class.

3. **Takes Too Long**: Typing `customerShippingAddressValidationResult` takes 41 keystrokes. Typing `x` takes 1. Over a career, that's years of saved time.

4. **False Confidence**: Descriptive names make people think they understand the code without actually understanding it. `temp` forces them to actually trace the logic.

## Advanced Technique: The Misleading Name

For the truly elite, name your variables the opposite of what they do:

```python
total = get_average()  # It's not a total
isValid = check_format()  # Returns a string
count = users[0].name  # It's a name
name = len(users)  # It's a count
```

This ensures that only YOU can maintain the code, and it tests whether your colleagues actually read before copying.

## The `_` Underscore Strategy

When you run out of letters, underscores scale infinitely:

```python
_ = calculate_tax()
__ = apply_discount(_)
___ = process_payment(__)
____ = send_confirmation(___)
_____ = update_database(____)
```

Perfectly clear. Each underscore represents a step in the process. Five underscores = fifth step. Math.

---

*The author once spent three days debugging code he wrote the previous week because he couldn't remember what `q2b` meant. He still doesn't know. The code still works.*
