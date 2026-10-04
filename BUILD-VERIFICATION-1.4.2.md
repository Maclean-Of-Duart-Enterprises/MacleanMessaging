# Maclean Messaging 1.4.2 — build verification

## Release identity

- Application ID: `com.macleanofduartenterprises.macleanmessaging`
- Version name/code: `1.4.2` / `10402`
- Minimum/target SDK: Android 10 (API 29) / API 35
- Signed APK: `Maclean-Messaging-1.4.2-Signed.apk`
- APK size: 1,139,346 bytes
- APK SHA-256: `e29ad96e0efd07d5da83ee60408a612778855c6e64143d0f2d70d0f45f2c2b61`

## Corrected behavior

- The former inline-reply path rebuilt the conversation before `SystemMessageRepository.send()` inserted the outgoing Android provider row. The thread therefore reloaded the old snapshot, and the short provider-observer quiet period could suppress the insertion notification. A second send or reopening the thread caused the later refresh seen in the supplied screenshots.
- Tapping Send now inserts an outgoing in-memory `SMS • Sending…` or `MMS • Sending…` row into the active adapter before storage or carrier work starts. The visible message count changes immediately and the list selects that exact row under every conversation sort mode.
- Temporary rows use negative local IDs and expose no provider-backed overflow or long-press operations. This prevents delete/read/forward actions from targeting an ID Android has not created yet.
- Immediate inline replies are durably queued and then dispatched directly instead of racing a zero-delay JobService for the same claim.
- Dispatch completion automatically reopens the same in-place thread model after provider insertion, replacing the temporary row with the canonical SMS/MMS row and normal controls. The user does not need a second send, manual refresh, close, or reopen.
- Queue failures remove the temporary row and restore its text. Transport failures remain reviewable in Drafts & Outbox.

## Verification record

- All 73 production Kotlin files compiled for JVM 17 against Android API 35 without compiler errors, producing 791 class files.
- Android Build Tools 35.0.1 compiled and linked resources, generated release DEX, ZIP-aligned the APK, and passed compressed-archive integrity checks.
- Python host suites passed 52/52 checks: 12 workflow/SQLite, 5 transport-state, and 35 release-contract tests.
- The added regression contract proves that the pending row is submitted to the active adapter before background dispatch, direct inline dispatch is not registered as a competing zero-delay job, the exact row is scrolled into view, provider-only actions remain disabled, and the authoritative refresh occurs after `ScheduleCoordinator.perform()`.
- `ImapProtocolTest.kt`, `MmsImageFitPolicyTest.kt`, `MmsPduWriterTest.kt`, and `MmsPduParserTest.kt` compiled and passed.
- Compiled-manifest inspection reports package/version `com.macleanofduartenterprises.macleanmessaging` / `1.4.2` (`10402`), API 29–35, and the retained default-SMS/share integration surface.
- Compiled DEX inspection found the 1.4.2 identity and sending-state strings. No Google Messages package dependency is present.
- APK Signature Scheme v3 verification passed with one 4096-bit RSA signer.
- Signer certificate SHA-256 is `62ebb3c5d7d722b2e1174a4622f7593926204579084f94e11151ef606b32d136`, exactly matching the signed 1.4.1 APK for an in-place update.
- ZIP alignment and APK archive-integrity checks passed.
- Compiler output contains only the existing compact email-code and Android API deprecation warnings; no new warning was introduced by this repair.

## Device-specific validation still required

This build environment has no attached Android handset, ADB runtime, active SIM, or carrier account. The following checks are therefore not represented as completed:

- Install 1.4.2 over the currently signed release without clearing data and confirm retained conversations, drafts, schedules, email accounts, credentials, backgrounds, display profiles, and Android default-SMS role.
- In an already-open conversation, send the first short SMS and confirm the `Sending…` row and incremented count appear on the first tap, then become the normal provider row without sending again or leaving the thread.
- Repeat with multipart SMS, text-only MMS, image/file/voice MMS, each saved thread sort, slow attachment preparation, carrier rejection, and dual-SIM selection.
- Background or rotate the app during queueing and carrier handoff, then confirm the durable Outbox/provider state reconciles without duplication.
- Recheck incoming/outgoing carrier delivery, notification reply, email sync/send, Android sharing, keyboards, schedules, accessibility, and display/font profiles on representative Android 10–15 devices.

The APK is source-, compile-, host-test-, archive-, signer-, and package-identity-verified. It is not represented as live-device or carrier verified.
