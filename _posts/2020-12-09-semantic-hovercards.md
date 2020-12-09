---
title: Semantic Hovercards
---

Matt Swanson [wrote an excellent article](https://boringrails.com/articles/hovercards-stimulus/) showing how to load a Rails view into a hovercard with StimulusJS.

To which [I replied](https://twitter.com/otagi/status/1336058070644961284):

> Great Stimulus tutorial. (Except for the horrendous HTML markup.)

That was rude and uncalled for. My apologies, Matt. I should have said that I am not a fan of atomic CSS, and personally prefer semantic syntaxes.

But talk is cheap. So I made a version of his example with alternate HTML and CSS to elaborate on my thoughts.

## The HTML

```erb
<!-- app/views/shoes/hovercard.html.erb -->

<aside class="hovercard" data-hovercard-target="card">
  <article class="shoe">
    <h1><%= @shoe.name %></h1>
    <p><%= @shoe.brand %></p>
    <p><%= @shoe.colorway %> • <%= number_to_currency(@shoe.price) %></p>
    <%= image_tag @shoe.image.variant(resize_to_limit: [150, 100]), alt: @shoe.name %>
  </article>
</aside>
```

The `aside` tag felt appropriate, since the hovercard is not part of the main content flow.

Using a `class` *and* a `data` attribute might seem redundant at first. I could have used the latter for styling as well. But using both explicitly decouples styling and interaction, which feels cleaner to me.

The `article` tag is ideal for content that makes sense on its own, as a unit. A good candidate for our shoe description.

Note that both `hovercard` and `shoe` classes describe *what* the element is, not *how it looks*.

Display this as is in the browser without any style, and it already makes sense, because the content is hierarchically structured with meaningful tags. Show the code to a human, and they understand in a fraction of a second what it’s all about. Give it to someone with bad vision, and their screen reader will have no problem finding the most relevant information to enunciate.

## The CSS

```scss
// app/assets/stylesheets/hovercard.scss

.hovercard {
  position: absolute;
  bottom: 2.5em;
  left: 0;
  padding: 1rem;
  background: #fff;
  border-radius: 0.5rem;
  box-shadow: 0px 5px 10px #ccc;

  &::after {
    content: " ";
    position: absolute;
    top: 100%;
    left: 1rem;
    border-style: solid;
    border-width: 0.5rem;
    border-color: white transparent transparent transparent;
  }
}
```

The CSS code for the hovercard container takes care of the box positioning and its contour style, including the little bubble triangle at the bottom (the `::after` part).

```scss
// app/assets/stylesheets/shoes.scss

#shoes > * {
  position: relative;
}

.hovercard .shoe {
  display: grid;
  grid-gap: 0 1rem;
  grid-template-rows: auto auto auto;
  grid-template-columns: 150px 1fr;

  > * {
    margin: 0;
  }

  img {
    border-radius: 0.5rem;
    grid-column: 1;
    grid-row: 1 / 4;
    width: 100%;
  }

  :not(img) {
    grid-column: 2;
  }

  h1 {
    grid-row: 2;
  }

  p:nth-of-type(1) {
    grid-row: 1;
  }

  p:nth-of-type(2) {
    grid-row: 3;
  }
}
```

The CSS for the inside of the card is in a separate file, and specific to the shoe card. This is more convenient if you plan to use different layouts for each type of hovercard. If not, both style sets could be grouped together, of course.

We are using a grid-based layout here, to easily play with the inner tags placement. Notice how the image is now displayed first on the left, although it’s still last in the HTML. Same with the brand paragraph, visually shown above the shoe name.

One minor annoyance: the hovercard style itself does not include the required `position: relative`, to avoid another wrapper tag just for that. So we have to remember to set that property on the parent tag.

## The JS

One last detail: in *hovercard_controller.js*, I used the `hidden` HTML attribute instead of a `hidden` class. The element will be ignored by screen readers as well that way (which might or might not be the desired effect, depending on the use case).

![Hovercard screenshot](/assets/images/blog/semantic_hovercard.jpg)

The full Rails example app is [available on GitHub](https://github.com/otagi/shoes).