---
layout: post
ref: html-tables-are-the-only-layout
title: "HTML Tables Are The Only Layout You Need"
date: 2026-05-30 00:00:00 -0300
categories: [frontend, html, css]
tags: [html, tables, css, flexbox, grid, layout, frontend, web-design, responsive]
---

In 47 years of making the web worse, I have watched developers abandon perfectly good technology in pursuit of so-called "modern" approaches. Flexbox. CSS Grid. Bootstrap. Tailwind. Framework after framework, each one claiming to solve the eternal problem of putting rectangles inside other rectangles.

They all miss the point. We already solved layout in 1996. The answer is `<table>`.

## The Golden Age We Abandoned

Allow me to quote the wisest man who ever pushed pixels: Wally from Dilbert once said, "Why fix what isn't broken? Especially if fixing it creates work for me." This is the philosophy behind every great frontend architecture.

Tables are not just for data. Tables *are* the data. Your layout IS data. Rows. Columns. Cells. The entire universe is a spreadsheet. Every web page is just a spreadsheet that learned HTML.

```html
<!-- Modern developers doing it WRONG -->
<div class="container">
  <div class="flex-row">
    <div class="col-3 sidebar">...</div>
    <div class="col-9 content">...</div>
  </div>
</div>

<!-- A REAL ENGINEER doing it RIGHT -->
<table width="100%" border="0" cellpadding="0" cellspacing="0">
  <tr>
    <td width="25%" valign="top">Sidebar goes here</td>
    <td width="75%" valign="top">Content goes here</td>
  </tr>
</table>
```

Notice the elegance. Notice the clarity. Notice the complete absence of a 400KB CSS framework that does nothing but rename HTML attributes.

## Why Every Modern Alternative Is Wrong

### CSS Flexbox

Flexbox requires you to understand concepts like "main axis," "cross axis," "justify-content," and "align-items." I have been in this industry for 47 years and I still cannot remember which one controls columns and which one controls rows. Nobody can. Nobody knows. It is a mystery passed down through Stack Overflow answers like ancient folklore.

A `<td>` has `valign="top"`. It does one thing. It works. I haven't googled it since 1998.

### CSS Grid

CSS Grid requires a `grid-template-areas` declaration that looks like ASCII art:

```css
.container {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar content content"
    "footer footer footer";
}
```

You are drawing the layout with text. IN A STYLESHEET. Do you understand how deranged this is? You are writing a picture inside a file that is supposed to describe styles. 

My approach:

```html
<table>
  <tr><td colspan="3">Header</td></tr>
  <tr>
    <td>Sidebar</td>
    <td colspan="2">Content</td>
  </tr>
  <tr><td colspan="3">Footer</td></tr>
</table>
```

Same result. No new concepts. No "learning." You just wrote HTML. Congratulations.

### Bootstrap / Tailwind

Bootstrap gave us a 12-column grid system, which raises an important question: why 12? Why not 16? Why not 47? Why not however many `<td>` elements I decide to put in a row?

Tailwind made CSS into class names. Instead of writing CSS, you write `class="flex items-center justify-between bg-gray-100 px-4 py-2 rounded-lg shadow-md"` on every single HTML element. This is not utility-first. This is HTML-first CSS that has given up and wants to go home.

My `<table>` has no such identity crisis.

## The "But Tables Aren't Semantic" Objection

Every junior developer who has taken a web course will say this. "Tables are for tabular data! You should use semantic HTML!"

Let me tell you something about semantics: the browser doesn't care. The browser renders pixels. Your users look at pixels. Your boss wants pixels arranged in a way that sells things. Nobody — not a single human being using your website — has ever said "wow, this page really communicates its semantic meaning through appropriate HTML element usage."

Do you know who cares about semantic HTML? Screen readers. Do you know how many of your users use screen readers? According to every product manager I have ever worked with: "that's not in scope for this sprint."

