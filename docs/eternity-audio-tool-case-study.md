# Static review case note: Eternity Audio Tool 1.0beta1

## Scope and provenance

This note captures a prior static review of the Qt interface and `eternity_common` source for Eternity Audio Tool 1.0beta1. The executable and game were not run. The inspected source tree is not present in this documentation worktree; its exact commit/hash, filenames, and line ranges were not preserved in the handoff. No REA Evidence bundle or source-location index is available here. These are therefore **provisional source-review findings**, not a reproducible REA analysis report or a runtime support claim.

Before relying on the details for another release or presenting them as independently reproduced, acquire the user-authorized source snapshot, record its identity and SHA-256, and attach precise source file/range evidence to each claim. This note paraphrases behavior; it includes no source code, proprietary executable, game file, or extracted asset.

## Findings recorded from the static pass

| Area               | Finding                                                                                                                                                                                                             | Evidence type                                                  | Unknown / not established                                                               |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| ACB dependencies   | The UI can use an internal AWB or look for an external AWB with the same base name.                                                                                                                                 | Reported static UI/source observation                          | Every naming variation, missing-file behavior, and real container compatibility.        |
| Other containers   | CSB workflow requires CPK; UAsset workflow requires ACB and internal AWB.                                                                                                                                           | Reported static UI/source observation                          | Whether all game/build variants follow this route.                                      |
| Codec labels       | HCA, ADX, AAX, and CWAV are recognized; unknown formats may be shown as BIN.                                                                                                                                        | Reported static implementation observation                     | Decoder correctness, every codec variant, and whether a BIN can be safely replaced.     |
| WAV import/replace | WAV input is expected to be PCM or IEEE float; Replace appears to validate channel count more strongly than full sample-rate compatibility.                                                                         | Reported static UI/common-library observation                  | Acceptance boundaries for every sample width, rate, channel layout, and malformed file. |
| ACB metadata       | High sample-rate metadata may be truncated.                                                                                                                                                                         | Reported static serialization observation                      | Exact cutoff, affected versions, and audible/game impact.                               |
| Save path          | Existing ACB/AWB outputs are removed before temporary outputs are renamed into place.                                                                                                                               | Reported static control-flow observation                       | Crash/power-loss behavior, filesystem edge cases, and atomicity.                        |
| WAV repair         | A malformed-loop repair can write back to the selected source path.                                                                                                                                                 | Reported static UI/save-path observation                       | Every repair mode and output-file choice.                                               |
| HCA cipher         | Automatic inference is ambiguous for mixed cipher inputs or `CIPH=56`; the observed fallback is cipher 0.                                                                                                           | Reported static inference observation                          | Meaning of all cipher modes and actual encoded output acceptance.                       |
| Loop and combine   | The FFmpeg backend does not expose loop markers. Combining tracks requires 2–8 inputs with identical sample-frame count, rate, and channel count. A loop-related bug in combined HCA was suspected, not reproduced. | Reported static backend/UI review plus an unverified suspicion | Marker preservation, actual HCA loop behavior, and playback result.                     |
| Batch/add-track    | Duplicate or missing mappings may collide/abort; add-track sequencing appeared fragile.                                                                                                                             | Reported static control-flow observation                       | Determinism across inputs, the exact triggering sequence, and game behavior.            |

“Reported static” means the prior source pass described this code path; no raw REA Evidence ID or exact source span was retained. Do not upgrade this label to “runtime observed” or “verified in game.”

## Risk controls before any edit

- Keep the original bank set read-only. ACB/AWB companions must be copied together.
- Make edits on a disposable copy and save to a fresh, separate destination; the reported Save sequence is destructive to existing outputs.
- Repair only a duplicate WAV. Hash the source before and after to demonstrate it remained unchanged.
- Use explicit one-to-one track IDs. Do not automate batch import or add-track while any mapping or sequencing is ambiguous.
- Treat codec recognition as classification only. Do not replace unknown/BIN tracks or CWAV.
- Do not rely on automatic HCA cipher selection in ambiguous cases. Resolve it from a known compatible sample or stop.
- Avoid loop modifications until a round trip demonstrates marker preservation. Reopen and decode every output before testing it in the game.
- Any runtime test must use an authorized, backed-up offline profile; do not interact with an online/anti-cheat-protected client.

## Reproduction plan (not yet executed)

1. **Identify the source.** Record the tool version, source archive/tree digest, relevant file paths and line ranges. Keep source material outside public repositories.
2. **Confirm file routing.** Use disposable ACB samples with internal/external AWB companions, a CSB/CPK sample, and a UAsset sample. Record successes and failures without inferring from filenames.
3. **Probe import validation.** Import PCM and float WAVs that vary one parameter at a time (sample rate, channels, bit depth, frames); reopen the output and inspect ACB metadata. This is needed to determine whether the suspected rate truncation has a concrete boundary.
4. **Exercise destructive paths safely.** On copies only, run Save As and loop-WAV repair. Hash the untouched originals and record the new files and any deletion/rename sequence.
5. **Test combine boundaries.** Confirm two and eight compatible inputs, then reject one and nine, plus mismatches in frame count, rate, and channels. For HCA, include a looped sample and compare decoded output/markers.
6. **Resolve cipher and batch behavior.** Use known HCA samples with a single known cipher, a mixed set, and `CIPH=56`; record selected values and decode results. Test duplicate and missing batch mappings separately from add-track sequencing.
7. **Verify in game.** Only after bank-level checks, test an authorized offline profile with a backup; listen for the intended event, timing, loop, channels, silence, and clipping.

No step above is represented as completed by this note. A WAV manifest checker in a separate modding repository can validate input structure and explicit mapping constraints, but cannot parse Eternity banks or prove any tool/game behavior.
