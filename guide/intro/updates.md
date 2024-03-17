# Getting updates

## What is an Update?

Updates are events that happen in Telegram and sent
to the clients. This includes events about a new message, joined chat member,
inline keyboard callbacks, etc.

In a bot environment, a [Dispatcher](/guide/dispatcher/intro.html) is normally used to handle the updates.

### Don't need them?

Sometimes, you might not really need any updates.
Then, just disable them in `TelegramClient` parameters:

```ts
const tg = new TelegramClient({
  // ...
  disableUpdates: true
})
```

## Setting up

The parameters themselves will be explained a bit below, for now let's just focus on how they are passed.

`TelegramClient` has updates handling turned on by default, and you can configure it using `updates` parameter

```ts
const tg = new TelegramClient({
  // ...
  updates: {
    messageGroupingInterval: 250,
    catchUp: true
  }
})
```

The updates themselves are dispatched on the client as events (see [reference](https://ref.mtcute.dev/classes/_mtcute_core.index.TelegramClient.html#on)):
```ts
tg.on('new_message', (msg) => {
  console.log(msg.text)
})

// You can also handle any supported update at once:
tg.on('update', (upd) => {
  if (upd.name === 'new_message') {
    console.log(upd.data.text)
  }
})

// As well as raw MTProto updates:
tg.on('raw_update', (upd, users, chats) => {
  console.log(upd._)
})
```

::: tip
Client events are based on EventEmitter. It expects handlers to be synchronous,
so if you want to do something async, make sure to also handle the errors:

```ts
tg.on('new_message', async (msg) => {
  try {
    await msg.answerText('test')
  } catch (e) {
    console.error(e)
  }
})
```
:::

### Missed updates

When your client is offline, updates are still stored by Telegram,
and can be fetched later (client "catches up" with the updates).

When back online, mtcute may "catch up", fetch any missed updates and
process them. To do that, pass `catchUp: true` parameter as shown above:

### Message grouping

As you may already know, albums handling in Telegram is not very trivial, as they are sent by the server
as separate messages. To make it easier to handle them, you may opt into grouping them automatically.

To do that, pass `messageGroupingInterval` as shown above. It is a number of milliseconds to wait
for the next message in the album. If the next message is not received in that time, the album is
considered complete and dispatched as a single `message_group` object.

The recommended value is `250` ms.

::: warning
This **will** introduce delays of up to `messageGroupingInterval` ms for every message with media groups,
and may sometimes break ordering. Use with caution.
:::

## Opening chats

For Telegram to properly send updates for channels (e.g. for channels that you are not a member of, 
and for more consistent updates for channels that you are a member of), you need to open them first. 

This is done by calling `openChat` method:

```ts
await tg.openChat('durov')
```

Once you're done, you can close the chat by calling `closeChat`:

```ts
await tg.closeChat('durov')
```

## Dispatcher

Dispatcher is a class that dispatches client events to registered handlers,
while also filtering and propagating them as needed.

You can think of it as some sort of framework that allows you to do everything
more declaratively and handles most of the boilerplate related to state.

Dispatcher is provided by `@mtcute/dispatcher` package.

### Registering

Dispatcher is quite a powerful thing, and we will explore it in-depth in
a separate section. For now, let's just register a dispatcher and add a simple handler:

```ts
const tg = new TelegramClient(...)
const dp = new Dispatcher(tg)

dp.onNewMessage(async (msg) => {
  await msg.forwardTo('me')
})

tg.run()
```

Pretty simple, right? We have registered a "new message" handler and made
it forward any new messages to "Saved Messages".

### Filters

Example above is pretty useless in real world, though. Most of the
time you will want to *filter* the events, and only react to some of them.

Let's make our code only handle messages containing media:

```ts
dp.onNewMessage(
  filters.media,
  async (msg) => {
    await msg.forwardTo('me')
  }
)
```

Filters can do a lot more than that, and
we will cover them further [later](../dispatcher/filters.html).
