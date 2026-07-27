# Connect Claude Code to SAP (Beginner Guide)

This guide walks you through connecting Claude Code to a local SAP system,
step by step. No prior experience needed — just follow along in order.

**What you'll end up with:** Claude Code will be able to read/create ABAP
objects, check transports, and run tests directly against your SAP system.

## Before you start, make sure you have

- [ ] VS Code with the **ABAP Development Tools** extension installed
- [ ] Claude Code installed in VS Code and signed in
- [ ] Your SAP system details handy, for example:

  | Field | Example value |
  |---|---|
  | Application server | `vhcala4hci` |
  | System ID | `A4H` |
  | Instance | `00` |
  | Client | `001` |

  | Field | Value |
  |---|---|
  | Application server | ___________ |
  | System ID | ___________ |
  | Instance | ___________ |
  | Client | ___________ |

---

## Step 1: Turn on the ADT MCP Server in VS Code

This starts a small local service that Claude Code will talk to.

1. In VS Code, open **Settings** (gear icon, bottom-left → Settings).
2. In the search box, type `Adt Mcp Server`.
3. Tick the checkbox next to **Adt › Mcp Server: Enabled**.
4. Two new fields will appear — leave them as they are and just write down:
   - **Port** (a number, e.g. `2236`)
   - **Token** (a long random string) — click into the box and copy it

Keep this browser tab/window open — you'll need the Port and Token in Step 2.

> 🔒 The Token is like a password. Don't share it or paste it anywhere public.
> It changes every time you switch the checkbox off and on again.

---

## Step 2: Tell Claude Code how to reach it

Claude Code keeps a settings file with a list of tools it's allowed to use.
We're going to add one entry to it by hand.

1. Open this file in a text editor (VS Code works fine):
   - **Windows:** `C:\Users\<your-username>\.claude.json`
   - **Mac:** `~/.claude.json`
2. **Make a backup copy first** — copy the file and rename the copy to
   `.claude.json.backup`. If a step below goes wrong, you can restore it.
3. Inside the file, look for a section called `"mcpServers"`. It sits at the
   very top level of the file (a sibling to other settings like `"userID"`,
   `"projects"`, etc.) — **not** nested inside any of those.

   - **If you see `"mcpServers": { ... }` already** — add your `"adt-mcp"`
     entry inside its `{ }`, like this:

     ```json
     "mcpServers": {
       "adt-mcp": {
         "type": "http",
         "url": "http://localhost:2236/mcp",
         "headers": { "Authorization": "Bearer PASTE_YOUR_TOKEN_HERE" }
       }
     }
     ```

   - **If `"mcpServers"` doesn't exist anywhere in the file** — add it as a
     brand new top-level entry. Right after the file's very first `{`, paste
     the whole block below, followed by a comma so it connects to whatever
     key comes next:

     ```json
     {
       "mcpServers": {
         "adt-mcp": {
           "type": "http",
           "url": "http://localhost:2236/mcp",
           "headers": { "Authorization": "Bearer PASTE_YOUR_TOKEN_HERE" }
         }
       },
       "userID": "...",
       ... (rest of the file stays the same)
     }
     ```

4. Either way, make sure you replaced the port and token with **your own
   values** from Step 1, not the example ones shown here.
5. Double-check every `{`, `}`, `,` and `"` matches up — a single missing
   comma will stop Claude Code from starting. Save the file.

---

## Step 3: Check it worked

1. Open (or restart) Claude Code.
2. Type `/mcp` and press Enter.
3. You should see `adt-mcp` listed with a green ✔ **Connected** status.

If it says "not connected," double check:
- Is the ADT MCP Server checkbox (Step 1) still turned on?
- Does the port in your JSON match the port shown in VS Code settings?
- Did you copy the full token with no extra spaces?

---

## Step 4: Connect it to your actual SAP system

So far Claude Code can reach the *local service* — now we point that service
at your SAP system.

1. In VS Code, open the Command Palette:
   - Windows/Linux: `Ctrl+Shift+P`
   - Mac: `Cmd+Shift+P`
2. Type `ABAP: New Destination` and select it.
3. Choose **RFC** (this is for a regular company/on-premise SAP system).
4. Fill in the form using your system details from the checklist at the top:
   - System ID
   - Application server
   - Instance number
   - Client
   - Your SAP username, password, and logon language
5. Finish the wizard — this saves the connection in VS Code.

---

## Step 5: Try it out

In Claude Code, just ask in plain English:

```
Can you list the ABAP destinations available?
```

If everything is set up correctly, it will show your system, something like:
```
[A4H_001_YOURUSER_EN]
```

You're now connected! Try asking things like:
- "List the creatable ABAP object types"
- "Summarize the changes in transport request XXXXXXXXXX"
- "Run ABAP unit tests for class ZCL_MY_CLASS"

---

## When your token stops working

The Token from Step 1 changes if you ever turn the ADT MCP Server checkbox
off and on. When that happens:

1. Go back to VS Code settings and copy the **new** Token.
2. Open your `.claude.json` file again.
3. Replace the old token (the part after `Bearer `) with the new one.
4. Save, then run `/mcp` in Claude Code again to reconnect.

---

## Quick troubleshooting

| Problem | Likely fix |
|---|---|
| `adt-mcp` not connected in `/mcp` | Recheck the ADT MCP Server checkbox and port in VS Code settings |
| Claude Code won't start / JSON error | You likely have a typo in `.claude.json` — restore your backup and redo Step 2 carefully |
| Asking for destinations returns nothing (`[]`) | You haven't done Step 4 yet — no SAP connection exists in VS Code |
| Token error / "unauthorized" | Your token changed — see "When your token stops working" above |