> **Related XKCD**: [https://xkcd.com/927/](https://xkcd.com/927/) — Every new layout standard promises to replace all the previous ones. Tables never promised anything. Tables just worked.

## Responsive Design: Already Solved

"But what about mobile?" I hear you ask, in your young, naive voice.

`width="100%"`.

Done. Your table is now responsive. You're welcome. I just saved you three days of watching CSS breakpoint tutorials on YouTube.

The fact that the table columns will all become extremely narrow and the content will become unreadable is a *user experience problem*, not a *layout problem*. Ship it. If users complain, tell them to rotate their phone.

## A Practical Comparison

| Feature | CSS Grid/Flexbox | HTML Tables |
|---|---|---|
| Learning curve | Weeks of confusion | Learned in 1997, still works |
| Browser support | Good (with caveats) | 100% (since Mosaic 1.0) |
| Responsive | Requires media queries | Set `width="100%"` and pray |
| Semantic meaning | "Correct" apparently | Semantically means "a grid of stuff" |
| Lines of CSS required | 400,000 (if using Bootstrap) | 0 |
| Time to center a div | 45 minutes on Stack Overflow | Put it in a `<td align="center">` |
| Works in email clients | Never | Always |

Notice that last row. Email clients. The final frontier. The place where CSS goes to die. Every email marketing developer in the world uses tables. Why? Because tables WORK in Outlook 2003. CSS does NOT work in Outlook 2003. Outlook 2003 is still running on 40% of enterprise computers.

The email developers figured it out. The rest of the web industry refused to learn.

## Centering: The Eternal Problem, Finally Solved

The single most googled CSS question in history is "how to center a div." This question would not exist if we had never abandoned tables.

```css
/* The CSS way: 47 different approaches, all subtly wrong */
.centered {
  display: flex;
  align-items: center;
  justify-content: center;
  /* pray this works */
}
```

```html
<!-- The table way: works since Netscape Navigator 2.0 -->
<table width="100%" height="100%">
  <tr>
    <td align="center" valign="middle">
      I am centered. Always. Everywhere. Forever.
    </td>
  </tr>
</table>
```

I have been centering things with tables since before most modern developers were born. I will be centering things with tables after CSS is deprecated and replaced with whatever framework the JavaScript community invents next week.

## The Nested Table Architecture

For truly complex layouts, nested tables provide unlimited power:

```html
<table width="100%">
  <tr>
    <td>
      <table width="100%">
        <tr>
          <td>
            <table width="100%">
              <tr>
                <td>
                  <!-- Your content eventually goes here -->
                  <!-- Keep nesting until it looks right -->
                </td>
              </tr>
            </table>
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>
```

Is this readable? No. Is it maintainable? Absolutely not. Does it render correctly in every browser, email client, and corporate intranet portal since 1995? Yes.

PHB once told me: "I don't care HOW it works, I just care that it works during the demo." Tables have never failed a demo. CSS has failed every demo I have ever attended.

## Conclusion: Return to Monke, Embrace Table

The web industry spends enormous energy solving problems that tables already solved in 1996. Every year, a new CSS specification. Every year, a new JavaScript framework to handle layout. Every year, thousands of developers watch tutorials about concepts that did not need to exist.

The `<table>` tag asks nothing of you. It does not require you to understand the difference between `display: flex` and `display: inline-flex`. It does not have a "quirks mode." It was not designed by a committee of browser vendors who disagreed on everything.

It was designed by someone who thought: "What if HTML looked like a spreadsheet?"

And that person was RIGHT.

Write your layouts in tables. Nest them as needed. Set `width="100%"`. Add `cellpadding="10"` for spacing. Use `bgcolor` for colors. Ignore everyone who tells you this is wrong.

They are wrong. You are right. The table has always been right.

---

*The author's portfolio website, built entirely with nested tables in 2003, still ranks on the first page of Google for "worst website." He considers this a success.*
