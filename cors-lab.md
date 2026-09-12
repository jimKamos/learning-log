# cors-lab — a server that refuses, then allows

Built in S06 to trigger a real CORS error and fix it server-side.
Code lives at `C:\Users\absol\Documents\projects\cors-lab\server.js`, not in
git — this file is the record. The bare server was scaffolding; the four
response headers are mine.

## Run

```
node server.js      # terminal 1, at cors-lab
npm run dev         # terminal 2, at vite-project → localhost:5173
```

Node does **not** reload on save. `Ctrl+C` and restart after every edit.

Test from the `localhost:5173` console:

```js
fetch('http://localhost:3000', { credentials: 'include' })
  .then(r => r.json())
  .then(d => console.log('ok →', d))
  .catch(e => console.log('failed →', e.message));
```

## server.js

```js
const http = require('node:http');

const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);

  res.writeHead(200, {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': 'http://localhost:5173',
    'Access-Control-Allow-Credentials': 'true',
    'Vary': 'Origin'
  });
  res.end(JSON.stringify({ id: 1, name: 'Mock order', total: 42 }));
});

server.listen(3000, () => {
  console.log('API listening on http://localhost:3000');
});
```

## The four states

| Headers sent | Request | Result |
|---|---|---|
| none | plain `GET` | blocked — no `Allow-Origin` present |
| `Allow-Origin: *` | plain `GET` | works |
| `Allow-Origin: *` | `credentials: 'include'` | blocked — wildcard illegal with credentials |
| named origin + `Allow-Credentials: true` | `credentials: 'include'` | works |

| Header | Job |
|---|---|
| `Access-Control-Allow-Origin` | who may read the response |
| `Access-Control-Allow-Credentials` | may they read it *as the logged-in user* |
| `Vary: Origin` | the response depends on the `Origin` **request header** — a cache-key instruction, not a value |

## The point

Browser said `ERR_FAILED`, `catch` said `Failed to fetch` — and at that exact
moment the server logged `GET /` and sent a `200` with the full body. Nothing
was blocked at the server. The response arrived intact and the browser refused
to hand it to my JavaScript.

**The policy blocks the read, not the send.** If that had been
`POST /delete-account`, the account would be gone and the `catch` would still
say `Failed to fetch`. When the frontend swears the API is down, check the
server log first.

Origin = scheme + host + port, no trailing slash: `http://localhost:5173`.