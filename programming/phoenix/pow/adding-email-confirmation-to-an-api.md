# Adding Email Confirmation to a Pow API

This document is assuming you already have a basic Pow enabled API interface.
If that's not the case, first check out the [Pow API Guide](https://github.com/danschultzer/pow/blob/master/guides/api.md).

## Rough Guide Notice

This is my first pass at this.  I've had to do some hackish
changes.  Also, not all features of the `PowEmailConfirmation` extension are
implemented.  This is is more of a "What one person did to get it to work"
and not a "officially sanctioned guide."

## Mailer

First, make sure a mailer is configured.  Here's a fake mailer from the README.  Place in `WEB_PATH/pow_mailer.ex`.

```elixir
defmodule MyAppWeb.PowMailer do
  use Pow.Phoenix.Mailer
  require Logger

  def cast(%{user: user, subject: subject, text: text, html: html, assigns: _assigns}) do
    # Build email struct to be used in `process/1`

    %{to: user.email, subject: subject, text: text, html: html}
  end

  def process(email) do
    # Send email

    Logger.debug("E-mail sent: #{inspect email}")
  end
end
```

## Basic Extension Setup

### Generate and run migrations

in shell:
```sh
mix pow.extension.ecto.gen.migrations --extension PowEmailConfirmation
mix.ecto.migrate
```

### Add extensions to Config

in `config.exs`, add the `extensions` and `controller_callbacks` to the pow config:
```elixir
config :my_app, :pow,
  user: MyApp.Users.User,
  repo: MyApp.Repo,
  extensions: [PowEmailConfirmation],                              # Line Added
  controller_callbacks: Pow.Extension.Phoenix.ControllerCallbacks  # Line Added
```

### Add extensions to User

1. Add the extension schema near the top of `user.ex`:
```elixir
  use Pow.Extension.Ecto.Schema,
    extensions: [PowResetPassword, PowEmailConfirmation]
```

2. Add `|> pow_extension_changeset(attrs)` to your changeset in `user.ex`.
If you don't have a changeset defined in `user.ex` yet, you can use this one:
```elixir
  def changeset(user_or_changeset, attrs) do
    user_or_changeset
    |> pow_changeset(attrs)
    |> pow_extension_changeset(attrs)
  end
```

Example of my entire User.ex (note I have previously added one custom field,
unrelated to Email Confirm).

```elixir
defmodule MyApp.Users.User do
  @moduledoc """
  Represents a user of the system.
  Managed by the "pow" library.
  """
  use Ecto.Schema
  use Pow.Ecto.Schema

  use Pow.Extension.Ecto.Schema,
    extensions: [PowResetPassword, PowEmailConfirmation]

  schema "users" do
    pow_user_fields()
    field(:alias, :string)

    timestamps()
  end

  def changeset(user_or_changeset, attrs) do
    user_or_changeset
    |> pow_changeset(attrs)
    |> pow_extension_changeset(attrs)
    |> Ecto.Changeset.cast(attrs, [:alias])
    |> Ecto.Changeset.validate_required([:alias])
    |> Ecto.Changeset.unique_constraint(:alias)
  end
end
```

## Make the API send a confirmation email on registration

### Set front end URL in config

First, we will need to tell our phoenix app the URL for registration
it should generate.  In my case, I have a react app frontend using
the phoenix app as an API server, so the user needs to click on a
URL that goes to the react app.

I added a `front_end_email_confirm_url` key to my WebEndpoint config,
inside `dev.exs`, `test.exs`, and `prod.exs`.  These are specific
to my frontend.

`dev.exs`
```elixir
config :myapp, MyAppWeb.Endpoint,
  http: [port: 4000],
  debug_errors: true,
  code_reloader: true,
  check_origin: false,
  watchers: [ .. truncated for brevity .. ],
  front_end_email_confirm_url: "http://localhost:3000/confirm-email/{token}"   # line added
```

Also add to your `test.exs` and `prod.exs`.  In prod, use your prod url.  Your
dev url may be different than mine.

## Make the registration controller send the email.

