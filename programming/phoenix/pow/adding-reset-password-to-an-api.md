# Adding Email Confirmation to a Pow API

This document is assuming you already have a basic Pow enabled API interface.
If that's not the case, first check out the [Pow API Guide](https://github.com/danschultzer/pow/blob/master/guides/api.md).

## Rough Guide Notice

This is my first pass at this. I've had to do some hackish changes. This is is
more of a "What one person did to get it to work" and not a "officially
sanctioned guide."

## Mailer

First, make sure a mailer is configured. Here's a fake mailer from the README.
Place in `WEB_PATH/pow_mailer.ex`.

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
  extensions: [PowEmailConfirmation, PowResetPassword],            # Line Added
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

## Review the Request flow

- User requests `/reset-password` on front-end. They're given a form with an
  email address.
- Front end POSTs the email address to `/reset-password` on the Pow API. Pow
  sends email to the user.
- User clicks on link with password reset token. It goes to
  `/reset-password/{reset_token}` on the front end. They're given a form with two
  password fields.
- Front end POSTs the token and the two password fields to
  `/reset-password/update` on the Pow API. POW Either updates the password or
  returns an error.

I will end up grouping all of these together as I post the changes I made
instead of reviewing them in steps.

## Set front end URL in config

Pow will send an email to the user, giving them a link to click that goes to
the front end. But Pow doesn't know anything about the front end. So we have
to specify the url for step 3 above in our config.

I added a `front_end_reset_password_url` key to my WebEndpoint config,
inside `dev.exs`, `test.exs`, and `prod.exs`. These are specific
to my frontend.

`dev.exs`

```elixir
config :myapp, MyAppWeb.Endpoint,
  http: [port: 4000],
  debug_errors: true,
  code_reloader: true,
  check_origin: false,
  watchers: [ .. truncated for brevity .. ],
  front_end_reset_password_url: "http://localhost:3000/reset-password/{token}" # Line Added
```

Also add to your `test.exs` and `prod.exs`. In prod, use your prod url. Your
dev url may be different than mine.

## Create Front End Pages

### /reset-password

`/reset-password` Should display an email address field and post to
`/api/v1/reset-password` on the backend. The shape of the data I posted is:
`user: { email: email }`.

### /reset-password/{reset_token}

`/reset-password/{reset_token}` Should display email and email_confirm fields and
post to `/api/v1/reset-password/update` on the backend. The shape of the data I posted is:
`{id: reset_token, user: { password: password, password_confirm: password_confirm } }`.

## Create Router Entries

In the `/api/v1` scope:

```elixir
post "/reset-password", ResetPasswordController, :create
post "/reset-password/update", ResetPasswordController, :update
```

## Create ResetPasswordController

This is my version, it's probably not the greatest, any pull requests or
suggestions welcome.

In `lib/my_app_web/controllers/api/v1/reset_password_controller.ex`:

```elixir
defmodule MyAppWeb.API.V1.ResetPasswordController do
  use MyAppWeb, :controller

  alias Plug.Conn
  alias PowResetPassword.{Phoenix.Mailer, Plug}

  @spec create(Conn.t(), map()) :: Conn.t()
  def create(conn, %{"user" => user_params}) do
    case Plug.create_reset_token(conn, user_params) do
      {:ok, %{token: token, user: user}, conn} ->
        url = confirmation_url(token)
        deliver_email(conn, user, url)
        conn |> create_success()

      {:error, _any, conn} ->
        conn
        |> create_success()
    end
  end

  def update(conn, %{"user" => user_params, "id" => token}) do
    case Plug.load_user_by_token(conn, token) do
      {:error, conn} ->
        conn
        |> put_status(401)
        |> json(%{error: %{status: 401, message: "Invalid reset token"}})

      {:ok, conn} ->
        update_user(conn, user_params)
    end

    conn
    |> json(%{success: %{message: "Reset password"}})
  end

  defp update_user(conn, user_params) do
    conn = conn |> Plug.assign_reset_password_user(user)

    case Plug.update_user_password(conn, user_params) do
      {:ok, _user, conn} ->
        conn
        |> json(%{success: %{message: "Password updated successfully"}})

      {:error, changeset, conn} ->
        conn
        |> put_status(:unprocessable_entity)
        |> put_view(MyAppWeb.ChangesetView)
        |> render("error.json", changeset: changeset)
    end
  end

  defp create_success(conn) do
    conn |> json(%{success: %{message: "Password reset email sent"}})
  end

  defp confirmation_url(token) do
    Application.get_env(:my_app, MyAppWeb.Endpoint)[:front_end_reset_password_url]
    |> String.replace("{token}", token)
  end

  defp deliver_email(conn, user, url) do
    email = Mailer.reset_password(conn, user, url)
    Pow.Phoenix.Mailer.deliver(conn, email)
  end
end
```

And you're done!

```
Created:       Tue 22 Oct 2019 05:45:21 PM CDT
Last Modified: Mon 16 Mar 2020 11:45:18 AM CDT
```
