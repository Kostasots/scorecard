# Scorekeeper

Files
- index.html  – scorekeeper (phone web app, works offline, saves on the device)
- board.html  – tournament live scoreboard (embed in the master site)
- schema.sql  – Supabase tables + PIN-protected write function (only needed for live sync)

Deploy: put index.html and board.html in a GitHub Pages repo (same as the VU Review Log / assessment app).

Live sync setup (about 10 minutes)
1. New Supabase project -> SQL editor -> run schema.sql.
2. Insert a tournament row: insert into tournaments(code,name,write_pin) values ('regionals-2027','USFHL Regionals 2027','4821');
3. Scorekeeper -> Settings: enter tournament code, PIN, project URL, anon key. Pill turns "live ✓".
4. Master site: board.html?url=PROJECT_URL&key=ANON_KEY&t=regionals-2027   (add &embed=1 to hide the header, &pitch=2 for one pitch)
   Or <iframe src="board.html?...&embed=1">. Board refreshes every 4 seconds.

Notes
- Anyone can read scores; only holders of the PIN can write. The anon key is public by design.
- If cell service drops, changes queue on the phone and send when it returns.

Multiple fields
- Each phone/tablet scores its own match (one match per device). All of them push to the same tournament code.
- board.html shows a column per pitch (up to 4 pitches) or compact tiles (5+, built for 16 fields; tap a tile for its goals and cards). Buttons switch between By pitch / Tiles / List. Any PC or TV can open it.
- The scorekeeper's Matches screen also lists every field live. "Take over" loads the latest copy if a device dies.
- Don't score the same match on two devices at once (last write wins).

If the phone is interrupted (call, alarm, text, lock)
- Every tap is saved to the phone immediately; the clock runs on timestamps so it keeps counting.
- A half-entered goal/card is restored on return. Unsent updates are kept and sent automatically when the phone is back online.
- Screen-awake is on by default (needs https, e.g. GitHub Pages). Use "Add to Home Screen" for best results.

Scale and multiple organizations
- 16 fields is well within range: scoreboards fetch only matches that changed since their last check, so an idle board costs almost nothing.
- Each organization/event is one row in `tournaments` (its own code + PIN). The code is needed to read scores, the PIN to write. Pick codes that aren't guessable.
- Settings -> "Organization name" sets the title bar; "Starters per side" sets the lineup counter.
- schema.sql was updated (reads now go through get_board/get_full). If you ran the earlier version, run the new file again.

Sports (other sports are ready, not just planned)
- Built-in profiles: Field hockey, Soccer, Lacrosse (women's), Lacrosse (men's), Baseball, Softball. Each match carries its own sport, so one tournament can mix sports.
- Goal sports (field hockey, soccer, lacrosse): goals with type, minute, scorer, assists (soccer/lacrosse), own goals (soccer), and cards/penalties.
- Baseball/softball: line score by inning with +Run, End half-inning, hits and errors; extra innings are automatic.
- A profile is one entry in the SPORTS table at the top of index.html (goal types, card types, periods, starters, default standings rules). Verify the defaults against your league's rules.
- Not included yet: lacrosse time-remaining recording, soccer stoppage-time minutes (45+2), baseball batting order/pitching stats.

Pools and standings
- Give matches a Pool (new-match form, Setup tab, or the schedule CSV). Standings build per sport and pool.
- board.html -> Standings tab (or ?view=standings, &pool=Pool%20A to show one pool). Updates with every score; games in progress count and are flagged "live".
- Rules (points for win/draw/loss and tie-break order) are in Settings -> Pool standings rules, and are sent to the scoreboards. Tie-breakers: h2h, gd, gf, ga, cards, wins. Teams still tied after all tie-breakers show a *.
- Import schedule: Settings -> Import schedule (CSV: date,time,pitch,pool,home,away[,game,sport]). Officials then tap "Score this" on their phone. Re-importing updates, never duplicates.
- schema.sql adds get_tournament and set_config: run the new file again.

Ties, crossovers and shootouts
- Pool play can end tied: a tie scores the "draw" points in Settings -> Pool standings rules (default 1; set it to whatever your event uses).
- Each match has a type: Pool play or Crossover / knockout (Setup tab, new-match form, or a "stage" column in the schedule CSV). Only pool matches feed the standings.
- Shootout / penalty strokes (field hockey and soccer): attempts per team = 1, 3 or 5 (default in Settings, changeable per match in Setup before it starts). On a tied match the Live tab offers "Start shootout". Pick who shoots first, then tap the shooter and Scored / Missed. It ends as soon as a team can't be caught, goes to sudden death if still level, and has Undo.
- The regulation score stays as played (e.g. 1-1) and the shootout shows beside it (SO 1-2, winner named) on the scoreboards, exports and the sign-off page. Marking a knockout match final with no winner asks for confirmation.
