---
layout: post
title: "How to create a dynamic search bar with Hotwire?"
date: 2023-12-03 20:00:00 +0300
---

# How to create a dynamic search bar with Hotwire?

Ever envisioned seamlessly incorporating a dynamic search bar into your web app, instantly triggered upon typing? Discover how to effortlessly implement this feature in your Rails 7 application using the powerful combination of Stimulus and TurboStream, a.k.a. Hotwire.
### Used gems:
- `faker` - to generate fake books
- `requestjs-rails` - to simplify the stimulus controller

### Example behaviour:
 
![Screenshot 2024-03-13 at 21 27 52](https://i.ibb.co/7bFwTgS/search.gif)

In this straightforward example, we're using a Book model with just one attribute: its title. The goal is to perform searches based on these book titles.

## Routes
Basic routes.

```rb
#routes.rb

Rails.application.routes.draw do
  root "books#index"

  resources :books do
    collection do
      get :search
    end
  end
end
```

## Migration
Before creating any book we first need to create a simple migration.

```rb
#create_books.rb

class CreateBooks < ActiveRecord::Migration[7.1]
  def change
    create_table :books do |t|
      t.string :title

      t.timestamps
    end
  end
end
```

## Seed
Creation of the dummy books with faker gem.

```rb
#seed.rb

100.times do
  Book.create(title: Faker::Book.title)
end
```
For the sake of the example app here, I run the migrations and seed file. Finish setting up our basic app.

## Views
Book partial.

```erb
#_book.html.erb

<div id="<%= dom_id book %>">
  <div>
    <%= link_to book.title, book %>
  </div>
</div>
```

Index view file.
```erb
#index.html.erb

<h1>Books</h1>

<input
  type="text"
  placeholder="Search Books"
  data-controller="search"               <-- Connecting stimulus controller 'search_controller.js'
  data-action="input->search#search"     <-- This search() method will be called inside our stimulus controller in every keystroke 
  data-search-target="searchBox"         <-- Sets the target name so we can get this spesifc DOM and read its input value inside the stimulus controller
  data-path="<%= search_books_path %>"   <-- Sets the path. We are going to use this path to send our search requests inside the stimulus controller
/>

<div id="books">
  <ol>
    <% @books.each do |book| %>
    <li>
      <%= render "book", book: %>
    </li>
    <% end %>
  </ol>
</div>

```

This turbo_stream view plays a crucial role in our functionality as it replaces the DOM with the id 'books' with the updated search results.

```erb
#search.turbo_stream.erb

<%= turbo_stream.replace "books" do %>
  <div id="books">

    <% if @books.empty? %>
      <h4>No books were found!</h4>
    <% else %>
      <ol>
        <% @books.each do |book| %>
          <li>
            <%= render 'book', book: book %>
          </li>
        <% end %>
      </ol>
    <% end %>

  </div>
<% end %>
```

## JS Controller
This is the search_controller.js and it constructs the search query.

```js
#search_controller.js

import { Controller } from "@hotwired/stimulus";
import { get } from "@rails/request.js"

export default class extends Controller {
  static targets = ["searchBox"]

  search() {
    let path = this.searchBoxTarget.dataset.path;
    
    let params = new URLSearchParams({
      query: this.searchBoxTarget.value
    });

    // request.js gem allows us to send turbo-stream requests from the stimulus controller
    get(`${path}?${params}`, { responseKind: "turbo-stream" });
  }
} 
```

## Controllers

This is how the books_controller.rb looks like, it gets the job done. You can customize it as you wish. Maybe add limit to the seach query or add pagination :)

```rb
#books_controller.rb

class BooksController < ApplicationController
  def index
    @books = Book.all
  end

  def search
    @books = if params[:query].present?
               Book.where('title LIKE ?', "%#{params[:query]}%")
             else
               Book.all
             end
  end
end
```

# Conclusion

This is how you can integrate a fundamental search bar into your Ruby on Rails application using Hotwire.

Feel free to tailor this approach to your specific needs—consider introducing a delay in the Stimulus controller to prevent requests after each keystroke or implementing a limit in the books controller for added customization.
