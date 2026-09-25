# Aidoku (KaiC's fork)

A fork of [Aidoku/Aidoku](https://github.com/Aidoku/Aidoku) built for the owner's own
TestFlight. Development happens on Windows; there is no Mac. Compile checks run on
GitHub Actions and signed builds run on Codemagic.

## Rules that are not up for debate

- Never commit secrets: the Codemagic token, signing keys, App Store Connect keys.
- Do not edit upstream files just to make the TestFlight build work. The bundle id and
  entitlement changes happen at build time in `codemagic.yaml`, so merging upstream
  never conflicts. Put your own feature changes wherever they belong.

## Remotes

- `origin` is `KaiC5504/Aidoku`, the public fork that Codemagic builds.
- `upstream` is `Aidoku/Aidoku`. To pull in upstream changes:
  `git fetch upstream && git merge upstream/main`.

## How to ship a change

1. Work on `dev`. Push. Upstream's `.github/workflows/nightly.yml` builds an unsigned
   archive on every push, which is the compile check (`gh run list --branch dev --limit 1`,
   then `gh run watch <id> --exit-status`).
2. `git checkout main && git merge --ff-only dev && git push`
3. `python scripts/codemagic.py status` (make sure nothing is already running), then
   `python scripts/codemagic.py start main` and `python scripts/codemagic.py watch`.
   If it fails, the watcher prints the last lines of the failing step.
4. Report the TestFlight build number and test steps the owner can do on the phone.

The owner has authorised commit, push, Actions and Codemagic runs without asking.
Codemagic has 500 free macOS minutes a month, shared with the owner's other apps, so
don't start a build until the Actions compile check is green.

## What codemagic.yaml does to the project

| Upstream | TestFlight build | Why |
| --- | --- | --- |
| Bundle id `app.aidoku.Aidoku` | `com.kaichuan.aidoku` | Upstream's id belongs to their Apple team. |
| iCloud, CloudKit, push, KV store and user-fonts entitlements | Removed | An iCloud container has to be registered by hand in the developer portal because the API can't do it. iCloud sync is an experimental toggle that's off by default, and it greys itself out without the container. |
| `CURRENT_PROJECT_VERSION = 3` | Codemagic `$BUILD_NUMBER` | App Store Connect rejects a build number it has already seen. |

`MARKETING_VERSION` comes from upstream (0.9). The app still compiles with
`-DCANONICAL_BUILD`, so it reports itself as sideloaded. The only visible effect is the
wording under the iCloud sync setting.

To bring iCloud sync back later: in the developer portal, create the container
`iCloud.com.kaichuan.aidoku`, enable iCloud (CloudKit, with that container) and Push
Notifications on the `com.kaichuan.aidoku` identifier, then stop deleting those keys in
the "Make the project ours" step.

## Account pieces

- Codemagic App Store Connect integration `NextStop ASC key` (team-level, shared).
- Codemagic environment group `code_signing` with `CERT_KEY_B64`. If the Aidoku app
  can't see it, copy the value from DailyStore's app settings into a group with the same
  name. Never paste it anywhere else.
- The Codemagic API token is in `~/.codemagic-token` on the Windows machine.
- The App Store Connect app record uses bundle id `com.kaichuan.aidoku`. Its name can't
  be "Aidoku" because upstream already uses it. The name on the home screen still comes
  from `CFBundleDisplayName`, which is Aidoku.
