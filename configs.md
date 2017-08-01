# Addict Configs

Addict allows you to extend behaviour by fine tuning some configurations.

# Post action hooks

After you register, login, reset password or recover password, you might want to trigger some kind of logic in your application.

Here's an example on how you'd print some debug information after you login a user. The same logic applies to the other endpoints.

The only thing you have to do, is to obey the signature in your custom functions:

```elixir
defmodule MyApp.PostLoginAction do
  def log(conn, status, model) do
    IO.inspect status
    IO.inspect model
    conn
  end
end
```

And add it to the configuration:

```elixir
# config.exs

config :addict,
  (...),
  post_login: &(MyApp.PostLoginAction.log/3)
```

If you want to take different flows according to the success criteria of the action, you can pattern match the arguments:

```elixir
defmodule MyApp.PostLoginAction do
  def log(conn, :ok, model) do
    IO.puts "User logged in successfully"
    conn
  end

  def log(conn, :error, errors) do
    IO.puts "User wasn't able to log in due to:"
    IO.inspect errors
    conn
  end
end
```

These configurations are exposed as:
```
post_login
post_logout
post_register
post_reset_password
post_recover_password
```

# E-mail configurations

When sending e-mails, you most likely want to personalize the way the e-mail is presented to the user.

## From E-mail

Set your `from_email` configuration to whatever e-mail makes sense to you. This is usually a `"no-reply@yourdomain.com"`.

## E-mail Subject

Set the subject for your registration e-mails via `email_register_subject` and for your reset password e-mails via `email_reset_password_subject`.

## E-mail Templates

Addict uses EEx templates to generate the e-mail body. These are set via `email_register_template` and `email_reset_password_template`.Here's an example on how you could do it:

```elixir
defmodule MyApp.EmailTemplates do
  def register do
    """
    <p> This is a registration e-mail. </p>
    <p> You can access your model params as you'd do in a normal EEx template</p>
    <p> For example, to render the e-mail you'd do <%= email %>. </p>
    <p> If you have a name on your model, you can also display it: <%= name %>. </p>
    """
  end
end
```

Then on your configuration file:

```elixir
# config.exs

config :addict,
  (...),
  email_register_template: MyApp.EmailTemplates.register
```

The same logic applies for the `email_reset_password_template`. Just take into consideration that the only available user field will be `email`.

# User model validation

Addict by default validates that the password is at least 6 characters long and the e-mail is valid and unique. If you need to add extra validation, you can define your function validator via `extra_validation`.

Here's an example (pay attention to the function signature):

```elixir
defmodule MyApp.User do
   (...)
   def validate({:ok, _}, user_params) do
     if user_params["name"] == "Murdoch" do
        {:error, [name: "Invalid name. I have this thing against Murdoch."]}
     else
        {:ok, []}
     end
   end

   def validate({:error, errors}, user_params) do
     IO.puts "I could do something fancy here. But I won't."
     {:error, errors}
   end
end
```

And in your configuration file:

```elixir
# config.exs

config :addict,
  (...),
  extra_validation: {MyApp.User, :validate}
```

# CSRF Token

If you're using CSRF token generation, use the `generate_csrf_token` configuration value to pass the function responsible for it. For instance on your configuration file:

```elixir
# config.exs

config :addict,
  (...),
  generate_csrf_token: (fn -> Phoenix.Controller.get_csrf_token end)
```

# Not Logged in Redirect

When using the `Addict.Plugs.Authenticated`, if the user isn't logged in, it will be redirected to `"/login"`. You may change the path by setting the url or path in `not_logged_in_url`:

```elixir
# config.exs

config :addict,
  (...),
  not_logged_in_url: "/some/other/path"
```

# Password Hasher

You can use the `password_hasher` configuration to specify a module to perform the encrypting and decrypting of passwords. The module must implement `hashpwsalt/1` and `checkpw/2`.

# Password Reset Token Time to Expiry

After the password reset email is sent, the reset token will expire in 1 day (86400 seconds). You can set a different length of time (in seconds as an integer) in the config:

```elixir
# config.exs

config :addict,
  (...),
  password_reset_token_time_to_expiry: 120
```

# Template Layout

By default, addict provides you with a base layout in `addict.html`. You can customize the base layout to use your own by supplying a `base_layout` config.
