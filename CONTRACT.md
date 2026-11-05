# Rich Text Contract

> **In short:** everything a layer needs to accept, check and answer formatted text with `@nuxt-layers/rich-text`, and nothing else. It is a stateless library: it persists no data, takes nothing from the host, has no settings, permissions or screens, and never renders HTML.

## Install

```json
"@nuxt-layers/rich-text": "github:nuxt-layers/rich-text#master"
```

Import it like `zod`, from the package root only; it is not a Nuxt layer, so it is never in `extends`. It needs Node 22 or later and `zod` 4. Your lockfile must record it as its `codeload.github.com` tarball, never `git@github.com`.

## Settings

None.

## Host passes

Nothing. There are no ports and no composition call.

## Functions and values

| Export | Takes | Answers |
|---|---|---|
| `richTextSchema({ profile, max })` | A profile and the longest source allowed, at most `RICH_TEXT_MAX_LENGTH` (1,000,000) | A zod string that normalises its input and refuses anything beyond its profile, longer than `max`, or holding a control character. Its profile is in its metadata under `RICH_TEXT_META_KEY` (`x-rich-text`) |
| `richTextFields(schema)` | Any zod schema | `{ path, profile, max }[]`, one per formatted field. A path is a name, then `.name` into an object or `[].name` into each item of an array: `intro`, `blocks[].body`. Plain strings are never listed |
| `richTextFieldSchema` | | Checks a `{ profile, max }` declaration |
| `normaliseRichText(source)` | A string | Unicode NFC with `\n` line ends |
| `checkRichText(source, { profile, max })` | A normalised source | `{ problems, outside, documentIds, assetIds }`: `problems` is empty when it may be saved; `outside` lists each construct used beyond the profile, once; the other two list each `doc:` and `asset:` target, once |
| `parseRichText(source, profile)` | Any string | `RichTextNode[]`, the safe tree. Never throws |
| `richTextTreeSchema`, `richTextNodeSchema` | | Check a tree received from elsewhere, nested no deeper than `RICH_TEXT_MAX_DEPTH` (64) |
| `safeHref(href)` | A link destination | Whether the format keeps it |
| `RICH_TEXT_PROFILES`, `RICH_TEXT_CONSTRUCTS`, `RICH_TEXT_PROFILE_CONSTRUCTS`, `RICH_TEXT_EXTENSIONS`, `RICH_TEXT_LINK_SCHEMES`, `RICH_TEXT_MAX_DEPTH`, `RICH_TEXT_MAX_LENGTH`, `DOC_LINK_PATTERN`, `ASSET_LINK_PATTERN`, `SITE_LINK_PATTERN`, `UNSAFE_CHARACTER_PATTERN` | | The format's rules, as data |

Types: `RichTextProfile`, `RichTextConstruct`, `RichTextField`, `RichTextFieldPath`, `RichTextCheck`, `RichTextProblem`, `RichTextNode`.

### Problems

| Code | Means |
|---|---|
| `too-long` | Longer than the field's `max` |
| `unsafe-characters` | A control, private-use or surrogate character other than tab and newline |
| `not-in-profile` | A construct the profile does not allow; `outside` names it |
| `more-than-one-paragraph` | An inline field with more than one paragraph |
| `image-without-alt` | An image with no alt text |
| `image-not-asset` | An image whose source is not `asset:<id>` |

### Profiles

| Profile | Allows |
|---|---|
| `inline` | One paragraph: emphasis, strong, links, line breaks |
| `block` | Inline, plus headings, lists and block quotes |
| `document` | Everything: also strikethrough, code, tables, rules, `asset:` images and `doc:` links |

The format is CommonMark with GFM tables and strikethrough. Raw HTML is always text.

### Links

| Kept | As |
|---|---|
| `https:` without credentials, `mailto:`, site-relative `/path` | `link` |
| `doc:<uuid>` | `document-link`, which the owning layer resolves |
| `![alt](asset:<id>)` | `image`, which the host's asset store resolves |

Any other destination, such as `javascript:`, `data:`, `http:` or `//host`, stays as its text.

### The safe tree

```ts
type RichTextNode =
  | { kind: 'text', value: string }
  | { kind: 'paragraph' | 'emphasis' | 'strong' | 'delete' | 'blockquote' | 'list-item', children: RichTextNode[] }
  | { kind: 'heading', depth: 1 | 2 | 3 | 4 | 5 | 6, children: RichTextNode[] }
  | { kind: 'list', ordered: boolean, start: number | null, children: RichTextNode[] }
  | { kind: 'code', value: string, inline: boolean }
  | { kind: 'link', href: string, children: RichTextNode[] }
  | { kind: 'document-link', documentId: string, children: RichTextNode[] }
  | { kind: 'image', assetId: string, alt: string }
  | { kind: 'table', align: ('left' | 'right' | 'center' | null)[], rows: RichTextNode[][][] }
  | { kind: 'break' }
  | { kind: 'rule' }
```

## Permissions

None.

## Screens

None. Drawing the tree is `ui`'s or the host's; editing a formatted field is `content-editor`'s, or a plain `<textarea>`.

## Headless surface

Whatever draws a tree must:

- draw nodes only, never treat a `value` as markup;
- resolve `document-link` through the layer that owns the document, and `image` through the host's asset store;
- give every image its `alt`.

## You will hit this

- **Check the normalised source.** `checkRichText` on a source with `\r\n` line ends reports `unsafe-characters`; normalise first. `richTextSchema` normalises for you.
- **Check on every save, on the server.** A screen offering only a profile's tools is not a check.
- **An inline tree is phrasing only,** with no paragraph around it.
- **Beyond the profile is text, not an error, when reading.** `parseRichText` keeps what the profile does not allow, and anything nested deeper than 64, as text.
- **Problems are codes.** Never put a source's text into a problem, message or log.
- **Plain text is not rich text.** A label, name or title is an ordinary string; there is no plain profile.
- **The rules only tighten.** Loosening a profile, or widening what the format or its links allow, needs a new ADR; tightening is a breaking change, stated in the changelog.
