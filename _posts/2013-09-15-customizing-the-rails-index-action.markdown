---
layout: post
title: "Customizing the Rails Index Action"
categories: rails routing
---

Assume we have a list of blog posts. The corresponding controller might look like the following:

``` ruby
class PostsController < ApplicationController
  def index
    @posts = Post.all.entries
  end

  # ... other restful actions
end
```

The routes file would likewise be fairly straightforward:

``` ruby
MyCoolBlog::Application.routes.draw do
  resources :posts
end
```

Everything is fine and well, and completely RESTful. So what happens when we want to introduce an archive feature and provide a route to show all the archived posts?

Let's try to make our `index` method smarter. If we have a link to our archived posts somehwere on our site, we could just pass in an `archived=true` parameter in the `get` request.

``` ruby
# in some ERB template linking to the archived posts
<%= link_to "Archived Posts", posts_path(archived: true) %>
```

This will create the following URL:

```
/posts?archived=true
```

From there, in the index action, we could simply branch on the presence of this parameter.

``` ruby
class PostsController < ApplicationController
  def index
    if params[:archived] && params[:archived] == true
      @posts = Post.where(archived: true).entries
    else
      @posts = Post.all.entries
    end
  end
end
```

The code above works, but leaves something to be desired. So how can we improve it?

Enter collection routes. If we return to our routes file, and create a new collection under the `:posts` resources, we'll have the desired behavior without the hacky feeling.

``` ruby
MyCoolBlog::Application.routes.draw do
  resources :posts do
    collection do
      get :archived
    end
  end
end
```

By using the above formulation, we now have an `archived_posts_path` helper method. The next step is to update our controller:

``` ruby
class PostsController < ApplicationController
  def index
    @posts = Post.where(archived: false).entries
  end

  def archived
    @posts = Post.where(archived: true).entries
    render action: :index
  end
end
```

And voila, we now can create the following link in our ERB template:

``` ruby
<%= link_to "Archived Posts", archived_posts_path %>
```

The link will generate the following URL: `/posts/archived`. And Rails will route the request to our `archived` method in the Posts controller which still uses the same template for our regular index.
