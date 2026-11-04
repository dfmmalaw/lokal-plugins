# lokal-plugins

This repo is where our shared Claude setup lives. Once the sync below is set up, every merged PR that bumps a plugin's version goes out to everyone in our Claude Team org on its own, so this is the one place to add or change how Claude works for the company.

## Where things live

**Business logic** is any rule that produces a number or a decision. Booking rate, qualified leads, CPQL, CPBL, ROAS, attribution, CRM matching and job value are all business logic. It lives in code, with tests, in the system that owns the data, which today is Magnus. It never gets copied into a skill as text. Claude gets those numbers from the system through its connector instead of working them out itself, so the dashboard and Claude give the same answer.

**Skill logic** is how we do the work. Report formats, review checklists, SOPs, how we write to clients. It lives here, as skills inside plugins.

**Data access** goes through connectors.

- Our own systems get a connector built into that system's code (Magnus is the first). People sign in with their own login and see exactly what they'd see in the app.
- Outside tools like Slack, Monday or Google Drive are added by a Claude org Owner under Organization settings > Connectors. No code needed.
- A plugin only points at a connector by URL.

**Secrets never go in this repo.** Everyone who installs a plugin gets its files. API keys stay in the server's environment, and people sign in to connectors with their own accounts.

**Read-only by default.** Connectors and skills read data. Anything that would write to a client's systems needs the repo owner's sign-off first.

**Shared means it's in this repo.** People can make personal skills in Claude and share one with a teammate to try it out. Once the team relies on it, it comes in here through a PR.

## Who owns what

| Role | Who | Responsible for |
| --- | --- | --- |
| Repo owner | TBD | Reviews and merges every change here |
| Claude org Owner | TBD | Admin settings: repo sync, connectors, who gets which plugin |
| Plugin owner | Named in each plugin's README | Keeping that plugin accurate |

## Layout

```
.claude-plugin/marketplace.json     list of plugins Claude syncs
plugins/
  <plugin-name>/
    .claude-plugin/plugin.json      name, version, description
    .mcp.json                       connectors it uses, URLs only
    skills/<skill-name>/SKILL.md    instructions Claude follows
    README.md                       owner, what it does, what data it touches
```

One plugin per area of work, e.g. `magnus`, `client-reporting`, `onboarding`. Keep plugin folders inside this repo and list them in `marketplace.json` with a `./plugins/...` path, which is what the org sync expects. The entry name in `marketplace.json` has to match the `name` in the plugin's `plugin.json`. Don't put a top-level `bin/` folder in a plugin, Claude rejects those.

## Making a change

1. Branch, make the change, and bump `version` in the plugin's `plugin.json`. The version bump is what tells Claude to sync it, so don't skip it.
2. Try it on your own account first. Zip the plugin folder and upload it at Customize > Plugins > Add > Upload plugin. If you have Claude Code, run `claude plugin validate .` from the repo root to check the files.
3. Open a PR. The repo owner reviews and merges.
4. Merging the PR into `main` syncs it to everyone. A direct push to `main` doesn't start a sync, so changes always go through a PR.

If a sync fails, people keep the last version that synced until it's fixed. If a merged change breaks something, the Claude org Owner sets that plugin to Not available, which turns it off for everyone right away, and the fix goes in as a normal PR.

Business logic changes go through the system that owns them instead, e.g. a PR to Magnus with tests and the normal deploy. The dashboard and Claude both pick it up from there.

Plugins also show up in Claude Code for anyone signed in with their work account, so developers get the same setup.

## One-time setup

This is done by an Owner of our Claude org, with Dean handling the GitHub side while the repo is on his account. The repo is private at `dfmmalaw/lokal-plugins` and has to stay private or internal, since the sync won't take a public repo. A GuloSolutions owner can move it into the org next to the Magnus repos later.

1. In Organization settings, make sure Cowork and Skills are both turned on. Plugin marketplaces need both.
2. Dean adds whoever does step 3 as a collaborator on this repo, so they can see it from Claude.
3. Go to Organization settings > Plugins & skills > Add > Sync from GitHub, connect GitHub and pick this repo. If Claude asks to install the Claude GitHub App on it, send Dean the link. Only he can install apps on his account, and he approves the app's webhook permission while he's there.
4. Leave Sync automatically on. Whoever turns it on needs admin on the repo, and on a personal repo only Dean has that. So either Dean does this step as a Claude Owner, or the repo moves into GuloSolutions first. Without it, the Owner clicks Re-sync on the Marketplaces tab after each merge.
5. Set Default access to Available to install while we try things out.
6. Under Plugins & skills > Policy, set Publishing to Off. Team plans default to Open, which lets anyone publish a plugin to the whole org from the Claude app and skip this repo.
7. For each connector a plugin uses, add it under Organization settings > Connectors. Turning a plugin on doesn't add its connectors. Each person then clicks Connect on the plugin's Connectors tab and signs in with their own account.
8. Once a plugin has been tested, set it to Installed by default (or Required) on the Inventory tab.

## Magnus

The first plugin, once its connector exists. It will give Claude read-only access to Magnus: the client list, a client's scorecard, the all-clients scorecard, lead search and a lead's full detail, plus the metric definitions. The CRM data Magnus already keeps on each lead (matched customer, jobs, estimates, bookings, revenue) comes with it.

- The connector will live in the Magnus API. People sign in with their Magnus login and only see what they'd see in Magnus. Lokal staff only to start.
- Part of the scorecard math (spend, CPQL, CPBL, ROAS) still runs in the dashboard today. It moves into the API as part of this, so the dashboard and Claude read the same code.
- The plugin gets listed in `marketplace.json` once the connector is live and added under Connectors. Until then the list is empty.
- Lead names, phone numbers and call transcripts will show up in people's Claude chats. Chats on our Team plan aren't used to train Anthropic's models.

## Reference

- Manage plugins for your organization: https://claude.com/docs/plugins/admin
- Same page in the help center, with the sync rules: https://support.claude.com/en/articles/13837433-manage-plugins-for-your-organization
- Sync your organization's plugins from a repository: https://claude.com/docs/plugins/org-sync
- Plugin structure: https://claude.com/docs/plugins/build
