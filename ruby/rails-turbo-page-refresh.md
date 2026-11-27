# How to live update a form page with a Turbo 8 page refresh

Instead of broadcasting replace streams with rendered partials, broadcast a `refresh` action. Every
open page then fetches itself again and morphs the result into the DOM. Each viewer renders the page
in its own session, so locale, time zone and permissions are correct without one stream per locale or
role.

```
# model, after the state change
site.broadcast_refresh_later_to(site, :applied_setting_values)
```

```
-# view
= turbo_exempts_page_from_cache
= turbo_refreshes_with method: :morph, scroll: :preserve
= turbo_stream_from(site, :applied_setting_values)

= f.number_field :threshold, data: { turbo_permanent: true }
```

## When this fits

The refresh moves rendering from the broadcaster to each viewer's own request. That pays off when the
broadcaster cannot render the right HTML for everyone:

- **Per-viewer formatting:** translations, number and date formats, time zones (`5 minutes ago`,
  `27.11.2026 14:00` in the viewer's zone), unit or currency preferences. A rendered partial has only
  one locale and zone; per-locale streams multiply the broadcasts.
- **Per-viewer permissions:** admins see edit buttons, others only values. A replace stream would need
  one stream per role, or would leak markup to viewers who should not see it. A refresh carries no
  data, and the fetch it triggers is authorized like any page load.
- **A change that shows up in many places:** a status that appears in the header, a badge, a list row
  and a warning banner. A refresh needs no target ids or partials for each place, and nothing is missed
  when the page changes later.
- **Changes from outside a request:** a webhook, a device callback or a background job has no current
  user, locale or view context to render partials with.

It fits less well for:

- **Frequent updates** (telemetry every second, progress bars): every refresh fetches the whole page
  for every viewer. The debounce only merges bursts within 0.5 s. Use targeted `update` streams.
- **Pages that are expensive to render** (external API calls, heavy queries): each broadcast costs a
  full render per open page.
- **Appending to long lists** (chat, logs, feeds): an `append` stream sends one item, while a refresh
  re-renders and diffs the whole list.
- **Forms with a lot of unsaved state:** possible, but see the next section.

## Keeping unsaved input

Morphing syncs input values with the server's HTML, so a refresh would wipe what the user typed.
`data-turbo-permanent` (with an id) makes the morph skip the input.

But the attribute applies to every Turbo visit, not only to the refresh. Without the cache exemption,
this happens:

1. The user edits an input without saving and follows a link to another page. Turbo caches a snapshot
   of the form page, including the edited values.
2. The user follows a link back. Turbo first renders the cached snapshot as a preview.
3. The fresh page arrives. Its permanent inputs have the same ids as the preview's, so Turbo keeps the
   preview's elements. The abandoned edits are back, as if they were saved.

`turbo_exempts_page_from_cache` removes the preview (and the snapshot that Back restores). Arriving
from another page then always shows the server's values, and permanent inputs only matter on the same
page: the refresh, and the redirect after a submit to the same URL (also a morph).

Remaining limit: a permanent input skips the morph even when the user did not touch it, so a value
changed on the server stays stale in that input. Keeping only edited inputs needs JavaScript instead
of the attribute: handle `turbo:before-morph-element` and call `preventDefault()` for controls whose
`value`, `checked` or `selected` differs from `defaultValue`, `defaultChecked` or `defaultSelected`.

## Gotchas

- `turbo_refreshes_with` and `turbo_exempts_page_from_cache` call `provide :head`, so the layout needs
  `= yield :head` inside `<head>`. Without it the meta tags are silently dropped.
- Stimulus controllers that set state on `change` do not see a morph, and a permanent select can
  disagree with server-rendered markup around it. Re-run them with an action on `turbo:morph@document`.
- The browser whose request caused the broadcast skips the refresh (matched by request id). That only
  matters if a user's own action triggers the broadcast.
- The development `async` cable adapter only delivers broadcasts made inside the server process. A
  broadcast from a runner or a separate job worker needs the `redis` adapter.
- `broadcast_refresh_later_to` is debounced: a `Turbo::ThreadDebouncer` waits 0.5 s, then enqueues
  `Turbo::Streams::BroadcastStreamJob`. A test has to wait for the debouncer before asserting:

```
include Turbo::Broadcastable::TestHelper

perform_enqueued_jobs do
  Model.apply_change(site)
  Turbo::StreamsChannel.refresh_debouncer_for(site, :applied_setting_values).wait
end

assert_equal 'refresh', capture_turbo_stream_broadcasts([site, :applied_setting_values]).sole['action']
```
