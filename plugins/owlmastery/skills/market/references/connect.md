# Connect Owl Mastery

Owl Mastery is for registered users, so the user needs an account on owlmastery.com to sign in. The client opens the Owl Mastery sign-in page in the browser; the user signs in and selects Allow. The connection reads market data and the user's watchlists, and cannot change anything. It stays connected until the user disconnects it under Profile, Connected apps on owlmastery.com. The page https://owlmastery.com/connect has the same steps.

## Steps by client

- **Claude.ai or the Claude desktop app.** Open Customize, then Plugins, then Owlmastery, then the Connectors tab, and select Connect. Sign in on owlmastery.com and select Allow.
- **Claude.ai in a Team or Enterprise organization.** If Connect is greyed out with "Contact an organization owner", an owner of the organization adds a custom connector with the URL `https://owlmastery.com/mcp` under Organization settings, then Connectors. The user then selects Connect.
- **Claude Code.** Run `/mcp`, pick owlmastery, and select Authenticate. If owlmastery is not listed, run `/reload-plugins` first.
- **Codex.** Run `codex mcp login owlmastery`.
- **Another MCP client.** Add the server URL `https://owlmastery.com/mcp` and sign in when the client asks.

## After a 401

A 401 means the client holds no valid sign-in: the user disconnected it in Profile, or it never signed in. Connecting again with the steps above fixes it. A client that cannot sign in in a browser uses an API key, which the user creates under Profile, API keys, and which stops working on the date it expires.
