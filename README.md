# blink-cmp-dbee

A [blink.cmp](https://github.com/saghen/blink.cmp) completion source for
[nvim-dbee](https://github.com/kndndrj/nvim-dbee).

Provides completions for:

- **SQL keywords** – always available in SQL-like filetypes
- **Tables** and **views** from the currently active dbee connection
- **Columns** discovered via the connection's schema structure
- **Schemas** / namespaces

---

## Requirements

| Plugin | Version |
|--------|---------|
| [saghen/blink.cmp](https://github.com/saghen/blink.cmp) | v1.x (stable) |
| [kndndrj/nvim-dbee](https://github.com/kndndrj/nvim-dbee) | latest master |
| Neovim | 0.10+ |

---

## Installation

### lazy.nvim

```lua
{
  "your-username/blink-cmp-dbee",   -- or a local path
  dependencies = {
    "saghen/blink.cmp",
    "kndndrj/nvim-dbee",
  },
},
```

Then wire it into blink.cmp:

```lua
{
  "saghen/blink.cmp",
  opts = {
    sources = {
      default = { "lsp", "path", "snippets", "buffer", "dbee" },
      providers = {
        dbee = {
          name   = "Dbee",
          module = "blink-cmp-dbee",
          opts   = {
            -- Limit to SQL-like filetypes (omit to enable everywhere)
            filetypes = { "sql", "mysql", "plsql" },
            -- Seconds before the schema cache is refreshed (default: 30)
            cache_ttl = 30,
          },
        },
      },
    },
  },
},
```

### Local / manual install

Copy `lua/blink-cmp-dbee/init.lua` to your Neovim runtime path, for example:

```
~/.config/nvim/lua/blink-cmp-dbee/init.lua
```

---

## How it works

1. **SQL keywords** are hardcoded and always returned instantly.
2. When a completion is triggered, the source calls
   `require("dbee.api").core.get_current_connection()` to find the active
   dbee connection.
3. It then calls `api.core.connection_get_structure(conn.id)` to walk the
   schema tree (schemas → tables/views → columns) returned by the Go backend.
4. Results are cached per-connection for `cache_ttl` seconds so the Go backend
   is not queried on every keystroke.
5. The `resolve()` hook adds a short Markdown documentation string shown in
   blink.cmp's documentation window.

---

## Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `filetypes` | `string[]` or `nil` | `nil` (all) | Restrict completions to these filetypes |
| `cache_ttl` | `number` | `30` | Schema cache lifetime in seconds |

---

## Limitations & tips

- **dbee must be set up first.** If `require("dbee.api")` fails (e.g. dbee
  is not installed), only SQL keywords are returned – no errors are raised.
- **Active connection only.** Completions reflect whichever connection is
  currently active in dbee's drawer. Switch connections with `<CR>` on a
  connection node to update suggestions.
- **Schema cache.** Reduce `cache_ttl` if you alter the schema frequently
  during a session. You can also clear the cache manually:
  ```lua
  require("blink-cmp-dbee")._cache = {}
  ```
- **Column context.** The source does not yet parse the SQL in the buffer to
  filter columns by table; all discovered columns are returned and fuzzy
  matching in blink.cmp narrows them down.

---

## Contributing

Pull requests are welcome. The whole source lives in one file:
`lua/blink-cmp-dbee/init.lua`.
