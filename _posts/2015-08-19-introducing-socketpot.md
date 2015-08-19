---
title: "Introducing Socketpot"
layout: post
---

We have just released a new websocket server that we've named Socketpot. It will handle all push notifications and real-time features for the purpose of polishing the user-experience on your app.

Socketpot lets you opt-in to only the events/features that you're interested in.  Upon connecting to Socketpot, you simply send it an array of subscriptions.

<span class="label label-info">Note</span> Currently, Socketpot is intended to be used from the Javascript front-end of your app in the context of a single user, but in the future we will extend its scope to anticipate direct server-side access.

![network diagram](/img/post-assets/introducing-socketpot/diagram.png)

## Socketpot Features

- Persistent chat system
  - Unlike Moneypot's previous chat-server, Socketpot saves all chat messages and the mutelist to the database.
  - Optionally implement your own chat rooms by setting the `channel` key on each chat message.
- Deposit notifications
  - Get notified every time the current user's unconfirmed or confirmed balance changes from external actions, like when they withdraw/deposit into your app from Moneypot.
  - Use unconfirmed-balance updates to instantly acknowledge a user's bitcoin-address deposit and notify them when the deposit is confirmed on the blockchain and ready to be spent.
- Bet notifications
  - Subscribe to a real-time stream of bets placed against your app.
  - Useful for creating an "All Bets" tab.
  - <span class="label label-info">Note</span> For apps with heavy bet volume, this is only a stream of the most interesting bets.

## Quick-start Example

Use the [socket.io][socket.io] Javascript library to speak to Socketpot.

Here's some basic client-side boilerplate to get you started:

{% highlight javascript %}
var socket = io('https://socket.moneypot.com');

////////////////////////////////////////////
// Setup
////////////////////////////////////////////

var config = {
    // Required Integer, your Moneypot app ID
    app_id: 1,
    // Optional String, uuid of user's access_token
    access_token: undefined,
    // Required Array<String>,
    // must include one or more of 'CHAT' | 'DEPOSITS'
    subscriptions: ['CHAT', 'DEPOSITS']
};

////////////////////////////////////////////
// General websocket events
////////////////////////////////////////////

socket.on('connect', function() {
    log('[socket] connected');
    var authRequest = {
        app_id: config.app_id,
        access_token: config.access_token,
        subscriptions: config.subscriptions
    };
    socket.emit('auth', authRequest, function(err, authResponse) {
        if (err) {
            log('[auth] Error:', err);
            return;
        }
        log('[auth] Success. Response:', authResponse);
    });
});
socket.on('disconnect', function() {
    log('[socket] disconnected');
});
socket.on('error', function(err) {
    log('[socket] error:', err);
});
socket.on('reconnect_error', function(err) {
    log('[socket] error while reconnecting:', err);
});
socket.on('reconnecting', function() {
    log('[socket] attempting to reconnect...')
});
socket.on('reconnect', function() {
    log('[socket] successfully reconnected');
});

////////////////////////////////////////////
// Socketpot-specific events
////////////////////////////////////////////

socket.on('client_error', function(text) {
    log('[client_error]', text);
});

// If subscribed to 'DEPOSITS'

socket.on('unconfirmed_balance_change', function(payload) {
    log('[unconfirmed_balance_change]', payload);
});
socket.on('balance_change', function(payload) {
    log('[balance_change]', payload);
});

// If subscribed to 'CHAT'

socket.on('new_message', function(payload) {
    log('[new_message]', payload);
});
socket.on('user_joined', function(payload) {
    log('[user_joined]', payload);
});
socket.on('user_left', function(payload) {
    log('[user_left]', payload);
});

// If subscribed to 'BETS'

socket.on('new_bet', function(payload) {
    log('[new_bet]', payload);
});

////////////////////////////////////////////
// Misc
////////////////////////////////////////////

function log() {
    document.write(args.join(' ') + '<br>');
}
{% endhighlight %}

## Socketpot API

The remainder of this post describes the Socketpot API in technical detail.

- Socketpot's URI: `https://socket.moneypot.com`

### Handling Errors

Socketpot echoes back errors in two ways:

1. It emits `client_error` with a developer-friendly message if the problem
has to do with your client's code (i.e. a bug with your code).

        socket.on('client_error', function(text) {
            console.warn('Client Error:', text);
        });

2. It potentially sends a error string back as the first argument to any
callback function that socketpot expects from you. Always be sure to check
for the presence of this error string before moving on in your callback functions. If defined, the `err` argument of a callback will always be a string constant like `"USER_IS_MUTED"` or `"INTERNAL_ERROR"`.

        socket.emit('new_message', { text: 'Hello, world!' }, function(err, msg) {
          if (err) {
            alert('Error submitting message: ' + err);
            return;
          }

          alert('Successfully submitted message');
        })

