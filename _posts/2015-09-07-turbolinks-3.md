---
title: Partial updates with Turbolinks 3
---

## Turbolinks?

The goal of [Turbolinks][turbolinks] is to make web pages load faster. The first one will load like usual. Then, when the visitor clicks on a link, the next page is loaded through an asynchronous call. The server sends back the HTML, and Turbolinks updates the page's `body` with the new content, leaving the `head` untouched. Your big *application.js* is not fetched nor parsed again, which saves some precious milliseconds before everything is rendered.

Version 3 of Turbolinks adds the possibility of updating any HTML tag of your choice, instead of `body`.

We'll use a simple Comment resource with standard CRUD as an example, and progressively modify the code to take advantage of Turbolinks 3 for the C(reate) part.

## Pre-flight checklist

Start by creating a new Rails app:

{% highlight bash %}
$ rails new turbolinks3-playground --skip-bundle
{% endhighlight %}

Use the Turbolinks master branch: (until 3.0 is officially out)

{% highlight ruby %}
# Gemfile
gem 'turbolinks', git: 'https://github.com/rails/turbolinks.git'
{% endhighlight %}

And don't forget to

{% highlight bash %}
$ bundle install
{% endhighlight %}

in the project's folder.

## Comments

First, generate the resource:

{% highlight bash %}
$ rails generate scaffold Comment content:text
$ rake db:migrate
{% endhighlight %}

Now, instead of using a separate page to create a new comment, we will create it directly on the comments index page.

In the comments index view, replace the `link_to 'New comment'` with the comment form:


{% highlight erb %}
<%# index.html.erb %>

<%#= link_to 'New Comment', new_comment_path %>

<%= form_for(@comment) do |f| %>
  <div class="field">
    <%= f.label :content, "New comment" %><br>
    <%= f.text_area :content %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
{% endhighlight %}

Then add the new comment instance in the controller:

{% highlight ruby %}
# comments_controller.rb
class CommentsController < ApplicationController
  def index
    @comments = Comment.all
    @comment = Comment.new
  end
end
{% endhighlight %}

If you try to create a new comment now, it will redirect to the `show` action. Let's change that in the controller:

{% highlight ruby %}
# comments_controller.rb
class CommentsController < ApplicationController
  def create
    @comment = Comment.new(comment_params)
    if @comment.save
      # redirect_to @comment, notice: 'Comment was successfully created.'
      redirect_to comments_path, notice: 'Comment was successfully created.'
    else
      render :new
    end
  end
end
{% endhighlight %}

Now when we submit a new comment, it redirects to the index. But the whole page is reloaded, without any Turbolinks benefit. To change that, we will submit our form with an asynchronous call:

{% highlight erb %}
<%# index.html.erb %>
<%= form_for(@comment, remote: true) do |f| %>
  ...
<% end %>
{% endhighlight %}

Note that all content is still refreshed, but Turbolinks now only updates the `<body>` part. This is essentially what Turbolinks 2 brings to the table.

As a bonus, you can see that Turbolinks displays a progress bar of its own at the top of the page during the Ajax request.

To benefit from the Turbolinks 3 main new feature, partial updates, we just have to set an id for the HTML tag we want to update:

{% highlight erb %}
<%# index.html.erb %>
<table>
  <tbody id="comments_list">
    ...
  </tbody>
</table>
{% endhighlight %}

And add the new `change` parameter to the `redirect_to` call in our controller:

{% highlight ruby %}
# comments_controller.rb
class CommentsController < ApplicationController
  def create
    @comment = Comment.new(comment_params)
    if @comment.save
      redirect_to comments_path, change: 'comments_list', notice: 'Comment was successfully created.'
    else
      render :index
    end
  end
end
{% endhighlight %}

That's it! When we create a new comment, the `<tbody>` is now the only updated part, with just 2 small changes to our code. No need to use a *create.js.erb* view to manually update the HTML with custom Javascript any more!

## Caveats

You may have noticed that the submit generates 2 requests: a POST for the `create` action, and a GET for the `index` action, just after the redirect.

Also, the data returned to the asynchronous call is the whole HTML page, not a JSON string. The Turbolinks Javascript code will do a simple copy-replace of the relevant HTML parts.

If you are freaking out right now, because of this relative overhead on the amount of data being transmitted, it's still time to revert back to old methods (custom JS views), or to look at that new Javascript framework on the block.

But before you do, please take a moment to consider the benefits of having less Javascript to maintain, and keeping a relatively standard Rails app code base. For a lot of use cases, Turbolinks may well be more than enough.

Version 3 is set to be officially released alongside Rails 5 this fall 2015. While the current master warns that specs might still change, it's already fairly useable with Rails 4.

## Going further

The [gem's README][readme] has many more capabilities listed. You can of course update multiple tags at once, hook to page events, change the progress bar style, or force tags to update (or not) thanks to new data attributes.

There is a [companion demo app][demo] for this article on Github. Feel free to browse the code and experiement with your own changes.

[turbolinks]: https://github.com/rails/turbolinks/
[readme]: https://github.com/rails/turbolinks/blob/master/README.md
[demo]: https://github.com/otagi/turbolinks3-playground