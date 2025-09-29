---
layout: post
title:  "Soft-hypen characters in json text values"
author: Arild Eide
tags: Nibe uplink json Alacritty 
---


## The backdrop
In 2019 we installed a Nibe F1155 Ground Source Heat pump. In essence, this is an expensive cabinet in our basement that circulates glycol into a 200 meter deep energy well to extract geothermal heat to substantially reduce the monthly electricity bill. BTW, if the [state-funded fixed price scheme](https://www.regjeringen.no/en/aktuelt/norgespris-skal-sikre-forutsigbare-og-stabile-strompriser-for-folk/id3090849/) was even heard of at the time we would certainly not have been able to justify the investment. In any case, this is the cabinet:


![Heat_pump_photo](/images/f1155.jpg)


For years I have a had script running to fetch various useful and less useful parameters from Nibe's API, but this broke down when they decided to redo their service entirely. This morning I set out to restore the every-minute read of paramaters and this post presents a small but interesting detour hit when doing so.


## Authenticating
The API allows authentication using client credentials flow, so a single call to the *token* endpoint is all it takes to obtain an access token. In Nushell, setting to request headers, *Authorization* and *Content-Type*:
```
let token = (http post --headers
  {Authorization:$'Basic ($'($client_id):($client_secret)' | encode base64 | str trim)'
  Content-Type: application/x-www-form-urlencoded}
  https://api.myuplink.com/oauth/token
  'grant_type=client_credentials&scope=READSYSTEM' | get 'access_token')
```

The access token is now assigned to `$token` and can be used to interact with the API for an hour.
First we obtain the device id:


## Fetching data
```
let device_id =
  (http get -r --headers {Authorization: $'Bearer ($token)'}
  https://api.myuplink.com/v2/systems/me
  |  get systems.devices.0.id.0)

// Actual API resonse
{
  "page": 1,
  "itemsPerPage": 10,
  "numItems": 1,
  "systems": [
    {
      "systemId": "***",
      "name": "F1155-12 EM 3x230",
      "securityLevel": "admin",
      "hasAlarm": false,
      "country": "Norway",
      "devices": [
        {
          "id": "***",
          "connectionState": "Connected",
          "currentFwVersion": "9719R2",
          "deviceType": "",
          "product": {
            "serialNumber": "***",
            "name": "F1155-12 EM 3x230"
          }
        }
      ]
    }
  ]
}
```

Let's fetch the parameter *priority* which conveys if and for what purpose the heat pump is running right now, this is parameter *49994*:

```
http get --headers {Authorization: $'Bearer ($token)'}
  $'https://api.myuplink.com/v2/devices/($device_id)/points?parameters=49994'
```

This is were we hit the problem, in the screenshot below - what is going on with *Priority*?
![Screenshot character issue](/images/nibe_priority.png)

Further progress fully stopped - we need to understand why there is a garbled character in the output!

## What can possible introduce the character encoding issue?

- The API itself? Possibly, but the issue is not present when running the same request using *curl* in *iterm2* which suggests a local issue
- Alacritty - the terminal?
- Zellij - the terminal multiplexer?
- Nushell or its `http get` command?

ChatGPT claims it is a font encoding issue and that we should select a different system font for Alacritty. No difference.

The next step is to eliminate both Zellij and Nushell and we fined issue present even when doing a regular curl request in Alacritty using zsh and without Zellij, under both MacOS and Ubuntu.

## A closer look on the response body

ChatGPT suggests that we pipe the API response to `od` to see any non-visible characters:
```
http get -r  --headers {Authorization: $'Bearer ($token)'} $'https://api.myuplink.com/v2/
  devices/($device_id)/points?parameters=49994' | od -c
```
This is the output:
![Scre, nibe_response_od.png)
In the red ellipsis we see a two-byte code point right before the last *i* in *priority*, *302 255*. This code point is the soft-hypen or SHY or discretionary hyphen that is used to signal where in a word a hyphen should be placed when rendering would benefit from breaking the word into lines.

For reasons I have not looked into, Alacritty renders the double dagger substitution character when it encounters the SHY character, at least on my two systems. It is also hard to see how any usage of the SHY in json text value would be useful. 

## Removing the SHY
```
http get -r  --headers {Authorization: $'Bearer ($token)'}
  $'https://api.myuplink.com/v2/devices/($device_id)/points?parameters=49994'
  | str replace "\u{AD}" "" | from json
```
Replacing the SHY with the empty string fixes the rendering issue:
![Screenshot shy removed](/images/nibe_shy_removed.png)

## Next steps
With this issue solved we can progress what we really want to build a Tauri-based Android app to monitor the heat pump, in much greater detail than then [[official app](https://play.google.com/store/apps/details?id=com.myuplink.consumer). 
