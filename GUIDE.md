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
```
##
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
**Next.js** *(in next.config)* 
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
