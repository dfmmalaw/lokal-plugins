# lokal-plugins

This repo is where our shared Claude setup lives. Anything merged to `main` syncs to everyone in our Claude Team org automatically, so this is the one place to add or change how Claude works for the company.

## Where things live

**Business logic** is any rule that produces a number or a decision. Booking rate, qualified leads, CPQL, CPBL, ROAS, attribution, CRM matching, job value. It lives in code, in the system that owns the data (today that's the Magnus API), with tests. It never gets copied into a skill as text. Claude reads it through the connector, so the dashboard and Claude always give the same answer.

**Skill logic** is how we do the work. Report formats, review checklists, SOPs, how we write to clients. It lives here, as skills inside plugins.

**Data access** goes through connectors.

- Our own systems get a connector built into that system's code (Magnus is the first). People sign in with their own login and see exactly what they'd see in the app.
- Outside tools like Slack, Monday or Google Drive are added by a Claude org Owner under Organization settings > Connectors. No code needed.
- A plugin only points at a connector by URL.

**Secrets never go in this repo.** Everyone who installs a plugin gets its files. API keys stay in the server's environment, and people sign in to connectors with their own accounts.

**Read only by default.** Connectors and skills read data. Anything that would write to a client's systems needs the repo owner's sign off first.

**Shared means it's in this repo.** People can make personal skills in Claude to try things out. Once something is useful to more than one person, it comes in here through a PR.

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

1. Branch, make the change, and bump `version` in the plugin's `plugin.json`.
2. Try it on your own account first. Zip the plugin folder and upload it at Customize > Plugins > Add > Upload plugin. If you have Claude Code, `claude plugin validate .` checks the files too.
3. Open a PR. The repo owner reviews and merges.
4. Merging to `main` syncs it to everyone. If something breaks, revert the commit. While the fix is in progress the Claude org Owner can set the plugin to Not available.

Business logic changes go through the system that owns them instead, e.g. a PR in the Magnus API with tests and the normal deploy. The dashboard and Claude both pick it up from there.

Plugins also show up in Claude Code for anyone signed in with their work account, so developers get the same setup.

## One time setup (Claude org Owner)

1. This repo is private on GitHub at `dfmmalaw/lokal-plugins`. A GuloSolutions owner can move it into the org next to the Magnus repos later. Keep it private either way, the sync only takes private or internal repos. Whoever does step 2 needs access to the repo on GitHub, so add them as a collaborator first.
2. In Claude go to Organization settings > Plugins & skills > Add > Sync from GitHub. Connect GitHub, pick this repo, and install the Claude GitHub App on it if asked. Leave Sync automatically on so every merge to `main` syncs. Whoever turns that on needs admin access to the repo, since it adds a webhook.
3. Set Default access to Available to install while we try things out.
4. Under Plugins & skills > Policy, make sure Skills is on and set Publishing to Off. Team plans default to Open, which lets anyone publish a plugin to the whole org from the Claude UI and skip this repo.
5. For each connector a plugin uses, add it under Organization settings > Connectors. Turning a plugin on doesn't add its connectors. Each person then clicks Connect on the plugin's Connectors tab and signs in with their own account.
6. Once a plugin has been tested, set it to Installed by default (or Required) on the Inventory tab.

## Magnus

The first plugin. It gives Claude read only access to Magnus: the client list, a client's scorecard, the all-clients scorecard, lead search and a lead's full detail, plus the metric definitions. The CRM data Magnus already keeps on each lead (matched customer, jobs, estimates, bookings, revenue) comes with it.

- The connector is being built into the Magnus API. People sign in with their Magnus login and only see what they'd see in Magnus.
- The plugin gets listed in `marketplace.json` once the connector is live and added under Connectors. Until then the list is empty.
- Lead names, phone numbers and call transcripts will show up in people's Claude chats. Chats on our Team plan aren't used to train Anthropic's models.

## Reference

- Manage plugins for your organization: https://claude.com/docs/plugins/admin
- Sync your organization's plugins from a repository: https://claude.com/docs/plugins/org-sync
- Plugin structure: https://claude.com/docs/plugins/build
