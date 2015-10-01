# Waymark

> "Hello API, this is SuperClient on behalf of Bob. This is my authentication token. Can you let me know when /users/1 gets an update so I can reload my view? Thanks!"

This is what we want "realtime push" to do. Waymark lets you do it.

It's made of a websocket server in Node, a messaging queue (currently Redis) and a backend compoenent (currently PHP, or anything that can talk to redis pub-sub). It uses WAMP to talk to a frontend component - just simple calls both ways.

# How It Works

Client:
> "Hello API, this is SuperClient on behalf of Bob. This is my authentication token. Can you let me know when /users/1 gets an update so I can reload my view? Thanks!"

Waymark:
> "Sure!"

PHP backend:
> "User ID 1 is saving an update to their profile. Yo, Redis!"

```
$this->eventListener->listen(UserProfileUpdated::class, function (Event $event) {
  // this is just an example - you could naturally use your router's URL generator or what have you
  $this->waymark->bump('/users/' . $event->getUpdatedUserId);
});
```

Redis:
> "Mmkay, waymark, message coming your way via pubsub."

Waymark:
> "Incoming update on /users/1. Let's check my list here... Oh, SuperClient is interested. Here's his authentication token. Time to make an HTTP request with that token to /users/1..."

HTTP Server:
> "Ooooh, I'm a webserver! I'm a webserver. Hey, PHP, this guy with this token wants a page."

PHP:
> "Ah, that's Bob... My router says he wants /users/1. Yep, he's got access to it. Here's the page."

Waymark:
> "200 OK, page, headers... Alright, SuperClient, there was an update on /users/1, here's your page, headers and everything."

Client:
> "Thanks! Time to parse and re-render my page."


# Installation

Coming up once this cake is out of the oven!

# FAQ

## Ugh, extra HTTP requests?

You betcha. We're doing a _real request_ on behalf of the client, not some kind of workaround. Yes, it's heavier. But a lot of solutions already involve just _pinging_ the client-end library, saying "hey, do an update" - and then it does a manual HTTP request anyway.

## My GET operations have side-effects, like updating an article view count. How do I prevent updates from hitting that?

Waymark includes a `X-Waymark` header to indicate this.