<span class="label label-info">Tip</span> Always start development by listening for the `client_error` event. It's there to give you helpful debug messages when your client isn't following the rules.

### Authenticating with Socketpot

As soon as your client gets a websocket connection to socketpot, socketpot expects you to emit an `auth` request.

<span class="label label-warning">Warning</span> If you wait longer than 30 seconds to send the auth request, socketpot will emit a client_error and then disconnect you.

Example:

{% highlight javascript %}
socket.on('connect', function() {
  console.log('Connected to socketpot');

  var authRequest = {
    app_id: 42,
    access_token: '82c5bbe7-d9fd-4f5d-a06c-e34e588db2fd',
    subscriptions: ['CHAT', 'DEPOSITS']
  };

  socket.emit('auth', authRequest, function(errString, authResponse) {
    if (err) {
      console.log('Error in auth response:', errString);
      return;
    }
    console.log('Successful auth response:', authResponse);
  });
});
{% endhighlight %}

Spec:

{% highlight javascript %}
socket.emit('auth', AuthRequest, AuthCallback);
{% endhighlight %}

`AuthRequest` is a required object:

    {
      app_id:        Required Integer
      subscriptions: Required Array of Strings
      access_token:  Optional String (UUID)
    }

- `subscriptions` must be an array that includes one or more of "CHAT", "DEPOSITS", "BETS".
- `app_id` is the unique id of your Moneypot app.
- `access_token` is an optional UUID string that represents the user that is currently using your client. If you don't define it or if the access token is invalid/expired/revoked by the user, then the auth response will not have a `user` field.

`AuthCallback` is a required function with signature:

    function(errString, AuthResponse) { ... }

- `errString`: undefined or String
  - Possible values: `"INTERNAL_ERROR"`

`AuthResponse` is an object returned to your `AuthCallback` upon successful auth:

    {
      user: UserObject | undefined,
      chat: ChatObject | undefined,
      subscriptions: Array of Strings
    }

`UserObject` is only present if the access_token in your AuthRequest was valid/active. If AuthResponse has a user attached to it, then the user that's using your client successfully authenticated.

