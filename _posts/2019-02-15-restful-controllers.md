---
layout: post
title:  "RESTful Controllers"
date:   2019-02-14
categories: Ruby Rails REST controllers
---

## Rules of thumb
- Only use the [7 basic Rails actions](https://guides.rubyonrails.org/routing.html#crud-verbs-and-actions): index, new, create, show, edit, update and destroy
- Any custom action in one controller is a RESTful action in another.

## How to move from custom action to new controller?
- A [member route](https://guides.rubyonrails.org/routing.html#adding-member-routes) is a nested [resource](https://guides.rubyonrails.org/routing.html#singular-resources)
- A [collection route](https://guides.rubyonrails.org/routing.html#adding-collection-routes) is a new [resources](https://guides.rubyonrails.org/routing.html#resources-on-the-web)

- Your custom actions are not actions anymore!
  - You do not like a post, you create (post) a like for a post
  - You do not unlike a post, you destroy (delete) a like for a post

## Use the following actions when creating the new controllers:
  - `index`: When you get a collection of resources. (not possible/needed when nested)
  - `create`: When you create a record in the db.
  - `show`: When you get a specific resource.
  - `update`: When you update a record in the db.
  - `destroy`: When you remove a record in the db.

#### Create or Update?
- If the "#like" action means you create a new "like" record in the db, it should be a `create` action on the new controller.
- If the "#like" action means you update the like counter on an article, it should be an `update` action on the new controller.

#### Destroy or Update?
- If the "#unlike" action means you destroy a db record, it should be a `destroy` action on the new controller
- If the "#unlike" action means you update the like count on an article, it should be a `update` action on the new controller.

## Examples
Let's take the `users` resource:
```ruby
  resources :users, except: [:new] do
    member do
      put :follow
      put :unfollow
      ...
    end
    collection do
      ...
      get :activity
    end
  end
```

### Example 1: Follow and Unfollow on User
Let's say we want to move follow and unfollow to a new controller.
- It is in the `member` block for user, so we are going to nest it in `users`.
- `follow` creates a new record in the db so we make it a `create` action.
- `unfollow` destroys a record in the db so we make it a `destroy` action.
```ruby
  resources :users, except: [:new] do
    ...
    resource :follows, module: 'users', only: [:create, :destroy]
  end
```

Resulting in the following routes:
```batch
user_follows POST       /users/:user_id/follows(.:format)     users/follows#create
             DELETE     /users/:user_id/follows(.:format)     users/follows#destroy
```

Next we create a new controller `app/controllers/users/follows_controller.rb`
```ruby
class Users::FollowsController < ApplicationController
  def create
	# create follow record
  end

  def destroy
    # destroy follow record 
  end

  private

  def user
    @user ||= User.find(params[:user_id])
  end
end
```

We now have the following structure:
```
.
├── controllers
│   └── users
│   │   └── follows_controller.rb
│   └── users_controller.rb
```

Next we need to find and replace the old paths with the new ones.
- put: follow_user_path -> create: user_follows_path
- put: unfollow_user_path -> destroy: user_follows_path

### Example 2: Activity
We want to create an activities controller, but we can't nest this in users because the custom `#activity` action is set on the collection.

So we will create a `controllers/activities_controller.rb`
```
.
├── controllers
│   └── users
│   │   └── follows_controller.rb
│   ├── activities_controller.rb
│   └── users_controller.rb

```

Add our new routes:
```ruby
  resources :activities, only: :index
  resources :users, except: [:new] do
    ...
  end
```
giving us:
```
activities GET        /activities(.:format)            activities#index
```

And then in our controller
```ruby
class ActivitiesController < ApplicationController
  def index
    # return all activities
  end
end
```

## Sources:
- https://dzone.com/articles/fighting-custom-actions-in-rails-controllers
- http://jeromedalbert.com/how-dhh-organizes-his-rails-controllers/
- https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api
