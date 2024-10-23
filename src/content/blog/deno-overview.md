---
author: Ernesto Osuna
pubDatetime: 2024-10-23T03:26:02.527Z
title: Deno v Bun, an overview
slug: deno-v-bun
featured: true
ogImage: https://media2.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft724xom09eaifrx73urq.png
tags:
  - release deno bun first
description: Deno overview comparing to bun, basic features.
---

This post in inspired, by [this thread](https://x.com/jamonholmgren/status/1837911158344634741) by [Jamon](https://x.com/jamonholmgren). 

I this thread i'm gonna try to explain, my first real experience doing a project in Deno after probably a year of wanting to do something. The first feature that spoke to me, was the no building step from typescript to javascript, that can simplify things and even now node [wants to add that too](https://nodejs.org/en/blog/release/v22.6.0).

but i digress, in this post i will try to go over the points of Jamon in his thread thats talking about [Bun](https://bun.sh/), and how that is achieved in [Deno](https://deno.com/).

## Deno as a runtime and package manager.
In Deno as in Bun you can run your project with
```sh 
Deno run ./file.ts 
```
but i would suggest to add these commands in your deno.json file, if you will have a project
```json
{
  "tasks": {
    "dev": "deno run ./main.ts"
  }
}
```
Running Deno run will try to install all dependencies that you have on your deno.json file, and in your main.ts and it's imports. For the package management part you will have 2 preferred options [Jsr](https://jsr.io/) (preferred), and deno.land, you should always try to install dependencies from jsr, but you can also install from npm like
```typescript
import * as emoji from "npm:node-emoji";
```
then running your program will install `node-emoji` dependency first, then execute the rest of your program, you can also try `deno add npm:express` and this will add your dependency to the deno.json file (like the package.json)

Next lets try a basic server implementation with the standard library in Deno.
```typescript
const port = 8082;
const handler = async (
    req: Request,
): Promise<Response> => {
    const url = new URL(req.url);
    let responseStatus = 404;
    let body = "default response";
    if (url.pathname == "/main") {
        body = "Reponse from main";
        responseStatus = 200;
    }
    return new Response(body, { status: responseStatus });
};

console.log(`HTTP server running. Access it at: http://localhost:${port}/`);
Deno.serve({ port }, handler);
```
by the way Deno comes with it's linter and formatter out of the box, so that's appreciated. 

For the websocket in Deno is a little bit different.
`const { socket, response } = Deno.upgradeWebSocket(req);`
you will need this line to upgrade the http request to websocket, and then return the response `return response`from said result. the example would be like
```typescript
    if (req.headers.get("upgrade") == "websocket") {
        const { socket, response } = Deno.upgradeWebSocket(req);
        socket.addEventListener("open", () => {
            console.log("a client connected!");
        });

        socket.addEventListener("message", (event) => {
            if (event.data === "ping") {
                socket.send("pong");
            }
        });

        return response;
    }
// you can add this inside the handler function.
```
and the last element i will touch on this are crons, Jamon explains he can do it with a setInterval (and it can be done) `setInterval(() => processTask(), 60 * 60 * 1000)`.

but Deno helps you with an specific [interface for it](https://docs.deno.com/api/deno/~/Deno.cron), and the usage is pretty simple
```typescript 
Deno.cron("Log a message", "* * * * *", () => {
    console.log("This will print once a minute.");
});
```
be sure to run your program with the `--unstable-cron` flag, as the feature is not yet stable. 

![Image of a console, showing This will print once a minute. 3 times in a row](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o1pedo3cmqeb6r4uhuph.png)
i ran it for about 3 minutes, and this is the result.

And with that i explain all the points that Jamon explain for Bun, there are more tweets but those are generalities of javascript or how to deploy or use other non standard tools. not about the Bun runtime.

if you want to learn Deno you can use [Deno/examples](https://docs.deno.com/examples/) to start your journey.

in my limited experience, working with Deno has been a great experience to code with.

Thanks to [Jamon](https://x.com/jamonholmgren) for the inspiration to make this short post.