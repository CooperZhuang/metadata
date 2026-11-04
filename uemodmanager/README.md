# UE Mod Manager metadata

`manifest.json` is read by UE Mod Manager at every start: from a Cloudflare
Pages copy of this repo first (it deploys every push), and from GitHub when the
copy does not answer.

## `ue4ss`

The UE4SS release each game should use, keyed by the game profile id
(`phantom-blade-zero`, `mortal-shell-2`, ...):

| Field | Meaning |
| --- | --- |
| `version` | Release version, such as `v3.0.1-1181-g83330635`. Users whose installed UE4SS has another version are offered an update. |
| `url` | Direct `https://` download link of the release zip (`dwmapi.dll` + `ue4ss/` at the top). |
| `sha256` | SHA-256 of that zip, as in the release's `SHA256SUMS.txt`. A download that differs is refused. |
| `mirror` | Optional: a copy of the same zip in `ue4ss/`, relative to the manifest, so it comes from the same host. Apps since 1.5.2 download it first and `url` when it fails; older apps ignore it, so `url` stays the GitHub link. |

To ship a new UE4SS build: publish the release, then update `version`, `url`
and `sha256` of the game, copy the zip into `ue4ss/` and set `mirror` (only the
current zip of each game is kept there), and commit. Users get it at their next start, a
minute after the push (GitHub itself may serve the old file for up to about 5
minutes).

A game without an entry, or a manifest that cannot be reached, simply offers no
update; installing UE4SS from a downloaded file always works.

## `recommended`

Optional: the one mod the app recommends for a game, at the top of its Mods page
(apps since 1.5.3; older ones ignore it). Keyed by the game profile id:

| Field | Meaning |
| --- | --- |
| `repo` | The mod's GitHub repo, as `owner/name`. The app reads it when the copy below cannot be read. |
| `name` | The name shown before the mod is installed. |
| `folder` | Its folder in UE4SS's `Mods`, so a copy installed by hand is recognized. |
| `text` | What it does, by language code (`en`, `zh-CN`, `zh-TW`, `vi`); English when a language has none. |
| `filePrefix` | Optional: the start of the zip's name, when a release holds several zips. |
| `experimental` | Optional: `true` makes the card's label "Recommended - Experimental". |

Remove the entry when the mod is no longer needed: the card goes away with the
next start, and an installed copy becomes a normal mod.

`mods/<repo name>/` holds a copy of its newest release (`latest.json` and the zip,
as in `app/` below). The app reads this copy first at every start, so a new release
of the mod is on offer once its copy is pushed.

## `app/`

A copy of the newest UE Mod Manager release, for the app's own update check,
which reads it before it asks GitHub:

- `latest.json` has the shape of GitHub's `releases/latest` answer. Its
  `browser_download_url` is relative: the zip next to it. `github_download_url`
  names the same zip in the GitHub release, tried when the copy fails.
- The zip itself. Only the newest release is kept here.

The app checks the zip's SHA-256 (`digest`) wherever it comes from. The release
steps write both files after the GitHub release is out.
