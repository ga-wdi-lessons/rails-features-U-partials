# Rails Edit

## Learning Objectives

- Implement the update feature for a model in rails
  - Copy existing create form to build edit form
  - Add appropriate routes
  - Controller edit (get/read)
  - Controller update (patch/update)
  - Add links to existing pages to edit page
  - Edit data, explore in index/show
  - Edit data, explore it in rails console
- Convert edit & new forms to partials
  - Render form partials
- Convert show/index to render partial collections

## Framing

# Update Feature

### Add appropriate routes

The path `/todos/2/edit` is asking to go to the edit view for the todo with id=2

![routing error](images/routing_error.png)

I can check our current Url paths and matching controller actions by running

```
$ rake routes
```

To fix this error I need to add a route for `edit`. While I're there let's go ahead and add one for `update`

```rb
# avoid if using all 7 actions
Rails.application.routes.draw do
  resources :todos, only: [:index, :show, :new, :create, :destroy, :edit, :update]
end
```
This will also change what I see if I run `rake routes` again.

Now that I have all 7 actions listed for resources I no longer need to say `only:`

```rb
# recommended
Rails.application.routes.draw do
  resources :todos
end
```

If I run `rake routes` again then I'll notice nothing changed.

### Controller edit (get/read)

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

Touch `app/views/todos/edit.html.erb` and copy existing `new` form to build edit form view

### Controller update (patch/update)

If I try submitting the form now I'll again get the `Unknown action` error, this time for the `update action`. To fix it I need to add the appropriate controller action:

```rb
def update
  @todo = Todo.find(params[:id])
  @todo.update(todo_params)
  redirect_to todo_path(@todo)
end
```

### Add links from an existing page to the edit page

I should now be able to edit/update my todos. It's awkward to manually got to `/edit`, a link to the page would be much better.

Let's add a link_to helper to our show page
```rb
<h2><%= link_to "Edit", edit_todo_path(@todo) %></h2>
```

### Edit data, explore in index/show


### You do: Add Edit feature to tunr

1. Clone [this tunr repo](https://github.com/andrewsunglaekim/tunr_features/tree/index-show-solution
)
  - `cd` into it
  - `checkout` the solution from yesterday
  - then create a new branch and switch to it `checkout -b <branchname>`

  ```
  $ git checkout index-show-solution
  $ git checkout -b add-edit-feature
  ```
2. Add the appropriate route for edit

3. Add controller actions for routes

4. Create an `edit.html.erb` view and copy in the new form, and modify it for edit

5. Add links from an existing page to the edit page

6. Test editing data

7. Commit progress!

# Form Partials

The two files `new.html.erb` and `edit.html.erb` right now are identical. That's not very DRY. Fortunately Rails has something called *partials* that allow us to clean up our code.

### Convert edit & new forms to partials

Create a new file in views called `_form.html.erb`

### Render form partials


### You do: Convert forms to partials

Continue working on Tunr:

1. Convert existing new form to use a form partial

2. Test creating data

3. Convert existing edit from to use a partial

4. Test updating data

5. Commit progress!

# Other Partials

### Convert show/index to render partial collections


### You do: Convert artist index page to use collection partial

Modify



## Exit Ticket (3 min)

Before you leave, plase take ~3 minutes to complete [this exit ticket.](https://docs.google.com/forms/d/1d03NYFphG6m7yAMUY1OlnJZMQWof7Rt6b5MX3Xn4ZPs/viewform)

## Additional Resources




- rake routes
  - paths
  - link_to
