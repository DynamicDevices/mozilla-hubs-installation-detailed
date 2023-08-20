## Introduction

After [installing Mozilla hubs on local](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/README.md) and [Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md) I started to modify it.

## Where is the database schema?

[See this first](https://www.youtube.com/watch?v=X9AggnaEXrM)

## How to make hubs scene with blender

[Make scene](https://www.youtube.com/watch?v=ldHwbnMMKVY)

## Reduce logs info for webpack server based

Many logs message on hubs, hubs admin, spoke terminal sometimes makes us more confused. We can reduce it by adding this on `devServer` object on `webpack.config.js`

<details>
  <summary>Show Code</summary>

```js
devServer: {
  // This is for reducing logs output on terminal
  quiet: false,
  noInfo: false,
  stats: {
    assets: false,
    children: false,
    chunks: false,
    chunkModules: false,
    colors: true,
    entrypoints: false,
    hash: false,
    modules: false,
    timings: false,
    version: false
  }
}
```

</details>

<br>

## How to login & create admin account?

[See official tutorial](https://github.com/mozilla/reticulum#5-logging-in)
<br>

## How to modify UI (web2D)?

<details>

### 1. Take a look on the overview or make your own

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

### 2. Reverse engineering from the Output

Output -> ComponentChild -> ComponentParent -> etc ... -> Main.js -> Main.html

if you see some teks on the hubs output ( on browser) for example is "Avatar Settings" then copy that string and goto your vscode and find using search tool. then you can find the `ComponentChild` and search again what component which calling it until you found the thing that you need.

### 3. Becareful when editing the code

React js will rerender the page if there is state updated. as my experience if the web are complex. If the component structure is bad sometimes it over rendering. and cause FPS drop, Memory leak.

### 4. Testing

#### 4.1 Memory & FPS

Open memory menu on chrome. (inspect element>memory) and [fps monitor](https://www.bleepingcomputer.com/news/google/google-chrome-rolls-back-fps-meter-changes-after-user-complaints/#:~:text=To%20open%20the%20FPS%20meter,press%20enter%20as%20shown%20below.) on chrome

First, See the memory usage

Try to press all your layout button. and do some frequent actions.

If the memory is growing up maybe there's a problem with your code.

</details>
<br>

## How to modify UI (Web 3D / Aframe Component)?

<details>
![Component 3D](/docs_img/component_3d.png)

Like the position etc. its on the `hub.html` file

hubs using native html and controlled with native javascript.

you can reverse engineering it with search for the class name or id.

for example in vscode you can find the `video-volume-label`. its a class name

![Component 3D](/docs_img/reverse_engineering_3d.png)

You can see the aframe element (the output) and the javascript file which controlling that.

</details>

<br>

## Add Content Security Policy (CSP) rule in reticulum

check on file `add_csp.ex`

## Bypass Email Verification for Login

<details>

Find `lib/ret_web/channels/auth_channel.ex`

Find this function and replace it with this function below:

```
def handle_in("auth_request", %{"email" => email, "origin" => origin}, socket) do
    if !Map.get(socket.assigns, :used) do
      socket = socket |> assign(:used, true)

      account = email |> Account.account_for_email()
      account_disabled = account && account.state == :disabled

      if !account_disabled && (can?(nil, create_account(nil)) || !!account) do
        # Create token + send email
        %LoginToken{identifier_hash: identifier_hash, token: token, payload_key: payload_key} = LoginToken.new_login_token_for_email(email)

        encrypted_payload = %{"email" => email} |> Poison.encode!() |> Crypto.encrypt(payload_key) |> :base64.encode()

        # Just by pass login
        decrypted_payload = encrypted_payload |> :base64.decode() |> Ret.Crypto.decrypt(payload_key) |> Poison.decode!()
        broadcast_credentials_and_payload(identifier_hash, decrypted_payload, socket)
        LoginToken.expire(token)
      end

      {:noreply, socket}
    else
      {:reply, {:error, "Already sent"}, socket}
    end
  end
```

</details>

<br>

## Trick to Create First User as an Admin on Production Server

<details>

When you run hubs on vps. you using this command

```
PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
```

With that command bellow you can't access the log and the iex (interactive elixir)

But for the first time we need verification token right? like in [this section](https://github.com/mozilla/reticulum#5-logging-in) and [creating admin account with that first logged in account](https://github.com/mozilla/reticulum#6-creating-an-admin-user)

We can see and open iex with this command

```
MIX_ENV=prod iex -S mix phx.server
```

with that command you can see the verif token and creating admin account

</details>

<br>
<br>
<br>
<br>

## What you want to know ?

Write it on the issue. maybe i can help you.

<br>
<br>

<a href='https://paypal.me/AlbirrKarim' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://user-images.githubusercontent.com/29292018/186840848-65e25ff9-47e2-424b-bfa0-4ca5d027b346.png' border='0' alt='Donate via paypal' /></a>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

## Also read:

[How to Maintenance Server (Backup, etc)](MAINTENANCE.md)

[Hosting Mozilla Hubs on VPS](VPS_FOR_HUBS.md)

[The Problem I Still Faced](PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](PROBLEM_SOLVED.md)

[Tips for Modification](HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Experience Sharing About Hosting on Other Server](EXPERIENCE.md)
