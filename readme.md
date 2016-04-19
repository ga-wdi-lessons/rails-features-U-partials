# Rails Edit

## Learning Objectives

- Implement the update feature for a model in rails
- Introduce application layout
- Convert edit & new forms to partials
- Convert show/index to render partials for collections
- Utilize Rails validations to protect application's state from bad data

## Framing (5 min)

We've learned to do a lot so far: we've created simple apps that are able to interact with a database. We can create, read, and destroy entries in our database but we can't yet update them. Currently the UI of our pages varies a lot between views. We can add redundant pieces to multiple views to get a consistent UI but there are better, more DRY ways to achieve the same end.

# Update Feature

Create, read, and destroy are all you need if nothing ever changes and your users never make mistakes. So why bother learning update?

### Add appropriate routes (10 min)

The path `/todos/2/edit` is asking to go to the edit view for the todo with id=2

![routing error](images/routing_error.png)

I can check our current url paths and matching controller actions by running

```
$ rake routes
```

To fix this error I need to add a route for `edit`. While I'm there I'll go ahead and add one for `update`

- *Edit* will respond to the GET request with the view for the edit form
- *Update* will respond to the PATCH request sent by changing the appropriate entries in the database

```rb
# avoid if using all 7 actions
Rails.application.routes.draw do
  resources :todos, only: [:index, :show, :new, :create, :destroy, :edit, :update]
end
```
This will also change what I see if I run `rake routes` again.

### Remove `only:`

Now that I have all 7 actions listed for resources I no longer need to say `only:`

```rb
# recommended
Rails.application.routes.draw do
  resources :todos
end
```

If I run `rake routes` one more time I'll notice nothing changed.

Q. Why did I need to say `:only` to begin with?

<details>
<summary>
A.
</summary>
<br>
We don't want to implement routes that we don't support. Listing a route but not supporting it results in a 500 error, where as not having a route results in a 404 error (what we want).
<br>
</details>

### Controller edit -get/read (5 min)

If I refresh the page I get a new error 😄!

![unkown action](images/unkown_action.png)

To fix this new error I add a controller action

```rb
def edit
  @todo = Todo.find(params[:id])
end
```

### Build edit form view

If I refresh the page again I get another new error 😄!

![missing template](images/missing_template.png)

Create a new file `app/views/todos/edit.html.erb` and copy existing `new` form to build the edit form view. Then add a title to both to help distinguish the two apart.

```html
<h2>Edit</h2>
```

### You do: Doc dive! - Form Helpers (10 min)

