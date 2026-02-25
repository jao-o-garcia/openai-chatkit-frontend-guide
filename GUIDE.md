# OpenAI Chatkit Guide for Integrating Your Web frontend

There are two setup examples from openai that we can use as a guide. All options can be in production, but how much control you want would depend on the category that you will choose.

## Basic:
Considering you just want to use Agent Builder and plug it in 


## Advanced:
Considering that you have now setup your backend server, both can be setup into

Small scale application (one page)
1. Vite + React (https://github.com/openai/openai-chatkit-advanced-samples)
3. Next.js (https://github.com/openai/openai-cs-agents-demo)


### How does the front end connect to the backend?
Obviously the Frontend needs to connect to the Backend in order for the hole setup to work. 
Recall that running a server needs to be in a port. *Locally*, usually for backend, we run them on port 8000. So you must find a way to connect the front end to port 8000.
You can find how your frontend connects to the backend by checking your config settings


**Vite + React** *(in vite.config)* 
```ts
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5170,
    host: "0.0.0.0",
    proxy: {
      "/chatkit": {
        target: backendTarget,
        changeOrigin: true,
      },
      "/cats": {
        target: backendTarget,
        changeOrigin: true,
      },
    },
    // For production deployments, you need to add your public domains to this list
    allowedHosts: [
      // You can remove these examples added just to demonstrate how to configure the allowlist
      ".ngrok.io",
      ".trycloudflare.com",
    ],
  },
});
```

**Next.js** *(in next.config)* 
```mjs
const nextConfig = {
  devIndicators: false,
  // Proxy /chat requests to the backend server
  async rewrites() {
    return [
      {
        source: "/chat",
        destination: "http://127.0.0.1:8000/chat",
      },
      {
        source: "/chatkit",
        destination: "http://127.0.0.1:8000/chatkit",
      },
      {
        source: "/chatkit/:path*",
        destination: "http://127.0.0.1:8000/chatkit/:path*",
      },
    ];
  },
};
```

### How does the flow work?
Recall that in order for a browser to be able to read your website, it must be in html.

**Vite + React** *(in vite.config)* 

```
my-app/
├── src/
│   ├── main.tsx           ← entry point
│   ├── App.tsx            ← root component
│   ├── pages/             ← manual, not auto-routed
│   ├── components/
│   └── assets/
├── public/
├── index.html             ← actual HTML entry (unique to Vite)
├── vite.config.ts
└── package.json
```
You can start by checking *index.html*. Youl'll be able to get a hint which file it calls. You can see here that the file it calls is /src/main.tsx

Notice also it loads up the Chatkit Web Component *https://cdn.platform.openai.com/deployments/chatkit/chatkit.js*
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <link rel="icon" type="image/png" href="/favicon.png" />
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Cozy Cat Lounge</title>
    <!-- Load the ChatKit Web Component -->
    <script src="https://cdn.platform.openai.com/deployments/chatkit/chatkit.js"></script>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```
**Next.js** *(in next.config)* 
In Next.js, it doesn't have an index.HTML. It starts in layout.tsx (which is found in the folder called *app*)
```
my-app/
├── app/
│   ├── layout.tsx         ← root layout
│   ├── page.tsx           ← home route (/)
│   ├── about/
│   │   └── page.tsx       ← /about route
│   └── api/
│       └── hello/route.ts ← API endpoint
├── public/
├── components/
├── next.config.js
└── package.json
```
Here, the layout.tsx does not say which files it is not clear which files it loads first.
```
import type React from "react";
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import Script from "next/script";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "Airlines Agent Orchestration",
  description: "An interface for airline agent orchestration",
  icons: {
    icon: "/openai_logo.svg",
  },
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Script
          src="https://cdn.platform.openai.com/deployments/chatkit/chatkit.js"
          strategy="beforeInteractive"
        />
        {children}
      </body>
    </html>
  );
}
```
... The reason is because Next.js does automatic routing. How it routes is that based on its `children`. 

*(What we did before is we routed from one file to another)*

So how does one become a child? Simple - You just have to be a folder under the *app* folder. 

So if you want to access a different page, all you have to do is create a folder and then place a `page` file *(can be jsx/tsx/js/etc)*

```
app/about/page.tsx  →  accessible at /about
app/contact/page.tsx → accessible at /contact
```
In the example, notice that there are multiple pages.tsx 
```
my-app/
├── app/
│   ├── layout.tsx         ← root layout
│   ├── page.tsx           ← home route (/)
│   ├── about/
│   │   └── page.tsx       ← /about route
│   └── api/
│       └── hello/route.ts ← API endpoint
├── public/
├── components/
├── next.config.js
└── package.json
```
The parent page `page.tsx` under the `app/` folder.
If you open this file you will notice that it injects multiple pages or components inside of it.

*(Note: Syntax for injecting is <SomeComponent/> (this format alone already is calling the component)*
```
return (
    <main className="flex h-screen gap-2 bg-gray-100 p-2">
      <AgentPanel
        agents={agents}
        currentAgent={currentAgent}
        events={events}
        guardrails={guardrails}
        context={context}
      />
      <ChatKitPanel
        initialThreadId={initialThreadId}
        onThreadChange={handleThreadChange}
        onResponseEnd={handleResponseEnd}
        onRunnerBindThread={handleBindThread}
      />
    </main>
  );
```

The `page` file is a reserved word for Next.js. If you name it differently (i.e. HomePage) Next.js won't treat it as a page.
