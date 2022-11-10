<div align="center">

# `elixir-auth-facebook`

![img](https://i.stack.imgur.com/pZzc4.png)

_Easily_ add `Facebook` login to your `Elixir` / `Phoenix` Apps
with step-by-step **_detailed_ documentation**.

</div>

## Why?

Facebook authentication is used **_everywhere_**!
More than tens of millions of people use it everyday.
Facebook Login can be used to authenticate people without planning to access their data.

We wanted to create a reusable `Elixir` package
with beginner-friendly instructions and readable code.

## What?

A simple and easy-to-use `Elixir` package that gives you
**Facebook `OAuth` Authentication** for your **web app**
in a few steps with a minimal API.

❗️ If you target Android or IOS, use the SDK.

> If you're new to `Elixir`,
> please see: [dwyl/**learn-elixir**](https://github.com/dwyl/learn-hapi)

## How?

These instructions will guide you through setup in 5 simple steps.
By the end you will have **login with `Facebook`** in your **Web** App.

> **Note**: if you get stuck,
> please let us know by opening an issue!

## Step 1: Create a Facebook app 🆕

You need to have a Facebook developer account. It is free.
You will create an app and get the **credentials**.

#### Step 1.1 Create or use a developer account from your personal Facebook account

Go to <https://developers.facebook.com/apps/>

...after logging in to your facebook account, you can 'Register Now' for a developer account.

#### Step 1.2 Create an App

- select the app type: **"consumer"**
- provide basic info, such as:

  - app name (can be changed)
  - contact name

    ![type](priv/Screenshot%202022-10-16%20at%2014.08.35.png)

#### Step 1.3 Get and save your credentials

Once you are done, you arrive to the Dasboard.
Select **settings**, then **basic**.

![settings](priv/Screenshot%202022-10-17%20at%2017.58.35.png)

You will find your **credentials** there.
Copy the App ID and the App Secret into your `.env` file.

```env
# .env
export FACEBOOK_APP_ID=xxxxx
export FACEBOOK_APP_SECRET=xxxx
```

#### Step 1.4 Specify the base redirect URI

Lastly, you need to set the callback **base URL**.
Your app won't work if a wrong or incomplete base URL is set.

- At the bottom of the form, click on **+ Add platform**

![platform](priv/Screenshot%202022-10-17%20at%2018.04.57.png)

- click on **Web** in the "Select Platform" modal

![select platform](priv/Screenshot%202022-10-17%20at%2018.05.17.png)

- a new input will appear: fill the **Site URL** with:
  <http://localhost:4000>

![base url](priv/Screenshot%202022-10-17%20at%2017.39.49.png)

**Note**: this is the base redirect URI, so it has to be an _absolute_ URI, not only the domain. Make sure you include the `http://` prefix.

### Step 2: use the ElixirAuthFacebook module

You want to display a **login** link in one of your pages.
It will be an external navigation to the Facebook login dialog form.

#### Add a login link in your template ✨

```html
<a class="your-classes" href="{@oauth_facebook_url}">
  <img src={Routes.static_path(@conn, "/images/fb_login.png")}/>
</a>
```

#### Modify the template controller

We know need to generate this "href" address and set it in the assign `@oauth_facebook_url`.
This is done in the controller. You add the code:

```elixir
use MyAppWeb, :controller

def index(conn, _p) do
    oauth_facebook_url =
        ElixirAuthFacebook.generate_oauth_url(conn)

    render(
        conn,
        "indx.html",
        oauth_facebook_url: oauth_facebook_url
    )
end
```

#### Create the `auth/facebook/callback` endpoint 📍

Once the user has filled the dialog form, he will be redirected.

The redirection is set in the router. Add iths line:

```elixir
#MyAppWeb.Router

scope "/", MyAppWeb do
    pipe_through :browser
    get "/auth/facebook/callback",
        FacebookAuthController, :login
end
```

#### Create a `FacebookAuthController`

Add this code

```elixir
defmodule MyAppWeb.FacebookController do
    use MyAppWeb, :controller

    def login(conn, _,_) do

        {:ok, profile} =
            ElixirAuthFacebook.handle_callback(
                conn, params
            )
    end
end
```

It eventually sends back on object which identifies the user. _Done_! 🚀

```elixir
%{
  access_token: "EAAFNaUA6VI8BAPkCCVV6q0U0tf7...",
  email: "xxxxx",
  fb_id: "10223726006128074",
  name: "Harry Potter",
  picture: %{
    "data" => %{
      "height" => 50,
      "is_silhouette" => false,
      "url" => "xxxxx",
      "width" => 50
    }
  },
  session_info: "XAAFNaUA6VI8BACO99qVYqkGPxxxxx"
}
```

You receive a long term "access_token" and a "session_token".
The app can interact with the Facebook eco-system on behalf of the user.
It should be saved in the database, and put them in a session.

However, if you smiply need to authneticate a user, these token are useless.

### _Optional_:

In case of errors in the dialog server/facebook, we use a
termination function.
It puts flash messages and redirects to a chosen path, say "/".

```elixir
def terminate(conn, message, path) do
    conn
    |> Phoenix.Controller.put_flash(:error, message)
    |> Phoenix.Controller.redirect(to: path)
    |> Plug.Conn.halt()
end
```

In you want to overright it, define a custom termination function `happy_end/3`
and pass it as a third optional argument to the callback:

```elixir
{:ok, profile} =
    ElixirAuthFacebook.handle_callback(
        conn,
        params,
        &happy_end/3
    )
```

### Notes 📝

All the flow to build the Login flow can be found here:
<https://developers.facebook.com/docs/facebook-login/guides/advanced/manual-flow>

#### Meta / Privacy Concerns? 🔐

No cookie is set. It just provides a user authentication. You have the tokens to do more,
❗️ but need an [opinion(?) on Meta](https://archive.ph/epKXZ).
Use this package as a last resort if you have no other option!

#### Data deletion?

If you want to use the package to access Metas' eco-system, then you need to provide [a data deletion option](https://developers.facebook.com/docs/facebook-login/overview)

❗️ To be compliant with GDPR guidelines, you must provide the following:

- A way in your app for users to request their data be deleted
- A contact email address that people can use to reach you to request their data be deleted
- An implementation of the data deletion callback
