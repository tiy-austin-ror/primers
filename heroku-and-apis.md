# Working with Heroku (and building APIs)
[Heroku](https://heroku.com) is a website that we are using to host our rails apps.

## Working with Heroku
For the following examples we are assuming you are in a folder called `my_project` and it is a git repo and you are on the `master` branch. So `[my_project][master]$` is the prompt and not part of the command being run or explained.

### Creating a new Heroku app
```sh
[my_project][master]$ heroku create NAME_OF_APP_YOU_WANT_ON_HEROKU
```
### Deploying code to heroku
```sh
[my_project][master]$ git push heroku master
```
Heroku is another remote of your local git repo. That means you **need** to add, commit, *and* push all your changes to heroku for them to reflect on your live site.

### Checking your logs
```sh
[my_project][master]$ heroku logs
```
if you run into an error and heroku says to check the logs. This is how you check then.

### Checking your database on heroku
```sh
[my_project][master]$ heroku run rails console
```
Running this command will open a connection to rails console on heroku. It will be a little slower than running it locally, because every command (and response) has to go to and from the heroku server.

### Migrating the database
```sh
[my_project][master]$ heroku run rake db:migrate
```
Everytime you run `rake db:migrate` locally or otherwise change your database locally, you will need to do the same on heroku.

### Reseting the database (on heroku)
```sh
[my_project][master]$ heroku pg:reset DATABASE_URL
```
_(Note: you need to actually type out DATABASE_URL)_
Sometimes you need to get rid of all the data on your heroku database. Running the above command will clear out the entire database on heroku. This means it will erase all the tables and all the data inside them. You **will** need to run `rake db:migrate` again after this command has been run.


## Building an _(insecure)_ API
### Dealing with Requests
When building an API we want to respond with a format that can be consumed by the majority of our clients. The most popular of these is JSON, so all of our API calls will return a JSON response back. 

To return json from a controller action, you have to specify json as the render format

```rb
  def show
    render json: User.find(params[:id])
  end
```
Rails will call `.to_json` on whatever object you give to the render method.

### Custom JSON Object
We don't have to deal with only Active Record objects. Nearly any ruby object can be turned into a json object.

```rb
  def error
    render json: { error: 'not working' }, status: 500
  end
```

## Adding Methods or Data to an object
Lets say your `User` class has a method called `full_name` that returns the user's `last_name` and `first_name` joined by a comma. Now lets say you want the `full_name` to be in your json response. You can that by calling `.to_json` and supplying it with the right options. [I recommend reading the documentation for to_json](http://apidock.com/rails/ActiveRecord/Serialization/to_json)
```rb
  #In app/models/user.rb
  def full_name
    [last_name, first_name].join(', ')
  end
```
```rb
  #In app/controllers/users_controller.rb
  def user_info
    @user = User.find(params[:id])
    render json: @user.to_json(methods: :full_name)
  end
```
  
### Turning off CSRF
The **C**ross **S**ite **R**equest **F**orgery token is there to help us and our site against [malicious attacks](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29_Prevention_Cheat_Sheet). Because we are making an API that we want anyone to have access to, we are turning off this protection. 
To turn off CSRF protection in our rails projects we need to change one line and add one line to our `ApplicationController`

```rb
class ApplicationController < ActionController::Base
  protect_from_forgery with: :null_session
  skip_before_action :verify_authenticity_token
end
```
Make sure the above two lines are found within your `ApplicationController`

### Turning off CORS
**C**ross **O**rigin **R**esource **S**haring is a protocol created to give you control over what other websites are allowed to use your [resource](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS). This is good, but a public API wants everyone to be able to use our data, so we need to turn it off.

To turn off CORS protection we have to make sure all of our responses have the right headers set. Doing this manually for each route would be tedious and error prone. Luckily, Rails allows us to set our headers globally by adding a few lines to our `Application` class found in `config/application.rb`

```rb
module MyAppName
  class Application < Rails::Application
    config.action_dispatch.default_headers = {
      'Access-Control-Allow-Origin'   => '*',
      'Access-Control-Allow-Methods'  => 'POST, PUT, DELETE, GET, OPTIONS',
      'Access-Control-Request-Method' => '*',
      'Access-Control-Allow-Headers'  => 'Origin, X-Requested-With, Content-Type, Accept, Authorization'
    }
    config.active_record.raise_in_transactional_callbacks = true
end
```
I removed all the comments to save space, but it goes right there inside of that class.

### Setting up a 'catch all' route
When building an API, **consistency is key**. We need to make sure what we respond with is predictable to the end user. Unfortunately, rails throws an error when a user requests a route that we don't have. We need to be able to set what that response is so that it is consistent with the rest of our API. The following code will allow us to do that. _(It is worth noting that this is not a good coding practice and is not something you should be doing in your applications unless there is no alternative)_

Inside of `config/routes.rb` below **all** of your routes, but before the `end` you need to have this line:
`match '*not_found_route', to: 'application#not_found', via: [:get, :post, :put, :delete]`
We are telling rails to match literally *everything* that has not already been matched above that comes in and send it to an action on the `ApplicationController` called `not_found`.

Inside of `app/controllers/application_controller.rb` you will need to add this method:

```rb
 def not_found
   render json: { message: 'Requested route not found' }, status: 404
 end
```
Now we can change the returned json to be anything we want or need. _(Be sure to keep the status as 404)_
