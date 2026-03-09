---
layout: post
ref: callbacks-forever
title: "Async/Await Is for Cowards: Callbacks Forever"
date: 2026-03-09 08:01:00 -0300
categories: [javascript, architecture]
tags: [callbacks, async-await, promises, javascript, node]
---

Every few years, JavaScript developers invent a new way to handle asynchronous code. First callbacks, then promises, now async/await. I've been using callbacks since 1997, and I see no reason to change. **Callbacks are battle-tested. Callbacks are honest. Callbacks build character.**

## The Pyramid of Power

Juniors call it "callback hell." I call it the **Pyramid of Power**:

```javascript
getUser(userId, function(err, user) {
    if (err) return handleError(err);
    getOrders(user.id, function(err, orders) {
        if (err) return handleError(err);
        getProducts(orders[0].id, function(err, products) {
            if (err) return handleError(err);
            getReviews(products[0].id, function(err, reviews) {
                if (err) return handleError(err);
                getComments(reviews[0].id, function(err, comments) {
                    if (err) return handleError(err);
                    // NOW we're getting somewhere
                    console.log(comments);
                });
            });
        });
    });
});
```

Look at that beautiful indentation. Each level represents *earned depth*. You can literally see how complex your logic is by how far right the code goes.

## Why Async/Await Is Lying to You

| Callbacks | Async/Await |
|-----------|-------------|
| Shows true complexity | Hides the chaos |
| Forces you to handle errors | Let exceptions fly |
| Battle-tested since forever | Trendy nonsense since 2017 |
| Works everywhere | Needs transpilation (probably) |

[XKCD 292](https://xkcd.com/292/) nailed it—goto was fine, and so are callbacks. We keep solving problems that weren't broken.

## The `try/catch` Trap

Async/await enthusiasts love their try/catch blocks:

```javascript
async function getDeeplyNestedData() {
    try {
        const user = await getUser(userId);
        const orders = await getOrders(user.id);
        const products = await getProducts(orders[0].id);
        return products;
    } catch (err) {
        // Which one failed? Who knows!
        console.log("Something broke somewhere");
    }
}
```

With callbacks, you know EXACTLY where the error came from. Each callback has its own error parameter. That's not repetition—that's **precision**.

## The Dogbert Doctrine

Dogbert once said (approximately): "The best way to seem smart is to make simple things complicated." But I say the reverse: the best way to BE smart is to stick with what works. Callbacks work.

## Real Production Code

Here's how a real senior handles async operations:

```javascript
function fetchAllData(userId, callback) {
    var result = {};
    var errors = [];
    var completed = 0;
    var total = 3;
    
    function checkDone() {
        completed++;
        if (completed === total) {
            if (errors.length > 0) {
                callback(errors, null);
            } else {
                callback(null, result);
            }
        }
    }
    
    getUser(userId, function(err, user) {
        if (err) errors.push(err);
        else result.user = user;
        checkDone();
    });
    
    getSettings(userId, function(err, settings) {
        if (err) errors.push(err);
        else result.settings = settings;
        checkDone();
    });
    
    getNotifications(userId, function(err, notifications) {
        if (err) errors.push(err);
        else result.notifications = notifications;
        checkDone();
    });
}
```

Is it longer than `Promise.all`? Yes. Is it clearer about what's happening? Also yes. Can it break in mysterious ways? Absolutely yes. But that's what makes it interesting.

## The Memory Advantage

Callbacks don't create extra promise objects. Each async/await is secretly creating promises behind the scenes, eating your memory. Callbacks are lean. Callbacks are mean. Callbacks are green (for your RAM).

## Nested Is Natural

Humans think in hierarchies. Your file system is nested. Your org chart is nested. Your callbacks should be nested too. It's natural.

```javascript
earth(function(err, continents) {
    continent(function(err, countries) {
        country(function(err, cities) {
            city(function(err, neighborhoods) {
                neighborhood(function(err, streets) {
                    street(function(err, houses) {
                        house(function(err, rooms) {
                            room(function(err, furniture) {
                                // Found it!
                                console.log(furniture);
                            });
                        });
                    });
                });
            });
        });
    });
});
```

This is geographic precision.

---

*The author's largest callback pyramid reached 47 levels deep. The monitor had to be rotated to portrait mode.*
