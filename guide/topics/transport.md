# Transport

Transport is a way for mtcute to communicate with Telegram servers.

mtcute comes bundled with TCP and WebSocket transport, and also
supports proxies via additional packages.

## TCP transport

TCP transport is the default transport for NodeJS, and is implemented
using `net.Socket` in `TcpTransport`:

```ts{5}
import { BaseTelegramClient, TcpTransport } from '@mtcute/core'

const tg = new BaseTelegramClient({
    // ...
    transport: () => new TcpTransport()
})
```

::: tip
In NodeJS, it is used automatically, you don't need to pass this explicitly
:::

## WebSocket transport

WebSocket transport is mostly used for the browser,
but can also be used in NodeJS.

It is implemented in `WebSocketTransport`:

```ts{5}
import { BaseTelegramClient, WebSocketTransport } from '@mtcute/core'

const tg = new BaseTelegramClient({
    // ...
    transport: () => new WebSocketTransport()
})
```

::: tip
In browser, it is used automatically, you don't need to pass this explicitly
:::

## HTTP(s) Proxy transport

To access Telegram via HTTP(s) proxy, you can use
`HttpProxyTcpTransport`, which is provided
by `@mtcute/http-proxy`:

```bash
pnpm add @mtcute/http-proxy
```

```ts{5-8}
import { HttpProxyTcpTransport } from '@mtcute/http-proxy'

const tg = new BaseTelegramClient({
    // ...
    transport: () => new HttpProxyTcpTransport({
        host: '127.0.0.1',
        port: 8080
    })
})
```

## SOCKS4/5 Proxy transport

To access Telegram via SOCKS4/5 proxy, you can use
`SocksTcpTransport`, which is provided
by `@mtcute/socks-proxy`:

```bash
pnpm add @mtcute/socks-proxy
```

```ts{5-8}
import { SocksTcpTransport } from '@mtcute/socks-proxy'

const tg = new BaseTelegramClient({
    // ...
    transport: () => new SocksTcpTransport({
        host: '127.0.0.1',
        port: 8080
    })
})
```

## MTProxy transport

To access Telegram via MTProxy (MTProto proxy), you can use
`MtProxyTcpTransport`, which is provided by `@mtcute/mtproxy`:

```bash
pnpm add @mtcute/mtproxy
```

```ts{5-8}
import { MtProxyTcpTransport } from '@mtcute/mtproxy'

const tg = new BaseTelegramClient({
    // ...
    transport: () => new MtProxyTcpTransport({
        host: '127.0.0.1',
        port: 8080,
        secret: '0123456789abcdef0123456789abcdef'
    })
})
```

::: tip
mtcute supports all kinds of MTProxies, including the newer ones
with Fake TLS ⚡️
:::

## Changing transport at runtime

It is possible to change transport at runtime. For example, this
could be used to change proxy used to connect to Telegram.

To change the transport, simply call `changeTransport`:

```ts
tg.changeTransport(() => new MtProxyTcpTransport({...}))
```

## Implementing custom transport

When targeting an environment which is not supported already,
you can implement a custom transport on your own. In fact, it is
much simpler than it sounds!

You can check out source code for the bundled transports
to get the basic idea
[here](https://github.com/mtcute/mtcute/tree/master/packages/core/src/network/transports),
and re-use any packet codecs that are included.

[comment]: <> (TOOD: detailed docs on how this works)
