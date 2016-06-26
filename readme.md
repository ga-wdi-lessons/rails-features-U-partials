# Rails Edit, Update and Partials

## Learning Objectives

- Implement the update feature for a model in rails
- Introduce application layout
- Convert edit & new forms to partials
- Convert show/index to render partials for collections
- Utilize Rails validations to protect application's state from bad data

## Framing (5 min)

We've learned to do a lot so far: we've created simple apps that are able to interact with a database. We can create, read and destroy entries in our database but we can't yet update them.

On top of that, the UI of our pages varies a lot between views. We can add redundant pieces to multiple views to get a consistent UI but there are better, more DRY ways to achieve the same end.

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

<details>
<summary>
  Q. Why did we need to say `:only` to begin with?
</summary>
  <br>
  We don't want to implement routes that we don't support. Listing a route but not supporting it results in a 500 `Internal Server Error`, where as not having a route results in a 404 `Not Found` error. The latter indicates that the user did not make a valid request.
  <br>
</details>

### Add Edit to Controller (5 minutes)

If we refresh the page we get a new error!

![unknown action](images/unkown_action.png)

To fix this new error let's add a controller action...

```rb
def edit
  @todo = Todo.find(params[:id])
end
```

### Build Edit Form View

Let's refresh the page. Another new error!

![missing template](images/missing_template.png)

Create a new file `app/views/todos/edit.html.erb` and copy the existing `new` form into here. Then add a title to both to help distinguish the two apart.

```html
<h2>Edit</h2>
```

### You Do: Doc Dive - Form Helpers (10 minutes)

Read [form helpers](http://guides.rubyonrails.org/form_helpers.html#helpers-for-generating-form-elements) from 1.3 _Helpers for Generating Form Elements_ to 2.4 _PATCH, PUT, or DELETE_

Try to write down answers to the following questions...

- What is a form helper?
- How is the key used in params controlled?
- What are the long and short styles of invoking `form_for`? Is there an advantage to one or the other?
- Why did we not have to specify the method for the new & edit forms?

### Add Update to Controller (5 minutes)

If we try submitting the form now we'll again get the `Unknown action` error, this time for `update`. To fix it, we need to add the appropriate controller action...

```rb
def update
  @todo = Todo.find(params[:id])
  @todo.update(todo_params)
  redirect_to todo_path(@todo)
end
```

> The private method `todo_params` we created is called '[Strong Params](http://edgeguides.rubyonrails.org/action_controller_overview.html#strong-parameters)' and it prevents [mass assignment attacks](https://en.wikipedia.org/wiki/Mass_assignment_vulnerability)

### Add links from an existing page to the edit page (5 min)

I should now be able to edit/update my todos. It's awkward to manually got to `/edit`, a link to the page would be much better.

Let's add a `link_to` helper to our show page...

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



## Exit Ticket (3 min)

Before you leave, plase take ~3 minutes to complete [this exit ticket.](https://docs.google.com/forms/d/1d03NYFphG6m7yAMUY1OlnJZMQWof7Rt6b5MX3Xn4ZPs/viewform)

## Additional Resources

- [RailsGuides: partials](http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials)
- [RailsGuides: form helpers](http://guides.rubyonrails.org/form_helpers.html)
- [RialsGuides: action contorller](http://guides.rubyonrails.org/action_controller_overview.html)
