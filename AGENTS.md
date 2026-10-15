# GazeGLIDE repository guidance

## Project context

- Read [docs/gaze_project_context.md](docs/gaze_project_context.md) before working on GazeGLIDE. It records the research goal, user-reported hardware, repository findings, proposed designs, open questions, and current phase.
- Treat the context document as a maintained handoff, not an instruction to execute every proposed milestone. Follow the user's current request and update stale context when new information is confirmed.
- At the time this guidance was created, the user wanted background discussion and documentation only. Resume feature implementation when the user requests it; a background or design question alone does not authorize implementation.

## Collaboration

- For conversational replies, first translate the user's latest request into natural English, then place the Chinese translation immediately after each English sentence in the answer.
- Distinguish user-reported facts, findings verified in code, proposed designs, and unresolved questions. Do not describe a proposed feature as implemented.
- When discussing background or design, communicate first rather than inferring a request to change code. Complete explicitly requested changes within their stated scope.
- When project decisions, hardware details, or progress change, keep the context document current so another device or session can continue from it.

## Repository and validation

- Check the actual Git root, branch, status, and remotes before making changes; preserve unrelated user changes. Branch and commit information in the context document is an inspection snapshot.
- Reuse existing components where appropriate. For the proposed initial gaze milestone, preserve the existing robot command path and support offline operation without robot actuation.
- Verify the headset model, gaze interface, camera/display layout, calibration conventions, and clock domains from supplied information or code. Comments mentioning a device model are not hardware confirmation.
- Keep raw gaze, processed gaze, and inferred interaction targets separate; avoid smoothing image coordinates across camera-view switches.
- State whether verification uses synthetic fixtures, real recordings, or live hardware. Report missing inputs and limitations; do not claim real-data synchronization or hardware validation without evidence.