`UserObject` spec:

    {
      uname:   String (User's username),
      auth_id: Integer (Internal Moneypot ID),
      role:    String "OWNER" | "MOD" | "MEMBER"
    }

`ChatObject` is only set if you had `"CHAT"` as one of your subscriptions. This object will give you the latest 100 chat messages and a map of users that are currently connected (keyed by uname):

    {
        messages: [MessageObject],
        userlist: {Uname -> UserObject}
    }

Each MessageObject is a chat message in order from oldest (top) to newest (bottom).

`MessageObject` spec:

    {
        channel:    String,
        text:       String,
        created_at: Date,
        user:       UserObject | undefined
    }

Messages are either created by users ("chat messages") or the system ("system messages"). If a message has a user field, it's a chat message. If user is undefined, then it's a system message.

Example system message text: "Ryan muted Kate for 10 minutes".

It's recommended to display both system messages and chat messages in your chat box.

`AuthResponse` also includes a `subscriptions` field which is an array of strings. It echoes back the subscriptions that you were successfully subscribed to. However, it's important to point out that if you tried to subscribe to `"DEPOSITS"` but your access_token was invalid, then you will not be subscribed to deposits since it only makes sense in the context of a user.

If subscriptions comes back as an empty array, then socketpot will immediately disconnect you.

## Socket events

These are all the events that Socketpot either expects or will emit to your client.

### Always emitted

Socketpot will emit these events no matter what you're subscribed to.

#### [Server->Client] Event: `client_error`

Socketpot emits this event when your client code does something wrong.

If you receive client_error, then consider it a bug that you need to fix with your code. For example, you will receive this error if you try to send a new_message that is longer than Socketpot allows because Socketpot expects your client to enforce its validations.

Example:

{% highlight javascript %}
socket.on('client_error', function(text) {
  console.log('client_error:', text);
});
{% endhighlight %}

Client errors are always emitted with human-/developer-friendly messages to explain what you did wrong. Ideally you should only receive these while developing your client and it will steer you in the right direction.

### When subscribed to "CHAT"

Subscribe to "CHAT" if your app has a chatbox.

#### [Server->Client] Event: `user_joined`

Socketpot emits this event to all clients in the chatroom whenever an
authenticated user connects to the server.

Payload is an object.

    {
      "uname": String (username),
      "auth_id": Integer,
      "role": String "OWNER" | "MOD" | "MEMBER"
    }

Example payload:

    {
      "uname": "foo_foo",
      "auth_id": 2,
      "role": "MEMBER"
    }

#### [Server->Client] Event: `user_left`

Socketpot emits this event to all clients in the chatroom whenever a user's
final client disconnects from the server.

Payload is an object `{ "uname": String }`:

    socket.on('user_left', function(payload) {
      console.log(payload.uname + ' left the chat');
    });

#### [Client->Server] Event: `new_message`

Emit this event to socketpot when the user types a message into chat.

Spec:

    socket.emit('new_message, PayloadSpec, callback);

PayloadSpec is an object:

    {
      channel: undefined | String,
      text: String
    }

- `channel` defaults to "lobby".
    - Validation:
        - Cannot be more than 40 chars long.
        - Must only contain chars: a-z, 0-9, '_', '-', ':'
    - The purpose of channel is simply to allow you to sort messages on the client side.
    - For example, you could implement a Support tab for your app's chatroom by setting
channel to "support" if the user submits a message while that tab is selected.
All clients in your chatroom will receive the message and can display it in the
Support tab.
    - Just leave it blank if you have no need for it.

- `text` is the message that the user is sending to the server.
    - Validation: Must be 1-300 chars long after being `text.trim()`'ed.
- `callback` is a function with signature `fn(errString, message)`
    - If there was an error, only `errString` will be set.
    - Possible errString values:
        - `"USER_IS_MUTED"`: User's message was rejected because they are muted
        - `"INTERNAL_ERROR"`
        - `"FORBIDDEN"`: User tried sending a previleged command like `/mute ...`
        (Only OWNERS and MODS can use commands so far)
        - `"INVALID_COMMAND"`: Malformed `/mute` or `/unmute` command

Example:

{% highlight javascript %}
socket.emit('new_message', {
  text: '/mute foobar 10 mins'
}, function(err, msg) {
  if (err) {
    console.log('Error when submitting new_message to server:', err);
    return;
  }
  console.log('Successfully submitted message:', msg);
});
{% endhighlight %}

#### [Server->Client] Event: `new_message`

Socketpot emits this event whenever your chatroom received a new message.

A message is either a user-message or a system-message. A user-message is
submitted by users chatting with each other. A system-message broadcasts
events like "Ryan muted Sarah for 8 minutes".

If `payload.user` exists, it's a chat-message. Else, (if undefined) it's
a system-message.

The payload is an object.

Chat-message:

    {
      "created_at": Date,
      "text": String,
      "user": {
        "uname": String,
        "role": String ('OWNER' | 'MOD' | 'MEMBER')
      }
    }

Example chat-message:

    {
      "created_at": Date("2015-08-05T01:09:08.598Z"),
      "text": "Hello, world!",
      "user": {
        "uname": "Ryan",
        "role": "OWNER"
      }
    }

System-message:

    {
      "created_at": Date,
      "text": String
    }

Example system-message:

    {
      "created_at": Date("2015-08-05T01:09:08.598Z"),
      "text": "Ryan muted Sarah for 8 minutes"
    }


### When subscribed to "DEPOSITS"

Subscribe to "DEPOSITS" to receive push notifications whenever changes are
detected in the unconfirmed and confirmed balances of the user for
this app.

These notifications allow you to provide instant feedback whenever the
user deposits bitcoin into your app via the blockchain.

Reminder: Moneypot waits for one blockchain confirmation before it
lets a user spend bitcoin.

Within seconds of the user making a deposit to a deposit-address for your app,
you will receive an `unconfirmed_balance_change` event so that you can
immediately reassure the user that their bitcoin was successfully received but
awaiting confirmation before it can be spent.

When a user's deposit makes it into a block on the blockchain, socketpot
will send a `unconfirmed_balance_change` event and a (confirmed) `balance_change`
event that enable you to update your UI appropriately without requiring
the user to refresh the page.

#### [Server->Client] Event: `unconfirmed_balance_change`

Emitted whenever a change is detected in user's balance that
is awaiting confirmation.

The payload is an object:

    {
      balance: Integer, unconfirmed balance (satoshis) as of now
      diff:    Integer (satoshis)
    }

- `balance` is an unsigned integer representing the user's current unconfirmed balance amount. May be zero.
- `diff` is a signed integer representing the change delta.

#### [Server->Client] Event: `confirmed_deposit_change`

Emitted whenever a change is detected in user's balance that has one confirmation.

The payload is an object:

    {
      balance: Integer (satoshis),
      diff: Integer (satoshis)
    }

- `balance` is the updated confirmed balance for the user.
- `diff` is a signed integer representing the change delta.

### When subscribed to "BETS"

#### [Server->Client] Event: `new_bet`

Emitted when new bets are made from your app. Apps with high bet volume will only receive a sample of the most interesting bets.

The payload is an object:

    {
      id: Integer,
      TODO: Finish
    }

- `id` is the bet's unique ID. You can turn this into the post's permalink via `https://www.moneypot.com/bets/:id`.





[socket.io]: http://socket.io/
