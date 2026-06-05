# Coruscant Council Room — line-ending fix

The `kr2cc.set` shipped in this repo is stored with **LF-only** line
endings in the git blob, which crashes the NWN Windows toolset's
**File → New → Area** dialog at tileset enumeration time. The crash
gives no error message, no log entry, no crash report — the toolset
just dies.

This is caused by `core.autocrlf = input` (or `true`) on whoever first
committed the file — git silently converts CRLF → LF on commit, so the
local working copy looks fine but every clone/download gets LF.

## Two files in this package

### `kr2cc.set`

Same content, repacked with proper **CRLF** line endings (779 CRs,
779 LFs). Drop this into the repo at:

```
Tileset Projects/Coruscant Council Room/SET/kr2cc.set
```

…replacing the existing one.

### `.gitattributes`

Put this at the **root** of the repo. It marks every NWN file format
as `binary` so git never touches their bytes again — no future
contributor can accidentally re-corrupt them regardless of their
`core.autocrlf` setting.

`*.2da` and `*.nss` are kept as `text eol=crlf` because they really are
text and NWN expects CRLF on those.

## Re-commit flow

After dropping in both files:

```bash
git add .gitattributes
git commit -m "Add .gitattributes for NWN file types"

git add --renormalize .
git commit -m "Normalize line endings per .gitattributes"

git add "Tileset Projects/Coruscant Council Room/SET/kr2cc.set"
git commit -m "Fix kr2cc.set line endings (CRLF)"

git push
```

After this lands, anyone downloading via curl, browser, or
`git clone` (any platform, any autocrlf setting) gets a CRLF copy of
`kr2cc.set` and the toolset accepts it cleanly.

## Verifying the fix

After pushing, anyone can verify with:

```bash
curl -sSL 'https://raw.githubusercontent.com/Lamarr13/Sotor-projects/main/Tileset%20Projects/Coruscant%20Council%20Room/SET/kr2cc.set' \
  | tr -d -c '\r' | wc -c
```

Should print `779` (or whatever the new line count is). If it prints
`0`, git is still munging on commit and `.gitattributes` didn't take.
