# dsh-museav-assets

A **Museav** workspace for DeepSeek Harness: browse featured inspiration and available models, generate images, and manage project assets and works in one place.

Open **Museav** from the global left sidebar. The previous session-specific Assets tab is no longer registered. Existing projects and generation records remain available through the local **museav CLI**. Featured inspiration comes from MUSE AV's public API; the platform automatically selects the model for each generation.

## Platform images

Choose Xiaohongshu, X (Twitter), or Xianyu in the Studio. Enter one brief, select the images you want, and generate them together.

| Platform | Default images |
| --- | --- |
| Xiaohongshu | 3:4 post cover, content image, and summary page |
| X (Twitter) | 16:9 landscape hero and 1:1 square companion |
| Xianyu | 1:1 listing photo, product details, and condition image |

For Xianyu, select a real product photo from the project library first. The Studio shows each image's result and lets you retry failed images. Freeform generation and personal templates are still available. The workspace follows DSH's light or dark theme.

## Install in DSH Web

```sh
dsh plugin --profile web add @dsh-plugins/dsh-museav-assets
dsh --profile web
```

Restart any running Web server to load the plugin, then open **Museav** from the sidebar. On the machine running DSH, sign in to the local `museav` CLI with `museav login` to generate images and use project assets.

## Endpoints

| Route | What it does |
|---|---|
| `GET /api/dsh-museav-assets/health` | CLI presence + version + workspace cap |
| `GET /api/dsh-museav-assets/projects` | Workspaces with gen/asset counts + `used/limit/full` |
| `POST /api/dsh-museav-assets/projects` | `{name}` create a workspace (≤20 chars) |
| `GET /api/dsh-museav-assets/assets?project=<id\|name>` | Assets in that workspace |
| `GET /api/dsh-museav-assets/jobs?limit=<n>` | Recent generation records |
| `POST /api/dsh-museav-assets/gen` | Generate one image and archive it to the selected project |

Failure semantics: any upstream/CLI error returns **200 + `{ok:false,error}`** so the tab
can render the reason inline; only invalid parameters return 400. One flaky request
should not blank the tab.

## The CLI contract

`lib/museav.js` is the only file that knows what the CLI output looks like. museav-cli's
stdout/stderr split *is* its scripting contract:

| Command | stdout (source of truth) | stderr (human-facing extras) |
|---|---|---|
| `museav projects` | one id per line | name / gen x/y / assets n / brand |
| `museav projects assets --project X` | `id<TAB>url` | name / media_type / tags |
| `museav jobs` | **a full JSON array** | a pretty summary |
| `museav projects create --name N` | the new id | success message |

So: truth comes from stdout, stderr only fills in display fields. If stderr stops
parsing, the view degrades (name falls back to the first 8 chars of the id, asset name to
"(unnamed)") rather than going blank. `test/museav-parse.test.mjs` pins this against
output captured from the real CLI.

> Known fragility: `museav projects` and `projects assets` have **no `--json`**, so the
> display fields can only be parsed out of the stderr table. Adding `--json` to those two
> commands in museav-cli removes the fragility for good — the source of truth there is
> `~/dev/muse/museav-cli/src/commands/projects.ts`, a different repo.

## Tests

```sh
npm run build:client && npm run check && npm test
```

Route tests substitute a stub executable for the CLI (via the `MUSEAW_BIN` environment
variable) — no network, no real account.

Why an env var rather than plugin config: functional plugins in this Cordis cannot read
`ctx.config` (declaring it leaves the plugin `pending (waiting for service: config)`).
`MUSEAW_BIN` also matches the convention already used by the MCP bridge in the profile.
It is read per request, so changing it needs no plugin reload.
