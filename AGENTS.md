<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

- Leaflet map is lazy-loaded behind ClientOnly (src/components/GymMap.tsx); Leaflet touches window at import, so never import it statically from a route.
- Post images live in a private storage bucket; long-lived signed URLs are stored on the post row because public buckets are blocked in this workspace.
- AppShell owns the shared bottom publication action and composer so posting stays consistent across tabs.
- Stories store private object paths, enforce 24-hour expiry with a database trigger and RLS, and use short signed URLs; expired stories must be excluded from every public view.
- Profiles optionally reference a gym; feed gym links and profile gym links open the same searchable map via its validated academia search parameter.
- Shared brand components reference CDN asset pointers; use the original mascot for empty and sign-in states, and honor reduced-motion preferences in interface animation.
- Sign-in lives on its own /entrar page; AppShell shows a blocking dialog until a signed-in profile has an age and accepted terms, so Google and older accounts are covered too.
- Creating a post is a dedicated /publicar page, not a dialog; public profiles live at /u/$id with follows stored in a seguidores table.
- The AI coach uses owner-scoped conversation/message tables and route-derived thread IDs; server-only Gateway streaming verifies the caller and persists completed text parts so histories never cross accounts or threads.
- Compose the coach transcript and input with installed AI Elements; mount its draggable original mascot once in the root shell, with safe viewport bounds and reduced-motion support.
- Shared posts use same-origin feed links in existing private messages; render recognized links as clickable publication actions to preserve compatibility without changing the message schema.
- Profile content tabs query authored, liked, and saved posts separately; keep editing in a scrollable dialog so the main profile remains uncluttered.
- Moderation, roles and suspension live in `perfis_restritos` (private; readable only by the owner and admins), never in `profiles`. Admin authority is enforced in the database (`eh_admin()`, `admin_*` functions); the /admin UI only hides itself. Suspended accounts are blocked by RESTRICTIVE RLS policies and their content is hidden from others.
- Birth date and the minors' term are stored via the `registrar_nascimento` function (set once, never editable). `profiles.idade` is derived from it.
- Posts keep their old `categoria` values but new posts use `geral`; the feed is a single timeline with no category filter or badge.
- Gym verification (`academias.verificada`), owner (`dono_id`), plans and subscriptions are written only through security-definer functions. Commercial prices stay in `planos` (centavos); `config_app.cobranca_ativa=false` keeps the free 30-day test mode. Billing when enabled is SIMULATED (history events only, no gateway).
- Stories use one `StoryProvider` mounted in AppShell: full-screen black viewer, lazy signed URLs, and `AvatarComStory` opens the story from any avatar that has an active ring.
- Move recognition (`src/lib/golpes.ts`) is pure geometry over MediaPipe pose landmarks with all thresholds in CONFIG_GOLPES; the camera component loads `@mediapipe/tasks-vision` dynamically (client only). Cooldown is 3–5 s and ignores frames entirely.
- First admin: run in the SQL editor `insert into perfis_restritos (user_id, papel) values ('<uuid>', 'admin') on conflict (user_id) do update set papel='admin';`
