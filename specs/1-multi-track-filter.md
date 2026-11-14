# Filter the schedule by several tracks at once

## What this changes
On `/schedule` the Track pills toggle: clicking one adds or removes that track, and a session shows if it is in any selected track. The URL carries `?track=a,b`; All, or deselecting the last track, clears it. Any selection forces list view and disables Grid, as one track does today. `?track=a` behaves as before.

## Where
- `server/routes/sessions.js` — `GET /` line 38: split `trackSlug` on commas into `t.slug IN (?, ?)`; one value is the same query as today. `speakers.js` is untouched.
- `src/pages/SchedulePage.jsx` — `filters.track` stays the raw comma string (so `api.getSessions`, `activeKeys` and `BLOCKERS` need no change). In the Track rail (~line 165) derive `selected = filters.track === ALL ? [] : filters.track.split(',')`, toggle a slug in or out and `set('track', next.join(','))` (`set` already deletes an empty value); pills get `aria-pressed={active}`; All is lit when `selected` is empty.
- Tests: `tests/api/endpoints.test.js` (API, comma list of two slugs returns the union and one slug is unchanged); `tests/schedule.spec.js` (browser) using `visit`, `waitForResults`, `conferenceDays()` and `API` from `tests/helpers.js`, read-only so no lane or booking is needed; mobile opens the rail via the `Filters` button first.

## How it will be proved
| Done when | Check | Layer |
| --- | --- | --- |
| Two tracks show exactly both tracks' sessions; count matches the API | `trackSlug=a,b` returns the union of `a` and `b`; spec clicks two pills and `result-count` equals the API count for the pair | API + spec |
| Deselecting one leaves the other; count matches the API | spec clicks one selected pill off, `result-count` equals the API count for the remaining track | spec |
| Selection is in the URL and survives reload | spec asserts `track=a,b` in the URL, reloads, pills still `aria-pressed` | spec |
| Single-track link and API query unchanged | API test `trackSlug=a` equals today's result; spec loads `?track=a` with one pill pressed | API + spec |
| Selected pills `aria-pressed="true"`; All clears them | spec asserts the attribute, clicks All, URL has no `track` and none pressed | spec |
| Any track selection disables Grid and shows the list | spec asserts `view-grid` disabled and list rendered with two tracks | spec |
| Desktop and mobile | the spec runs in both Playwright projects | spec |

## Decisions
- Slugs in the comma list that match no track simply return nothing for that slug, with no error — OK?
- A stale `?track=` pill order follows the URL, not the rail order — OK?