Read [form helpers](http://guides.rubyonrails.org/form_helpers.html#helpers-for-generating-form-elements) from 1.3 _Helpers for Generating Form Elements_ to 2.4 _PATCH, PUT, or DELETE_

Try to write down answers to the following questions:

Q. What is a form helper?

Q. How is the key used in params controlled?

Q. What are the long and short styles of invoking `form_for`? Is there an advantage to one or the other?

Q. Why did we not have to specify the method for the new & edit forms?

### Controller update -patch/update (5 min)

If I try submitting the form now I'll again get the `Unknown action` error, this time for the `update action`. To fix it I need to add the appropriate controller action:

The private method `todo_params` we created is called '[Strong Params](http://edgeguides.rubyonrails.org/action_controller_overview.html#strong-parameters)' and it prevents [mass assingment attacks](https://en.wikipedia.org/wiki/Mass_assignment_vulnerability)

```rb
def update
  @todo = Todo.find(params[:id])
  @todo.update(todo_params)
  redirect_to todo_path(@todo)
end
```

### Add links from an existing page to the edit page (5 min)

I should now be able to edit/update my todos. It's awkward to manually got to `/edit`, a link to the page would be much better.

Let's add a link_to helper to our show page
```erb
<h2><%= link_to "Edit", edit_todo_path(@todo) %></h2>
```

### Edit data, explore in index/show

### Break! (10 min)

### You do: Add Edit feature to tunr (20min)

1. Clone [this tunr repo](https://github.com/andrewsunglaekim/tunr_features/tree/new-create-delete)
  - `cd` into it
  - `checkout` the solution from yesterday
  - then create a new branch and switch to it `checkout -b <branchname>`

  ```
  $ git checkout new-create-delete
  $ git checkout -b add-edit-feature
  ```
2. Add the appropriate route for edit

3. Add controller actions for routes

4. Create an `edit.html.erb` view and copy in the new form, and modify it for edit

5. Add links from an existing page to the edit page

6. Test editing data

7. Commit progress!

# Application Layout

Notice that the views we have been creating are not complete HTML pages. They are actually partial pages. By default Rails uses the file `app/views/layous/application.html.erb` to wrap all of these partial pages with the appropriate missing bits of HTML. Rails finds the proper layout and combines it with the view corresponding to the appropriate controller action and renders the two as a single HTML file.

### Yield (5 min)

If you take a look at that file you'll see:

```erb
<%= yield %>
```
> Within the context of a layout, yield identifies a section where content from the view should be inserted. The simplest way to use this is to have a single yield, into which the entire contents of the view currently being rendered is inserted

In other words yield is where the partial pages get inserted.

### Content in layout (5 min)

If we wanted some content to remain the same across pages we could include it directly in the layout. Nav bars and footers are great candidates for things you might want to remain constant.

```html
<nav>
  <h3><a href="/todos">Todos</a></h3>
  <h3><a href="javascript:history.back()">Back</a></h3>
</nav>

<%= yield %>

<footer>
  <p>Get stuff done!</p>
</footer>
```

### Additional layouts

There may be times when you need to use multiple layouts in an application. Rails will first look for a file in `app/views/layouts` with the same base name as the controller. If there is no such controller-specfic layout, Rails uses `app/views/layouts/application.html.erb`.

Instead of using the ones Rails looks for, you can explicitly tell it to use the layout of your choosing with the `render` method in a controller action. Or by using the `layout` method to override the layout conventions for an entire controller.

[Render also does a whole bunch more](http://guides.rubyonrails.org/layouts_and_rendering.html#using-render)

# Form Partials

The two files `new.html.erb` and `edit.html.erb` right now are identical. That's not very DRY. Fortunately Rails has something called *partials* that allow us to clean up our code.

### You do: Doc Dive! - Partials (10 min)

Read  [partials](http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials) from 3.4 _Using Partials_ to 3.4.7 _Spacer Templates_

Try to write down answers to the following questions:

Q. How do you add a partial to a page?

Q. When are partials useful?

Q. What can be included in a partial?

### Convert edit & new forms to partials (5 min)

- Create a new file in views called `_form.html.erb`

- Copy and paste the contents of either `new.html.erb` or `edit.html.erb`

### Render form partials

- Replace the current form with:

```erb
<%= render 'form' %>
```

### You do: Convert forms to partials (10 min)

Continue working on Tunr:

1. Create an appropriate `_form.html.erb`

2. Replace the existing new form with a render of the partial

3. Test creating data

4. Replace the existing edit form with a render of the partial

5. Test updating data

6. Commit progress!

# Other Partials

### Convert show/index to render partial collections

- Create a `_todo.html.erb` file

- Copy the code for displaying the collection data into the `_todo` partial

- Replace the existing code with a render method:

```erb
<%= #render partial: "todo", collection: @todos %>
```

### You do: Use partials in tunr (10 min)

Create a partial for artists and render it in the artists index

# Validations

Flash is especially helpful for correcting the user when they use your app in the wrong way -- entering bogus data, for example.

### Framing (5 min)

In our Rails apps, we've been entering a lot of bogus data -- artists with blank names, for instance. That's fine while we're in development, but we don't want the users of our live app to be able to get away with that.

Q. Based on what we know, how could we prevent users from entering **blank data** into a field?
---

> A. Javascript.

...but that's easily circumvented with Chrome's Inspector.


> A. We could put that in the controller, like so:

```rb
# app/controllers/artists_controller.rb
def create
  if params[:name] == ""
    redirect_to @artist
  else
    @artist = Artist.create(artist_params)
    redirect_to @artist
  end
end
  ```

...but putting it in the controller is going to lead to some pretty unwieldy controller methods. Besides, we may want there to be several routes that create or update an artist, and we'd need to copy and paste that bit of code for each one. That's hardly DRY!

> A. We could put in database constraints, like `NOT NULL` and `UNIQUE`, by going and playing around in SQL.

But...it's SQL.  Rails provides [some helpers](http://edgeguides.rubyonrails.org/active_record_migrations.html#column-modifiers) for managing database constraints.  Beyond those, it can be difficult.

Wouldn't it be nice if, before an Artist record was saved, we could validate that the data entered into its fields were correct?

### You do (15 min)

[The docs, for reference.](http://guides.rubyonrails.org/active_record_validations.html)

- Add the following code to your Artist model. What does each line do?

```rb
class Artist < ActiveRecord::Base
  validates :name, presence: true
  validates :name, uniqueness: true
  validates :name, length: {in: 1..20}
  validates :nationality, inclusion: ["USA", "Canada", "UK"]
end
```

- What's the *un*-DRY version of the following?

```rb
class Artist < ActiveRecord::Base
  validates :name, :nationality, :photo_url, presence: true
  validates :name, uniqueness: true, length: {maximum: 100}
end
```

- Can you figure out what these would do?

```rb
class User < ActiveRecord::Base
  validates :password, confirmation: true
  validates :age, numericality: {only_integer: true, greater_than_or_equal_to: 13}
  validates :country, exclusion: ["North Korea"]
end
```

- What are the differences between the two following snippets?

```rb
class User < ActiveRecord::Base
  validates :password, confirmation: true
  validates :ssn, uniqueness: true, allow_blank: true
end

class User < ActiveRecord::Base
  validates :password, confirmation: true, on: create
  validates :ssn, uniqueness: true, allow_nil: true
end
```

### Custom validations (10 min)

You can also easily create your own custom validations. For instance, this will make Tunr reject any artist named Billy Ray Cyrus:

```rb
# app/models/artist.rb
class Artist < ActiveRecord::Base
  validate :break_billy_rays_achy_breaky_heart

  def break_billy_rays_achy_breaky_heart
    if self.name == "Billy Ray Cyrus"
      errors.add(:name, "cannot be Billy Ray Cyrus, because he doesn't qualify as an artist.")
    end
  end
end
```

Q. Test it out by using the form to create a new artist named "Billy Ray Cyrus". What does `errors.add` do?

<details>
<summary>
A.
</summary>
<br>
Determines what the error message is.
<br>
</details>

---

Q. This validation won't be triggered if the user writes "billy ray cyrus" in all-lowercase. How could you fix that?

<details>
<summary>
A.
</summary>
<br>
`if self.name.downcase == "billy ray cyrus"`
<br>
</details>

---

## Validation in the console (10 min)

Let's test out these validations in the Rails console.

You can leave `rails s` running in one tab; it doesn't matter. In a *different* tab, run `rails c`.

Your "prompt" in the Rails console should look something like this:

```
2.2.3 :001>
```

Now, type in:

```
$ billy = Artist.new(name: "Billy Ray Cyrus")
$ miley = Artist.new(name: "Miley Cyrus")
```

**Q. What does `billy.valid?` do? What about `miley.valid?`**

> A. Tells us whether these Artists will be able to be saved to the database.

**Q. After typing `billy.valid?`, type `billy.errors`. What does it do? What about `billy.errors.full_messages`?**

> A. Contains the error messages generated by trying to save the Artist.

These are the methods Rails uses underneath the surface to trigger validation errors.

**Q. Imagine you see the below line of code has been added to the `views/artists/new.html.erb` page. What does it do?**

```
<%= @artist.errors.full_messages.first if @artist.errors.any? %>
```

> A. It prints out the first error message that was generated.

### Validation triggers

If you only write these:

```
$ billy = Artist.new(name: "Billy Ray Cyrus")
$ billy.errors.full_messages
```
...you'll get a blank array. That's because `.new` doesn't actually make the validations happen. The only methods that *do* are:

```
.valid?
.save
.save!
.create
.create!
.update
.update!
```

...so `.errors.full_messages` must be run after these.

## Break (10 min)

## Bangin' methods (10 min)

Let's try using `.create` instead of `.new`:

```
$ billy = Artist.create(name: "Billy Ray Cyrus")
```

### Rollback

You should see something like:

```
2.2.1 :024 > billy = Artist.create(name: "Billy Ray Cyrus")
   (0.2ms)  BEGIN
   (0.6ms)  ROLLBACK
 => #<Artist id: nil, name: "Billy Ray Cyrus", photo_url: nil, nationality: nil, created_at: nil, updated_at: nil>
```

That `ROLLBACK` indicates that ActiveRecord tried running a SQL command, but it was unsuccessful, so it's undoing -- or "rolling-back" -- any changes that it made.

### With a bang

Now try the same command, but put a bang `!` at the end of `.create`:

```
$ billy = Artist.create!(name: "Billy Ray Cyrus")
```

**Q. What's the difference between `.create` and `.create!`?**


> A. Adding in a bang makes the app throw a fatal error -- that is, "break" -- if a validation fails. Without the bang, it fails "silently".

You can add a bang to both `.create` and `.save`.

### Using a boolean instead of the exception (10 min)

The discussion on validations has so far shown you about 18 different ways a user can "break" your app.

Truth be told, however, we don't usually use `.create` or `.create!` in Rails apps. Instead, we use `.new` and `.save`.

**Q. What's the difference between `.create` and `.new / .save`?**


> A. `.create` is the same thing as `.new` and `.save` run right after each other.

This gives us a way of making sure the user doesn't see a broken app:

```rb
# app/controllers/artists_controller.rb
def create
  @artist = Artist.new(artist_params)
  if @artist.save
    redirect_to @artist
  else
    render :new
  end
end
```

...and in the form view:

```erb
<!-- app/views/artists/_form.html.erb -->
<%= @artist.errors.full_messages.first if @artist.errors.any? %>
<%= form_for @artist do |f| %>
```

This way, one thing happens when the user is successful -- and when they're *not* successful, something else happens and they're told what they did wrong.

#### This ^ is the "right way" to write a Rails controller.

You could also do:

```erb
<!-- app/views/artists/_form.html.erb -->
<% @artist.errors.full_messages.each do |message| %>
  <p><%= message %></p>
<% end %>
```

### Pro (debugging) tip

Add `<%= debug(@artist) %>` to "app/views/artists/new.html.erb" to see information contained in `@artist`.  Try submitting invalid information for a new Artist.


### You do: Apply this to the artists#update action (10 min)

```rb
# app/controllers/artists_controller.rb
def update
  @artist = Artist.find(params[:id])
  if @artist.update(artist_params)
    redirect_to @artist
  else
    render :new
  end
end
```
---


## Exit Ticket (3 min)

Before you leave, plase take ~3 minutes to complete [this exit ticket.](https://docs.google.com/forms/d/1d03NYFphG6m7yAMUY1OlnJZMQWof7Rt6b5MX3Xn4ZPs/viewform)

## Additional Resources

- [RailsGuides: partials](http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials)
- [RailsGuides: form helpers](http://guides.rubyonrails.org/form_helpers.html)
- [RialsGuides: action contorller](http://guides.rubyonrails.org/action_controller_overview.html)
