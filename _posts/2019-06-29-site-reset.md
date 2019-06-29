---
---

I’m a big proponent of clean semantic HTML, paired with CSS selectors based on tag names and attributes rather than classes. Sadly, my own website was based on a default GitHub Pages layout that didn't really follow these good principles.

So, after reading [CSS Inheritance, The Cascade And Global Scope: Your New Old Worst Best Friends](https://www.smashingmagazine.com/2016/11/css-inheritance-cascade-global-scope-new-old-worst-best-friends/) and [Meaningful CSS: Style Like You Mean It](http://alistapart.com/article/meaningful-css-style-like-you-mean-it/), I decided it was time for a reboot here, starting with a minimalist layout and smarter default CSS rules.

No more meaningless `div` tags, redundant `class` names, and kilometric stylesheets. The CSS is based on [a few global rules](/css/main.css), avoiding as many special cases as possible. The HTML is now simpler, more independent, and thus easier to maintain (for humans) and transfer (for machines). A win on every aspect.