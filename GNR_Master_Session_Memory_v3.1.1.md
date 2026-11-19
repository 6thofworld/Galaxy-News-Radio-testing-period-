# Fallout 34 – Galaxy News Radio (GNR)
## Master Session Memory and Project Notes — v3.1.1 (19 November 2026)

**Purpose:** The single handoff document for continuing the Fallout 34 Galaxy News Radio investigation. A new person or AI should be able to understand the project from this file alone, then open raw evidence only for the specific question at hand (see Section 11).

**What changed in v3.1.1:** fixes documentation errors found in the v3.1 pre-flight audit: package/manifest file counts, an outdated cross-reference in the console note, and an over-broad sentence implying every INFO condition is a simple AND. It also clarifies where the package root folder sits inside the ZIP and preserves the exact v3.1 master in `9_Archive_Superseded/`. The technical corrections introduced in v3.1 remain in place. Claude A's original PDF and searchable text are retained in `6_Session_Updates/`. Earlier v3.1, v3, v2 and v1 masters are preserved for history.

**Evidence rule:** Keep direct file findings, user-reported in-game results, independent-tester reports, logical deductions, and untested ideas separate. Do not describe a likely cause as a proven cause. The assistant has not run the game; in-game results are reports from the user or testers.

## Read this first — current position

- **Best current lead:** on `Reliable 01`, song playback correlated with whether the `SongsPlayed` condition was present. This is not yet a proven root cause because the runtime value was never measured and the fresh-reload procedure was not confirmed.
- **Important wording:** the Reliable 01 baseline was run with **no GNR diagnostic test ESP** (`TestA`, `TestB`, or `A0`–`A3`) enabled. The user explicitly confirmed that `GNR_Everywhere_F34.esp` and `GNR No Static_F34.esp` remained enabled throughout. Do **not** describe this as a test with every GNR-related ESP disabled.
- **Test naming:** use the canonical IDs and filenames A1, A2 and A3. The user colloquially called them “Test A01”, “Test A02” and “Test A03”; those are aliases, not separate plugins.
- **Missing evidence:** the separate `scripts.zip` mentioned in Claude A's transcript (9,301 compiled scripts) is not in this package. The original `Fallout34.esm` / `Fallout34old.esm` binaries are also not included.
- **Next technical gap:** the 1,786 `F34_QSnap` mentions in `scripts.zip` were not classified into definitions versus real calls. Do not treat 1,786 as a call count.

### Evidence labels used in this document

- **Record / bytecode finding:** derived from a plugin, export, or script listing; note the exact file and whether it was independently rechecked.
- **Screenshot finding:** read from a user's FO4Edit or Mod Organizer screenshot; it may apply only to the visible record or snapshot.
- **User-reported result:** something the user observed in-game; it is not a direct runtime measurement by the assistant.
- **Independent-tester report:** an observation from another person's game setup; keep it separate from the user's runs.
- **Inference / unknown:** a plausible interpretation or an unresolved question, never a confirmed fact.

