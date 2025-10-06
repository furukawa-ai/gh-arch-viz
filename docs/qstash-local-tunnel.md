# Local Tunnel

QStash requires a publicly available API to send messages to.
The recommended approach is to run a [development server](/qstash/howto/local-development) locally and use it for development purposes.

Alternatively, you can set up a local tunnel to expose your API, enabling QStash to send requests directly to your application during development.

## localtunnel.me

[localtunnel.me](https://github.com/localtunnel/localtunnel) is a free service to provide
a public endpoint for your local development.

It's as simple as running

```
npx localtunnel --port 3000
```

replacing `3000` with the port your application is running on.

This will give you a public URL like `https://good-months-leave.loca.lt` which can be used
as your QStash URL.

If you run into issues, you may need to set the `Upstash-Forward-bypass-tunnel-reminder` header to
any value to bypass the reminder message.

## ngrok

[ngrok](https://ngrok.com) is a free service, that provides you with a public
endpoint and forwards all traffic to your localhost.

### Sign up

Create a new account on
[dashboard.ngrok.com/signup](https://dashboard.ngrok.com/signup) and follow the
[instructions](https://dashboard.ngrok.com/get-started/setup) to download the
ngrok CLI and connect your account:

```bash
ngrok config add-authtoken XXX
```

### Start the tunnel

Choose the port where your application is running. Here I'm forwarding to port
3000, because Next.js is using it.

```bash
$ ngrok http 3000



Session Status                online
Account                       Andreas Thomas (Plan: Free)
Version                       3.1.0
Region                        Europe (eu)
Latency                       -
Web Interface                 http://127.0.0.1:4040
Forwarding                    https://e02f-2a02-810d-af40-5284-b139-58cc-89df-b740.eu.ngrok.io -> http://localhost:3000

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

### Publish a message

Now copy the `Forwarding` url and use it as destination in QStash. Make sure to
add the path of your API at the end. (`/api/webhooks` in this case)

```
curl -XPOST \
    -H 'Authorization: Bearer XXX' \
    -H "Content-type: application/json" \
    -d '{ "hello": "world" }' \
    'https://qstash.upstash.io/v2/publish/https://e02f-2a02-810d-af40-5284-b139-58cc-89df-b740.eu.ngrok.io/api/webhooks'
```

### Debug

In case messages are not delivered or something else doesn't work as expected,
you can go to [http://127.0.0.1:4040](http://127.0.0.1:4040) to see what ngrok
is doing.