I added a call to `send_confirmation_email()` in the `{:ok, ...}` path
of the `create` function.  I had to copy the function and modify it slightly,
since I needed it to make a url that pointed to my own frontend.

in `lib/my_app_web/controllers/api/v1/registration_controller.ex`

```elixir
defmodule MyAppWeb.API.V1.RegistrationController do
  use MyAppWeb, :controller

  alias Ecto.Changeset
  alias Plug.Conn
  alias MyAppWeb.ErrorHelpers

  @spec create(Conn.t(), map()) :: Conn.t()
  def create(conn, %{"user" => user_params}) do
    conn
    |> Pow.Plug.create_user(user_params)
    |> case do
      {:ok, user, conn} ->
        send_confirmation_email(user, conn)  # Line Added

        json(conn, %{
          data: %{
            token: conn.private[:api_auth_token],
            renew_token: conn.private[:api_renew_token]
          }
        })

      {:error, changeset, conn} ->
        errors = Changeset.traverse_errors(changeset, &ErrorHelpers.translate_error/1)

        conn
        |> put_status(500)
        |> json(%{error: %{status: 500, message: "Couldn't create user", errors: errors}})
    end
  end

  ## Two Functions Added below

  _ = """
  Sends a confirmation e-mail to the user.

  The user struct passed to the mailer will have the `:email` set to the
  `:unconfirmed_email` value if `:unconfirmed_email` is set.

  *** This is copied and modified from
  ./lib/extensions/email_confirmation/phoenix/controllers/controller_callbacks.ex
  in the 'pow' library.
  REASON: Customize the url sent to include the front-end. ***
  """

  @spec send_confirmation_email(map(), Conn.t()) :: any()
  defp send_confirmation_email(user, conn) do
    url = confirmation_url(user.email_confirmation_token)
    unconfirmed_user = %{user | email: user.unconfirmed_email || user.email}
    email = PowEmailConfirmation.Phoenix.Mailer.email_confirmation(conn, unconfirmed_user, url)

    Pow.Phoenix.Mailer.deliver(conn, email)
  end

  defp confirmation_url(token) do
    Application.get_env(:spades, MyAppWeb.Endpoint)[:front_end_email_confirm_url]
    |> String.replace("{token}", token)
  end
end
```

## Add a Controller that verifies email addresses

My api/v1 section of the `router.ex`:

```elixir
  scope "/api/v1", MyAppWeb.API.V1, as: :api_v1 do
    pipe_through :api

    resources "/registration", RegistrationController, singleton: true, only: [:create]
    resources "/session", SessionController, singleton: true, only: [:create, :delete]
    post "/session/renew", SessionController, :renew

    resources "/confirm-email", ConfirmationController, only: [:show]  # Line Added
  end
```

Add the controller, in: `lib/my_app_web/controllers/api/v1/confirmation_controller.ex`

```elixir
defmodule MyAppWeb.API.V1.ConfirmationController do
  use MyAppWeb, :controller

  alias Plug.Conn

  @spec show(Conn.t(), map()) :: Conn.t()
  def show(conn, %{"id" => token}) do
    case PowEmailConfirmation.Plug.confirm_email(conn, token) do
      {:ok, _user, conn} ->
        conn
        |> json(%{success: %{message: "Email confirmed"}})

      {:error, _changeset, conn} ->
        conn
        |> put_status(401)
        |> json(%{error: %{status: 401, message: "Invalid confirmation code"}})
    end
  end
end
```

## Make your front end query the confirmation controller

This part's up to you.  When the user clicks on the link you specified
in the config above, it should hit the confirmation controller you just made.

```javascript
axios(`/api/v1/confirm-email/${confirm_token}`)
```

is a decent place to start in javascript, but this is outside the scope of pow.

## What's missing

Note what we don't do:

* The entire "change email" flow is missing.
* There is no login denial on unconfirmed email address.
* No integration with `PowInvitation`.
* No way to resend confirmation email.

However, I was able to get an email sent on registration with a link that
updates that user as confirmed in the database, so the basics are working.