## Contents
1. [Current project status](#1-start-here-current-project-status)
2. [How GNR works and the counter cycle](#2-plain-language-explanation-of-how-gnr-works)
3. [ESM versions](#3-esm-versions-and-what-changed)
4. [Test plugins and reported results](#4-test-plugins-and-reported-results)
   - [4A. Independent testers and plugin inventory](#4a-independent-testers-plugin-inventory-and-master-file-safety)
   - [4B. Earlier investigation notes](#4b-earlier-merged-notes-17-nov-2026-pdf-f34-gnr-newpdf)
   - [4C. Reliable 01 tests](#4c-reliable-01-tests-in-claude-a-18-19-nov-2026)
   - [4D. Screenshots and auxiliary plugins](#4d-screenshot-and-auxiliary-plugin-findings-from-claude-a)
   - [4E. Main-script functions and caller search](#4e-main-script-functions-and-the-unfinished-caller-search)
5. [Archive and script findings](#5-what-the-project-archive-contains-and-what-the-scripts-show)
6. [Unknowns](#6-what-is-still-unknown)
7. [Recommended next steps](#7-recommended-next-steps)
8. [Cautions and working rules](#8-important-cautions-and-working-rules)
9. [Glossary](#9-quick-glossary)
10. [Archive inventory](#10-archive-inventory-and-coverage)
11. [File map](#11-file-map-where-the-raw-evidence-lives)
12. [Test-log template](#12-test-log-template)
13. [Starting a follow-up AI session](#13-how-to-start-a-follow-up-ai-session)

---

## Evidence ledger at a glance

| Class | Statement | Confidence |
|---|---|---|
| Verified from records | Song INFOs are silent handoffs gated by `SongsPlayed`, `LastSongPlayed1-4`, `SongIntro`, quest-running and transmitter conditions | High (record/plugin inspection) |
| Verified from records | Current ESM has a 0 → 1 → 2 → reset-to-0 counter cycle; reset offers only at N5 and N66 | High on records; **not confirmed in-game** |
| Verified from Claude A's bytecode reading | `F34_QRestore` can restore a nonzero snapshotted `SongsPlayed` value when the current value is 0; `F34_FragBegin/End` only pause a periodic check briefly; `F34_GameModeTick` does not change the song counter | Reported as directly read from `radiognrscript.pas`; not independently re-parsed in this session |
| Reported by user | On `Reliable 01`, baseline with no diagnostic test ESP = talk only; A1 = talk only; A2 and A3 = songs + talk | The two auxiliary GNR ESPs stayed enabled; reload procedure not fully recorded; A1 listen was shorter |
| Reported by user | A2/A3 played for a good amount of time and did not cut song duration; exact minutes/song count not recorded | Moderate |
| Reported by user, later retracted | A newly created save appeared to play songs with all patches off, but later became messed up after a few songs | **Do not use as evidence that patches are unnecessary** |
| Reported by user | `Fallout4` automatically creates an `exitsave` on quitting; user disabled use of that save for later tests | High as user report; effect on prior test state remains unknown |
| Reported by independent testers | Test A alone and Test B alone made radio work on their systems; disabling them broke it again | Separate external observations; do not merge with user's tests |
| Inferred | The `SongsPlayed` condition is the main condition correlated with silence in the Reliable 01 tests | Working inference, not proof of the actual runtime value/cause |
| Unknown | Actual `SongsPlayed` value in Reliable 01 when song INFOs are evaluated | Never measured |
| Unknown | Which script or game mechanism enables at least one transmitter at runtime | Not identified |
| Unknown | Which compiled scripts call `F34_QSnap` / `F34_QRestore`, beyond the older README's report of 56 CG02/CG03 fragment callers | Claude A's broad search was unfinished; see Section 4E |

# 1. Start here: current project status

The project is troubleshooting **Galaxy News Radio (GNR)** in Fallout 34, the Fallout 3 / New Vegas content port to Fallout 4. Three Dog's dialogue can play while music does not. The goal is to find why some saves show talk-only radio, identify the exact condition or saved state responsible, and make a safe fix without changing unrelated parts of the mod.

## Current best user test: Reliable 01

`Reliable 01` is a pre-project save that the user describes as showing the common GNR problem. In the Capital Wasteland, connecting to GNR with no diagnostic test patch produced Three Dog's conversation but no songs; there were gaps while the radio failed to find a song and moved on to the next conversation.

The user then tested the diagnostic patches one at a time. The reported outcomes were:

| Setup on Reliable 01 | Reported result | Important limitation |
|---|---|---|
| Baseline: no diagnostic test ESP | Conversation only; no songs | `GNR_Everywhere_F34.esp` and `GNR No Static_F34.esp` were still enabled; exact listening time not recorded |
| `F34_GNR_TestA1_SongsPlayed.esp` (user called this A01) | Conversation only; no songs | User listened for less time than A2/A3 |
| `F34_GNR_TestA2_LastSongPlayed.esp` (user called this A02) | Songs + conversation | Played for a good amount of time; exact minutes/song count not recorded |
| `F34_GNR_TestA3_SongIntro.esp` (user called this A03) | Songs + conversation | Played for a good amount of time; exact minutes/song count not recorded |
| `F34_GNR_TestA0_AllThree_control.esp` | Earlier report said silent | Not a fully documented timed Reliable 01 run |

The user said the test plugins were loaded one at a time and the same plugin list was used. The user explicitly confirmed that both `GNR_Everywhere_F34.esp` and `GNR No Static_F34.esp` stayed enabled during every Reliable 01 run. Therefore, “baseline/no patch” in this section means **no diagnostic test ESP**, not that every GNR-related ESP was disabled. It was **not confirmed** whether the exact original `Reliable 01` save was freshly reloaded before every run, with no intervening save writes. Keep that as an open test-method detail.

## Corrected interpretation of the earlier "new save, no patches" result

An earlier experiment seemed to show that a new save played songs with all patches disabled. The user later corrected that conclusion: the radio played for several songs, then became messed up again. The user thinks the Fallout 4 automatic `exitsave` may have retained state after A2/A3 tests, but that explanation has **not been proven**. Fallout 4 creates an `exitsave` when the game is exited; the user disabled using that save for subsequent tests.

**Retract the earlier v2 claim that a new save works normally with all patches off.** It is not valid evidence that the patches are unnecessary. Do not use the earlier old-save/new-save comparison as a controlled result; the identity and history of those saves were not pinned down in Claude A.

## Best current lead

Claude A parsed the test plugins and found that the original INFOs and A1 retain `SongsPlayed` checks, while A2 and A3 do not. The user's Reliable 01 results follow the same pattern: silence when the `SongsPlayed` check is present, songs when it is removed. This makes `SongsPlayed` the strongest lead, but does not prove the runtime value, why the condition fails, or whether a save/scene-state issue is involved.

The current ESM includes a reset path for the counter (Section 2). The actual `SongsPlayed` value in Reliable 01 has not been measured, and the reset route has not been confirmed in-game.

# 2. Plain-language explanation of how GNR works

GNR is controlled by **radio scenes**. A scene is a sequence of steps that can wait, start dialogue, play a song, or hand control to another scene.

A song is selected through a silent dialogue record called an **INFO**. The INFO has no spoken line and no visible text; it checks conditions and starts the scene phase that plays a licensed song. If no song INFO's conditions pass, the scene moves on to another talk phase. That matches the symptom of hearing Three Dog without music.

The song INFOs check conditions involving:

- `SongsPlayed`, a counter stored on the GNR quest script;
- `LastSongPlayed1-4`, which prevent a recently played song from immediately repeating;
- `SongIntro`, used for the song-introduction sequence;
- whether the GNR quest is running; and
- whether one of the two transmitters is enabled.

An INFO's conditions must be satisfied according to their condition flags. Do not assume every row is combined as a simple AND: an `Or` flag can make a group of rows alternatives. For INFO `0AED5399`, Claude A's screenshot showed the two transmitter checks grouped as transmitter 0 **OR** transmitter 1; the other required checks, including `SongsPlayed == 0` on that INFO, still apply. Do not generalise this exact row layout to every song INFO without checking it.

## The counter cycle – important correction

A later direct inspection of the current ESM corrected the earlier simplified theory that only counter values 0 and 1 are used. The later finding is:

1. One group of songs is offered when `SongsPlayed == 0`; playing one adds 1.
2. Another group is offered when `SongsPlayed == 1`; playing one adds 1 again.
3. A third group is offered when `SongsPlayed == 2`; playing one resets the counter to 0.
4. The intended cycle is therefore **0 → 1 → 2 → reset to 0** (three songs per cycle), not just 0 → 1 → reset.

The reset songs are only offered at two talk points, **N5 and N66**. They are not offered at the normal song-offer phases in the N34–N69 scene range; N66 is a separate talk phase where a reset offer is expected. The current ESM contains 104 INFOs using reset fragments, and all were reported to be in topics called by the scenes. The S73–S84 song path takes the counter to 2 and then flows into N66, where a reset song should be offered.

This flow was confirmed from plugin records, but **has not been confirmed in-game**. It corrects the earlier README claim that no live reset path had been found / that any value above 1 blocks every song. Those statements were based on an incomplete view of where reset fragments were attached.

---

# 3. ESM versions and what changed

There are two distinct builds in the project:

- `Fallout34old.esm` – the older build.
- `Fallout34.esm` – the current build.

The uploaded `Fallout34_1.esm` is byte-for-byte identical to `Fallout34.esm`; it appears to be a duplicate created with a filename suffix. The two current copies have the same size (**363,354,547 bytes**) and the same SHA-256 hash:

`16F35D6C23B9BD82E3BE3C3E7CC791DF2FFFAA29D9FF31806E7186D2FFECF7E9`

The old build's reported hash begins `53e89bfbc7df6d17323f`. The old and current files are different; the old build has about 9,000 fewer records.

## Old build

- Has one radio scene, scene A, with 435 phases.
- Its song INFOs start scene A itself, the scene that is already running.

## Current build

- Has two mirrored radio scenes, A and B.
- Adds 70 short wait phases (`N#_W`), each about 0.75 seconds, before talk phases.
- Scene A and scene B hand off to each other: song INFOs in one scene start the other scene rather than restarting the same scene.
- Retargets 172 INFOs and changes some existing ones; 468 of the 640 GNR INFOs shared between the builds were reported byte-identical.
- The current build contains a restructuring of the same radio flow area where the song failure was observed.

The conversation also reported that the old scene A has 435 phases, while the current scenes A and B each have 505 phases. The quests were reported identical between the two builds. Other non-GNR changes exist in the ESM diff but were not investigated in detail.

**What this proves:** the current ESM has a changed A/B radio-scene flow.

**What it does not prove:** who made that change, why it was made, or whether it was intended as the fix for the original problem. The old build's real-world radio behavior was not tested as part of the file comparison.

---

# 4. Test plugins and reported results

The diagnostic plugins changed conditions on the 96 song INFOs. The test variants were described as:

- **A0:** all three variable-check groups (`SongsPlayed`, `LastSongPlayed`, and `SongIntro`) added/retained (control).
- **A1:** adds the `SongsPlayed` variable-check group only; the base transmitter and quest-running checks remain.
- **A2:** adds the `LastSongPlayed` variable-check group only; the base transmitter and quest-running checks remain.
- **A3:** adds the `SongIntro` variable-check group only; the base transmitter and quest-running checks remain.
- **Test A:** base transmitter + quest-running checks.
- **Test B:** transmitter checks only; removes the quest-running check.

The reliable, latest user test is now the `Reliable 01` matrix in Section 1 and Section 4C. The earlier apparent new-save/no-patch success is retracted because the radio later became messed up after several songs and the save may have been affected by earlier test activity. Do not claim that the issue has been proven to follow save state based on that experiment.

The strongest current correlation is that the original INFOs and A1 keep the `SongsPlayed` condition and are silent, while A2/A3 remove that condition and play songs. That is a strong lead, not proof of the live variable value or root cause.

Do not repeat A0–A3 indiscriminately. First record the unanswered reload detail. If a repeat is needed, use the same original `Reliable 01` source save for every test, freshly reload it each time, keep the rest of the load order unchanged, run each variant for the same timed duration, and record songs heard.

---

# 4A. Independent testers, plugin inventory, and master-file safety
*(Added in v2 from `Fallout34_GNR_Session_Memory_2026-11-18_Continued.pdf`, which v1 did not include.)*

## Independent tester results (reported, not verified by the assistant)

| Environment | Configuration reported | Observed result |
|---|---|---|
| Independent testers | Test A alone, then Test B alone; no other archive patches | Radio worked with either; disabling the patch broke it again |
| User's own system | Earlier claim that radio worked with archive patches disabled | **Retracted by user**; see Section 1 and 4C |

Keep these as **separate observations**. Do not average them into one conclusion. The independent results strengthen the case that the Test A / Test B condition changes matter on those setups; they do not prove why the user's system differed, and they do not prove `GetQuestRunning` is the root cause.

**How this relates to Section 1:** Claude A's later clarification retracts the earlier "new save works with patches off" conclusion. The Reliable 01 results are now the main user-side test record. The independent tester results remain separate and are not explained by the user's later correction.

**Note discrepancy to keep visible:** one earlier note said both Test A and Test B played songs; another said no test result had been reported yet and defined Test B as Test A minus `GetQuestRunning`. Timing and context of those notes are uncertain.

## Diagnostic plugin inventory (structural counts from binary inspection, not runtime tests)

All six diagnostic variants reportedly override the same 96 song INFOs. Added variable checks use CTDA function ID 629 with CIS2 metadata such as `::SongsPlayed_var` and `::LastSongPlayed*_var`.

| Plugin | Structure | CTDA count |
|---|---|---:|
| `F34_GNR_TestA.esp` | Two transmitter checks + `GetQuestRunning` on the 96 INFOs | 288 |
| `F34_GNR_TestB.esp` | Two transmitter checks only (removes `GetQuestRunning`) | 192 |
| `F34_GNR_TestA0_AllThree_control.esp` | Base + `SongsPlayed` + `LastSongPlayed` + `SongIntro` groups | 814 |
| `F34_GNR_TestA1_SongsPlayed.esp` | Adds `SongsPlayed` checks (96) | 384 |
| `F34_GNR_TestA2_LastSongPlayed.esp` | Adds `LastSongPlayed` checks (382: 94 INFOs × 4, 2 INFOs × 3) | 670 |
| `F34_GNR_TestA3_SongIntro.esp` | Adds `SongIntro` to 48 INFOs | 336 |
| `GNR_Everywhere_F34.esp` | Description: increases range of two GNR transmitters in `F3_Wasteland`; effect not independently verified | — |
| `GNR No Static_F34.esp` | Overrides `F3_RadioGalaxyNewsRef01`; effect versus masters not independently verified | — |

A0 contains all three added groups (526 additional conditions). The bundle's `Patches/` folder holds all eight files (`GNR No Static_F34.esp` has a space in its name, which breaks some command-line tools).

## Master-file safety

The inspected ESPs do not rewrite `Fallout34.esm` on disk. They supply overrides while enabled; disabling one removes its override unless another plugin wins the same record. A running game can still write quest and scene state into a save, which is separate from editing a master. This statement covers the inspected ESPs only, not unrelated external tools, scripts, or manual edits.
---

# 4B. Earlier merged notes (17 Nov 2026 PDF `F34 GNR NEW.pdf`)
*(Added in v2. This PDF predates the counter-cycle correction in Section 2, so its "only 0 or 1 pass" statements are **superseded**. Items below are the parts that add something.)*

**Setup facts recorded:** FO4Edit 4.1.5f; Fallout 4 at `C:\Games\Fallout 4`; GNR scripts are in `Fallout34_2 - Main.ba2`. xEdit shows `Fallout34.esm` at index `0A` while dumps and logs show `01`; compare only the last six digits. The Papyrus log has no GNR lines; `F34MUSIC` logs a music-stack clear on every load.

**Live vs shell topics:** the live set is quest `F34_Radio_RadioGalaxyNews` (`0AD0EE64`). The 78 `F3_RadioGNR...` topics under `F34_DialogueShared` are empty shells, not the live radio.

**Ruled out on paper (as of 17 Nov):**

| Suspect | Evidence | Status |
|---|---|---|
| Missing scripts / loose override / missing variables | `radiognrscript`, `radiostation`, `radiogate` and 639 fragments in the archive; variable names match the conditions; no GNR errors in log | Cleared |
| Missing song files | All 20 `.wav` files exist under `sound\f3\songs\radio\licensed\`; every descriptor matches one | Cleared |
| Sound settings | All 20 descriptors: `AudioCategoryRadio`, `SOMStereo`, 12.78 attenuation, no looping; loudness of the `.wav` files unchecked | Cleared on paper |
| Transmitter state | 17 Nov note said both were enabled; Claude A later inspected screenshots showing both are Persistent + Initially Disabled in the master. Successful A2/A3 runs imply at least one satisfied the transmitter condition. | **Not fully resolved**; actual runtime state and enabling source unknown (Section 4D) |
| Scene phase missing or silent | B has S73 (Ink Spots), A has S96 (Cole Porter) | Cleared |
| Voice files for song INFOs | Not needed; "0 voice files" lines concern Radio New Vegas | Not a problem |
| Modder's `.fuz` / speaker theory | Contradicts how songs are built | Unsupported |
| Wrong worldspace | Still failed in the Capital Wasteland | Weakened |

**Console dead end:** the console cannot read the counter (`GetVMQuestVariable` reports "Param type 74 unimplemented"). A console read of `SongsPlayed` has already been tried and failed; do not suggest it as an untried way to measure the value. The unresolved counter-value question is in Section 6, and the remaining measurement options are discussed in Section 7.

**Other observations:** 12 of the 20 song descriptors are also referenced by the House radio scenes (`HD00HouseRadio`); the other 8 ("Generic") are GNR-only. Scene A calls song topic `0AEAECDA` at 11 points (action indexes 146, 149, 154, 156, 247, 249, 260, 264, 268, 272, 276); scene B mirrors it with `0AC92CD7`. About 34 talk segments come before the first of the 11 offer points. INFO `0AED5399` has flags Start Scene on End, Random, Random End, End Running Scene, an empty text, and a null sound file; its single fragment is named `Fragment_End` but flagged OnBegin. Claude A later inspected one FO4Edit screenshot for INFO `0AED5399`: conditions #4–#7 are "Not equal to 1" on that INFO. Do not generalise this screenshot alone to every INFO; see Section 4D. A third party's `F34_RadioFix` log (779 DJ lines, 225 song INFOs fixed across several stations) suggests radio problems beyond GNR, but none of its 87 unmatched songs is one of the 96 triggers and the log is truncated, so it proves little. Only scene A has "Begin on Quest Start".

**Earlier hypothesis list (17 Nov, in its own order):** counter stuck or reset by `F34_QSnap`/`F34_QRestore`; variables not resolving at runtime; song topics rarely reached; silent-line problem (empty-text INFO with Start Scene on End never "ending"); runtime quest/transmitter checks failing despite looking fine. Section 5 has since weakened the `F34_QSnap`/timer ideas.

**Test-outcome decision table (from that plan):**

| Result | Meaning / next step |
|---|---|
| Test A plays songs | Audio, scenes, INFOs fine; a variable condition blocks. Re-add condition groups to find which |
| A silent, B plays | The quest-running check is the blocker |
| Both silent | Song topics probably never reached: read the N34 talk INFOs (topic `0AB326D6`) and topics `0AE97EDA`, `0AD68B0C` |
| Song starts, no sound | Audio side: check the Papyrus log and descriptor paths, then test one song as a `.fuz` |

**Tools mentioned** (not in the bundle unless listed in `5_Tools/`): `ExportRecordList.pas`, `ExportSongInfos.pas`, an ExportFullInfo script, `ba2_gnr_check.py`, `ba2_extract_script.py`, `ba2_extract_gnr_script.py`, `MakeTestA.pas`, `MakeTestB.pas`. The test ESPs are diagnostics, not release patches, and would overwrite another F34 version of those INFOs. Use several fresh loads per test because the INFO is picked at random.

# 4C. Reliable 01 tests in Claude A (18–19 Nov 2026)

This section records the updated user account from Claude A's 21-page transcript. It supersedes the earlier ambiguous "old save vs new save" summary.

- `Reliable 01` is a pre-project save the user describes as a common/baseline GNR no-song save.
- With no diagnostic test patch, Three Dog's conversation played but no music was selected; there were gaps before the next conversation.
- A1 (user alias “Test A01”; file `F34_GNR_TestA1_SongsPlayed.esp`) produced conversation only.
- A2 (user alias “Test A02”; file `F34_GNR_TestA2_LastSongPlayed.esp`) and A3 (user alias “Test A03”; file `F34_GNR_TestA3_SongIntro.esp`) produced songs + conversation. The user said the songs played for a good amount of time and were not cut short. Exact minutes and song counts were not recorded.
- A1 was listened to for less time than A2/A3. A timed A1 comparison is still desirable.
- The user said patches were loaded one at a time and the same plugin list was used. Whether `Reliable 01` was freshly reloaded from the original source before every run remains unanswered.
- Both `GNR_Everywhere_F34.esp` and `GNR No Static_F34.esp` were explicitly confirmed enabled during all these tests. The test ESPs were at the bottom of the Mod Organizer list; only the selected test variant was enabled for an individual run, according to the user's description.
- `Fallout34.esm` was enabled in the shown load-order snapshot; `Fallout34old.esm` was not listed. The user's `Fallout34.esm` was reported as **363,354,547 bytes** (about 346 MB), matching the previously recorded current-build size. This confirms size only, **not SHA-256 identity**. The user's phrase “all master files were enabled” was not a complete list of every active master; the visible snapshot confirms `Fallout34.esm` enabled and `Fallout34old.esm` absent.
- The user's earlier "new save + all patches off = radio works" conclusion is withdrawn: it initially played songs but became messed up after several songs. The user suspects an `exitsave` may have retained test-related state, but no causal link was proven.

**Interpretation:** Among the tested Reliable 01 configurations, the condition that is present when songs fail and absent when songs play is `SongsPlayed`. That is the strongest current lead. The actual counter value was not read, and the unequal A1 listen time / unconfirmed fresh-reload procedure limit the strength of the comparison.

Source: Claude A transcript, especially pages 6–10 and 17. The full PDF and searchable text are in `6_Session_Updates/`.

---


# 4D. Screenshot and auxiliary-plugin findings from Claude A

## INFO `0AED5399` screenshot

Claude A inspected the user's FO4Edit screenshots and compared the shown INFO with the existing export. The transcript records the following for **this one INFO only**:

- Conditions #4–#7 compare the four `LastSongPlayed` variables as **Not equal to 1**.
- The two transmitter conditions use an `Equal to / Or` relationship for transmitter 0 and `Equal to` for transmitter 1. Claude A described the intended logical grouping as transmitter 0 **or** transmitter 1 enabled, plus the other required conditions.
- `SongsPlayed` on this INFO is **Equal to 0**.
- For this INFO's export flags only, Claude A decoded `10000000` as Equal to, `00000000` as Not equal to, and `10010000` as Equal to / Or.

Do not treat a single INFO screenshot as proof that every one of the 96 INFOs has identical comparison operators. The two song CSVs and test plugins are the broader evidence sources.

## `GNR_Everywhere_F34.esp` vs `GNR No Static_F34.esp`

Claude A inspected both plugins and compared them with the user's screenshots:

| Detail | `GNR_Everywhere_F34.esp` | `GNR No Static_F34.esp` |
|---|---|---|
| Author/description | "Fallout 34 GNR everywhere"; says it raises the range of two GNR transmitters in `F3_Wasteland` | "Fallout 34 mod converter"; description says converted for `Fallout34.esm` |
| References overridden | Two transmitter references: `0AFDDC93` and `0AD7D651` | One reference: `F3_RadioGalaxyNewsRef01` (`022488`) |
| Data observed | Frequency 99.3; modifies transmitter radio range values | No radio or static field was found; only position and a persistent flag were visible |
| What remains unknown | Whether each of the four changed values is specifically a range value, and whether the original mod description accurately describes the in-game effect | Whether the original FO3 mod removed static and whether that effect survived conversion |

The two plugins touch different references and do not directly override one another. Neither contains quest, scene, dialogue INFO, or script records, so neither directly changes the GNR song INFO conditions.

**Range values transcribed from Claude A's screenshots (recheck before relying on them):** transmitter 0 was read as changing from `48,000 / 80,000` to `1,000,000 / 1,200,000`; transmitter 1 was read as changing from `0 / 0` with “Ignores Distance Checks” to `1,000,000 / 1,200,000` without that flag. Claude A specifically said the transmitter 0 numbers were worth a second look. The plugin description says it raises range, but the exact semantics of all four values were not independently confirmed.

## Transmitter flags and runtime state

- In `Fallout34.esm`, both transmitter references `0AFDDC93` and `0AD7D651` are marked **Persistent, Initially Disabled**. `GNR_Everywhere_F34.esp` has the same flags; it did not introduce them. Neither record has an Enable Parent.
- `GNR_Everywhere_F34.esp` changes radio range data, but that is not the same as enabling a transmitter.
- A2 and A3 retained the transmitter `GetDisabled` checks and still played songs. Therefore at least one transmitter satisfied the plugin condition during those runs; Claude A interpreted this as at least one transmitter being enabled/usable at runtime. This is an inference from the successful test and condition structure, not a direct live-state read. Claude A searched the main GNR script and the 639 GNR fragments and did not find the enabling action. The actual enabler and which transmitter satisfied the condition remain unknown.
- The 17 Nov note saying both transmitters were simply "enabled" should not be treated as verified runtime state. The screenshots show their record flags, not their live state in the game.

Source: Claude A transcript pages 11–13 and 16–18. One transmitter 0 range value was read from a screenshot and Claude A explicitly recommended a second look.

---


# 4E. Main-script functions and the unfinished caller search

Claude A reports directly reading these functions from the complete bytecode listing `1_Scripts/main_script/radiognrscript.pas`. They are functions inside the main script `F3:RadioGNRScript`, **not five separate `.pex` files**. This explains why searching the BA2 archive for files named `F34_QSnap`, `F34_QRestore`, etc. returned no results.

| Function | Claude A's bytecode finding | Relevance |
|---|---|---|
| `F34_QSnap` | Before the quest starts, copies quest variables into a snapshot array; `SongsPlayed` is in slot 18 | Can preserve a nonzero counter value in a snapshot |
| `F34_QRestore` | After the quest starts, if the snapshotted `SongsPlayed` is nonzero and the current value is 0, restores the saved value; same pattern applies to other variables | Could reintroduce a nonzero value, **if it runs for the GNR quest in the failing save**; that has not been shown |
| `F34_FragBegin` | Sets a timer to current time + 2 seconds, pausing the periodic check for that short interval | Not a permanent counter lock |
| `F34_FragEnd` | Clears that timer to 0 | Not a counter reset |
| `F34_GameModeTick` | Runs the unrelated Harkness/MS08 exploit logic; does not touch `SongsPlayed` | Not the cause of a counter value by itself |

The 172 fragments that use `F34_FragBegin` call `F34_FragEnd` at the end. The pause expires by itself after about two seconds. This does not support the earlier idea that a fragment could freeze the periodic tick indefinitely or that the tick itself holds the song counter.

## Important caller-search status

The nested README already preserved in `9_Archive_Superseded/README_F34_GNR__nested_revised.md` reports **56 callers** in CG02/CG03 fragments, operating on their own quests, and says no GNR fragment calls the functions. Claude A independently found no calls in the main GNR script or the 639 GNR fragments. Those statements are not necessarily contradictory: the 56 older callers are outside the GNR fragment set.

Claude A later searched a separate `scripts.zip` reportedly containing **9,301 compiled scripts** and found **1,786 files mentioning `F34_QSnap`**. That is a raw mention count, not a confirmed count of actual call sites. Claude A had started separating function definitions from real references but the session expired before results were produced.

**Important availability note:** that separate `scripts.zip` is referenced as an attachment in the Claude A transcript, but it is **not included** in the current v3.1 Claude B package or in the files supplied to this session. Do not pretend its contents are available. To finish this branch, Claude B will need that archive (or a fresh equivalent extraction of compiled scripts) and must distinguish definitions, references, and actual call sites. The current conclusion about whether `F34_QRestore` affects GNR runtime remains unresolved.

Source: Claude A transcript pages 18–20; existing nested README for the 56 CG02/CG03 callers.

---

# 5. What the project archive contains and what the scripts show

The original upload was `Fallout34_GNR_Actual_Project.rar` (a RAR archive, not a ZIP). The current working evidence library is the `Fallout34_GNR_Lean/` folder inside `Fallout34_GNR_Claude_B_Package_v3.1.1.zip`. The current package also includes Claude A's transcript and addendum under `6_Session_Updates/`. The original project archive contained two nested ZIP files under `Papyrus_Source/`:

- `gnr_psc.zip` – 639 GNR topic-info fragment source files (`.psc`).
- `upload_me.zip` – decompilation reports and main-script diagnostic outputs, including a README.

The RAR also contains expanded copies of some project material. The two README Markdown versions are preserved verbatim in `9_Archive_Superseded/`.

## The 639 GNR fragments

The fragments are named `tif_f3_radiogalaxynewsquest_<hex>.psc`. Each is a small script attached to a GNR dialogue INFO. The hex in the filename is the original Fallout 3 INFO FormID with load-order byte `01`, not the F34 FormID. To map a fragment to the F34 INFO, open the INFO in FO4Edit and inspect its Papyrus Fragments tab.

The archive README's searches reported:

| Fragment type | Count | Behaviour |
|---|---:|---|
| Song-play increments | 120 | Calls `F34_Add_SongsPlayed(1)`; shifts `LastSongPlayed1-4` and sets `LastSongPlayed` to a song number 1–20 |
| Song-play resets | 52 | Sets `SongsPlayed = 0`; also updates last-song bookkeeping |
| Intro fragments | 52 | The README's count needs correction; see below |
| Other fragments | Remaining files | Do not change `SongsPlayed` |

All 120 increments add exactly 1; all 52 reset fragments set exactly 0. The `F34_Add_SongsPlayed` helper is uncapped. The 104 reset INFOs found in the current ESM correspond to the reset logic being present in the two-scene radio flow. This distinguishes the number of fragment scripts from the number of INFO records that attach/use them.

### Corrections to the archive README

The conversation found these mistakes or clarifications:

- **Intro fragments:** the README says 52 fragments set values 1–12. The corrected breakdown is 12 set values 1–12, 36 set the value to 0, and 4 set it to −1.
- **`Fragment_End`:** seven fragments (`01057B11`–`01057B17`) use `Fragment_Begin` and only touch `DadStoryVar`. They are not song-related. Therefore, the README claim that every fragment uses `Fragment_End` is wrong.
- **Song CSV duplicates:** both song CSVs are exactly duplicated 2×, with 407 unique rows each. The reason for the duplication is unknown.
- **Intro/song number mapping:** S61 maps to `LastSongPlayed` / `SongIntro` 12 and S72 maps to 1, in all three groups. Whether that matches Three Dog's spoken intro lines was not verified.

## Main script: `F3:RadioGNRScript`

The main source script `radiognrscript.psc` is truncated because Champollion crashes while decompiling it. The latest project README says the `.pas` file is a complete bytecode listing (60,094 bytes), and the strings extraction contains 275 identifiers. The earlier root README incorrectly treated the `.pas` output as uncertain / possibly invalid; the nested README is a revised note that corrects this point.

The useful findings from the complete bytecode listing, as described in the nested README, are:

- `SongsPlayed` is a conditional variable with initial value 0, so it starts at 0 and is visible to dialogue conditions.
- The helper-function details from Claude A's later direct bytecode reading are in Section 4E. The functions are inside `F3:RadioGNRScript`, not standalone script files.
- `F34_FragBegin` / `F34_FragEnd` only pause a periodic check briefly (about two seconds); they do not directly write the song counter or hold the timer indefinitely.
- `F34_GameModeTick` handles the unrelated Fallout 3 MS08 / Harkness logic and does not change `SongsPlayed`, `SongIntro`, or `LastSongPlayed`.
- The nested README reports 56 `F34_QSnap` / `F34_QRestore` callers in CG02/CG03 fragments, working on their own quests, and no calls in the 639 GNR fragments. Claude A's later search across a separate 9,301-script archive was incomplete; see Section 4E.

The main script's timer helpers are not the leading explanation for silence. The possible relevance of `F34_QRestore` depends on whether it can be invoked for the GNR quest in the user's runtime; that remains unproven.

---

# 6. What is still unknown

1. What exact value does `SongsPlayed` have in `Reliable 01` when the next song INFO is evaluated?
2. Is the Reliable 01 source save freshly reloaded before each A1/A2/A3 run, with no save writes between runs?
3. What were the exact listening durations and song counts for the baseline, A1, A2 and A3 runs? A1 was shorter; exact times were not recorded.
4. What code or game mechanism enables either transmitter at runtime? The records show both are Initially Disabled, while successful A2/A3 runs imply at least one transmitter satisfied the INFO condition.
5. Does `F34_QRestore` ever run against the GNR quest in the user's game? The older README reports 56 CG02/CG03 callers operating on their own quests; Claude A's broader 9,301-script scan was unfinished.
6. Where is the `scripts.zip` mentioned in Claude A's session, and can the 1,786 `F34_QSnap` mentions be separated into definitions, references and actual call sites? That ZIP is not inside the current v3.1.1 Claude B package.
7. Does Reliable 01 reach the N5 and N66 reset-song offers in-game as expected?
8. Do House radio songs play? They share 12 of the 20 sound descriptors.
9. Do silent song INFOs with the “Start Scene on End” flag behave as intended in Fallout 4? A vanilla radio INFO could be compared.
10. Do the Three Dog intro lines match the S61/S72 song-number mapping?
11. Which 12 files made up the Gist that earlier sessions referred to? The former `Source_Exports/` folder reportedly held 13 files; the mapping was not reconciled. The large exports have not all been read end-to-end in the earlier v2 pass.
12. The current ESM size matches the known current build, but was its SHA-256 hash ever checked against the exact file used in the Reliable 01 tests? Not in Claude A's session.

**Current working inference:** `SongsPlayed` is the condition that best correlates with the Reliable 01 failure. A nonzero snapshotted value could potentially be restored by `F34_QRestore`, but neither the runtime value nor a GNR call to that function has been established. Do not elevate this to a proven cause.

# 7. Recommended next steps

1. **Preserve the source save.** Keep `Reliable 01` untouched. Continue disabling/avoiding the automatically generated `exitsave` during controlled tests, as the user chose, and record exactly which save file is loaded.
2. **Resolve the test-method gap before drawing a stronger conclusion.** Confirm whether each Reliable 01 test was loaded fresh from the original source save. If not, repeat only the necessary A1/A2/A3/baseline comparison, one variant at a time, with the same load order and a matched listening duration. Record exact minutes and song counts. Do not use a save created by a previous test as the next test's baseline.
3. **Continue the compiled-script caller search.** Obtain the `scripts.zip` that Claude A used (reportedly 9,301 PEX files), or regenerate an equivalent set. Separate definitions from actual call sites for `F34_QSnap` / `F34_QRestore`; reconcile with the older README's 56 CG02/CG03 callers. Do not search for these five functions as separate filenames; they are functions inside `F3:RadioGNRScript`.
4. **Find the transmitter enabler.** Search relevant scripts and load-order plugins for actions that enable references `0AFDDC93` and `0AD7D651`. The two reference records being Initially Disabled does not explain which one becomes enabled at runtime.
5. **Measure or isolate `SongsPlayed`.** The console read already failed (`GetVMQuestVariable`: "Param type 74 unimplemented"). Use a safe diagnostic/probe approach only after reviewing the existing evidence and defining what the test proves. Do not change the counter or build another ESP yet.
6. **Check the reset route in-game.** If the test setup allows, record whether the N5/N66 reset offers are reached after the 0 → 1 → 2 → reset-to-0 cycle.
7. **Only consider a script change after confirming the failure mode.** If the counter is proven to be stuck or restored, then a targeted fix may be designed. Recompilation requires a Papyrus compiler; its availability must be confirmed before proposing a source edit.
8. **Do not rebuild or reconvert a clean Fallout 4 installation yet.** The current evidence can still be narrowed on the existing build with controlled tests and script-call inspection.

# 8. Important cautions and working rules

- An ESP patch does not permanently rewrite the ESM file on disk. Disabling a patch removes its overrides; it does not reset variables already stored in a save.
- A save can retain quest-variable values and running-scene state.
- The earlier "new save + all patches off = radio works normally" result was retracted by the user because the radio became messed up after several songs. Do not use it as evidence that patches are unnecessary.
- Do not assume `SongsPlayed` is definitely the cause until it is measured or isolated on Reliable 01.
- Do not claim the same Reliable 01 save was freshly reloaded for every test; that detail remains unanswered.
- Do not claim both transmitters are simply enabled. Their plugin records show Persistent + Initially Disabled; at least one became usable during A2/A3 runs, but the enabler is unknown.
- The Claude A `scripts.zip` is not included in the current package; the 1,786 QSnap mentions have not been classified into actual callers.
- The current ESM's reset path is present in live topics according to direct record inspection; do not repeat the earlier claim that no reset INFOs are attached to live topics.
- Do not use New Vegas-specific INFOs or New Vegas-specific radio records as evidence for the Fallout 34 GNR implementation.
- Keep original ZIPs and ESM files unchanged. Work on copies and keep tests reproducible.

---

- **Standing rules from the user's earlier sessions (added in v2):** do not guess; ask when a missing detail changes the conclusion; never say a file was read unless it was retrieved and checked to its end; do not treat filenames, summaries, or snippets as substitutes for source files; do not create or edit another diagnostic ESP until the original implementation and evidence are understood; change one variable at a time; keep a ledger of verified-from-source / reported / inferred / unknown; distinguish record overrides, master-file edits, save-state changes, and runtime behaviour.
- **Scope rule:** `ba2_radio_report.txt`, `ba2_voice_report.txt` and `radio_dump.txt` are New Vegas material. `fo4_radio_lookup.txt` and `gistfile1.txt` are Fallout 4 comparison material. None is evidence for how `Fallout34.esm` configures GNR.

# 9. Quick glossary

- **ESM:** the main plugin file that contains the mod's records.
- **ESP:** a patch plugin that overrides or changes records while enabled; it does not rewrite the ESM on disk.
- **Save state:** game values and running-game state stored in a save file.
- **Scene:** the ordered steps that control when the radio waits, talks, plays a song, or starts another scene.
- **INFO:** a dialogue record. Song INFOs are silent records that check conditions and start song phases.
- **TIF / fragment:** a small Papyrus script attached to a topic INFO.
- **Papyrus:** Fallout's scripting language.
- **PEX:** compiled Papyrus script; **PSC** is source text; **PAS** here is the bytecode listing created by the decompiler's assembly mode.
- **FO4Edit / xEdit:** a tool for inspecting and editing Fallout 4 plugin records.
- **`SongsPlayed`:** the counter used by GNR song INFO conditions.
- **`LastSongPlayed1-4`:** history values used to avoid repeating recent songs.
- **`SongIntro`:** a variable used by the intro/song sequence.

---
# 10. Archive inventory and coverage

**Original supply.** `/mnt/data/Fallout34_GNR_Actual_Project.rar` (RAR v5, 695 entries) contained two nested ZIPs and expanded copies. A later supplied `Fallout34_GNR_Actual_Project.zip` (685 files) holds the same project plus a `Reference/` folder with three notes files: the 17 Nov session memory (`.md`, identical in substance to v1's former Appendix C), the 17 Nov "merged investigation notes" PDF (`F34 GNR NEW.pdf`), and the 18 Nov continuation PDF.

- `Papyrus_Source/gnr_psc.zip`: 639 `.psc` fragment scripts. Byte-identical to the extracted `gnr_psc/` folder.
- `Papyrus_Source/upload_me.zip`: 12 files (README, two `.psc`, one `.pas`, CSV, logs). Byte-identical to the extracted `upload_me/upload_me/` folder.
- Root `radiognrscript.psc` and `.pas` are byte-identical to `psc_main/radiognrscript.psc` and `asm/radiognrscript.pas`.

**Correction to v1:** v1 stated that no PDF was present. That was true of the RAR as inspected then, but the bundle now supplied contains two PDFs. Their content is incorporated in Sections 4A and 4B, and their full text is kept in `9_Archive_Superseded/`.

**Two README versions** exist (`README_F34_GNR.md`, archive root, 10,328 bytes, SHA-256 `56be3799201e872f905f5412d5972527eae9cd3322a7a662920a9b357fb25c0e`; nested copy, 9,494 bytes, SHA-256 `f100956b39ff49b6925aa7242a5939766a6a6aaae67bb1ae4025ea3cdd3a2df6`). The nested one is the more complete revision for main-script analysis; the later findings in this master further correct the counter/reset interpretation. Both are kept verbatim in `9_Archive_Superseded/`.

**Current v3.1.1 package check:** the ZIP contains **696 files** under `Fallout34_GNR_Lean/`. `MANIFEST_SHA256.txt` contains **695 SHA-256 entries**, covering every other file; the manifest does not list its own hash. All 695 entries were checked against the files in this v3.1.1 package and matched. The original ESM binaries and Claude A's separate 9,301-script `scripts.zip` are not included.

---

# 11. File map: where the raw evidence lives
(Full per-file table with sizes: `01_FILE_INDEX.md`.)

| Question | Open this |
|---|---|
| What does a fragment script do? | `1_Scripts/fragments_639/tif_f3_radiogalaxynewsquest_<hex>.psc` (hex = FO3 INFO FormID with `01` prefix) |
| Main script logic | `1_Scripts/main_script/radiognrscript.pas` (complete bytecode); `.psc` is truncated by a decompiler crash |
| Decompile run evidence | `1_Scripts/decompile_run/` (`report.txt`, `summary.csv`, logs) |
| Scene phases and handoffs | `3_Exports_F34/F34_RadioScene_RadioGalaxyNews_0A93BF68.txt` and `..._B_0AC0FA30.txt` |
| Song INFO conditions | `3_Exports_F34/SongInfoExport(for0AC92CD7).csv`, `...(for0AEAECDA).csv`, `FullInfo_Export.txt`, `NoEditorID_0AED5399.txt` |
| Test plugins | `2_Patches/` |
| Claude A's Reliable 01 test report, FO4Edit screenshot findings, and function review | `6_Session_Updates/Claude_A_Session_Addendum_2026-11-19.md` and the source PDF beside it |
| Compiled-script caller search | The `scripts.zip` mentioned in the transcript; **not present in this package** |
| Comparison only, never F34 evidence | `4_Reference_Only/` |
| Claude A findings and original transcript | `6_Session_Updates/` |
| Older notes, READMEs, PDFs, v1/v2/v3/v3.1 masters and tree listing | `9_Archive_Superseded/` |

---

# 12. Test-log template
Copy one block per run. Record every field; write "not recorded" rather than guessing.

```
Date/time:
Save used (exact filename; e.g. Reliable 01):
Was the original source save freshly loaded for this run? (yes/no/not recorded):
ESM used (name + file size + SHA-256 if available):
Enabled diagnostic test ESP (exact filename; none/A0/A1/A2/A3/TestA/TestB):
Other enabled patches and relevant load-order entries:
Were GNR_Everywhere_F34.esp and GNR No Static_F34.esp enabled? (yes/no):
Was exitsave disabled/avoided? (yes/no):
Time listened (minutes):
Songs heard (count):
Three Dog dialogue continued? (yes/no):
Did music stop or the radio become abnormal later? (describe/time/song count):
Did the result repeat on a second fresh load? (yes/no/not tested):
Notes:
```

---

# 13. How to start a follow-up AI session

1. Give the new AI this `00_START_HERE.md` first. It is the current v3.1.1 master. The downloadable external copy is `GNR_Master_Session_Memory_v3.1.1.md`; it must match the copy inside the ZIP.
2. The ZIP has one top-level folder, `Fallout34_GNR_Lean/`. Extract it or make that folder's contents available as project knowledge/file library. If the AI can see only the ZIP filename but cannot open its internal files, it must say so and ask for the archive to be extracted; it must not claim to have read the contents. Do **not** ask the AI to ingest every raw file at once. Have it read `01_FILE_INDEX.md` and open only the specific source files needed for the current question.
3. The Claude A source transcript is `6_Session_Updates/Claude_A_Session_2026-11-18_to_19.pdf`; the searchable text and concise addendum are beside it.
4. The next unfinished technical branch is the caller search for `F34_QSnap` / `F34_QRestore`. The separate `scripts.zip` mentioned in Claude A's transcript is not in this package; the AI must request it or an equivalent compiled-script extraction before claiming that search is complete.
5. Require the AI to separate verified record/bytecode facts, user-reported game results, inference, and unknowns. Do not let it edit plugins or propose a patch before it has explained what the current evidence proves and what it does not.

A ready-to-paste starter message is in `CLAUDE_B_START_PROMPT.md` (external download: `GNR_Claude_B_Start_Prompt_v3.1.1.md`).
