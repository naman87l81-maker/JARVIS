JARVIS: Android AI Voice Operator

Master Build Prompt and Technical Specification

Target device: Android tablet and phone, initially Redmi Pad 2
Platform: Native Android application
Language: Kotlin
UI: Jetpack Compose with Material 3
Minimum Android version: Android 10 (API 29), unless a documented dependency requires a higher minimum
Target SDK: Latest stable SDK supported by the build environment
Primary objective: Build an installable Android app in which the user can call a saved AI contact, speak naturally, interrupt responses, and delegate real tasks to the AI.

---

1. Mission and product definition

Act as a senior Android engineer, AI-agent architect, Kotlin developer, UX designer, security engineer, and test engineer.

Build JARVIS, a native Android AI assistant that behaves like a callable contact rather than a conventional chat application.

The user must be able to:

1. Open JARVIS from an app icon, saved in-app contact, or supported Android shortcut.
2. Tap a Call button and enter a dedicated voice-call interface.
3. Speak naturally and receive spoken responses.
4. Interrupt the AI while it is speaking.
5. End the call immediately.
6. Give the AI instructions to create real documents, research permitted websites, and execute explicitly implemented tasks.
7. Approve sensitive operations and screen-sharing permissions.
8. Inspect task progress, results, errors, and generated files.

This must be a real application. Do not simulate successful API calls, file creation, web research, screen access, or device control.

Core design principle: The AI decides which permitted tool is appropriate. Deterministic application code validates and executes that tool. The AI must never be allowed to execute arbitrary instructions merely because it generated them.

2. Development rules

Before writing code:

- Inspect the existing repository, development environment, available SDKs, and Gradle configuration.
- Determine whether the environment can build a native Android APK.
- If working in a browser-based app builder, do not substitute a website or PWA for the requested native Android application.
- If native compilation is unavailable in the current environment, create the complete Android project source and provide a reproducible build procedure.
- State actual environmental limitations rather than claiming that an APK was built when it was not.
- Create a short implementation plan, then begin implementation without waiting for approval of routine technical decisions.
- Work incrementally, compiling and testing after each major module.
- Do not discard existing working functionality without explaining why.
- Use stable, compatible dependencies. Avoid unnecessary libraries and paid SDKs.

Maintain these files:

- "README.md"
- "IMPLEMENTATION_STATUS.md"
- "ARCHITECTURE.md"
- "SECURITY.md"
- "TESTING.md"
- "CHANGELOG.md"

The implementation status must distinguish working, partially working, and not implemented features.

3. Technical architecture

Use a modular architecture with clear separation between UI, domain logic, platform integrations, and remote services.

Suggested package structure:

"app/src/main/java/<package>/"

- "ui/": Jetpack Compose screens, components, navigation, and theme.
- "voice/": audio capture, speech recognition, text-to-speech, interruption, and call state.
- "agent/": conversation orchestration, task planning, tool selection, and execution policy.
- "ai/": provider interfaces, API clients, model configuration, and response parsing.
- "tools/": registered tool definitions, validation, execution, and results.
- "files/": TXT, Markdown, and PDF generation.
- "research/": web-search providers, page retrieval, and source extraction.
- "permissions/": permission state and consent flows.
- "screen/": MediaProjection and screen-sharing integration.
- "contacts/": in-app AI contact, recents, and shortcuts.
- "data/": Room database, settings, conversation history, and task records.
- "security/": secret handling, redaction, and sensitive-action confirmation.
- "background/": permitted foreground services and cancellable work.
- "di/": dependency injection, if needed.
- "util/": shared utilities.

Use:

- Kotlin coroutines for asynchronous work.
- "StateFlow" or equivalent observable state for UI updates.
- Jetpack Compose and Material 3.
- Android SpeechRecognizer where supported.
- Android TextToSpeech for speech output.
- Room for persistent structured data.
- DataStore for non-sensitive preferences.
- Android Storage Access Framework for user-selected file destinations.
- Android PdfDocument or an equivalent stable native PDF-generation implementation.
- OkHttp or Ktor for network requests.
- A tested dependency-injection approach, or straightforward constructor injection.

Avoid placing API calls, file operations, or task execution directly inside composable functions.

4. App identity and contact experience

Create an AI contact named JARVIS with a configurable avatar, description, and voice settings.

The main screen must include:

- JARVIS avatar and name.
- A prominent Call JARVIS button.
- Connection and provider status.
- Recent conversations.
- Recent tasks and their statuses.
- Generated files.
- A Settings entry.
- Clear error and permission states.

The contact should work inside the app. Add a supported Android shortcut or contact-launch integration where technically appropriate.

Do not invent a telephone number or claim that the AI can be dialled from the native Phone app as a cellular contact.

The initial call should be an internet-based voice session inside JARVIS. Keep the architecture extensible for a future real-time WebRTC implementation, but do not claim that a production-grade internet calling stack exists unless it has been implemented and tested.

5. Voice-call experience

Implement a dedicated call screen with:

- JARVIS avatar and connection state.
- Animated audio waveform or audio-level indicator.
- Call duration.
- Microphone mute/unmute.
- End-call button.
- Listening, processing, speaking, and error indicators.
- Transcript display that can be hidden.
- A control for stopping speech immediately.
- A task-progress panel that does not block the call controls.

Use an explicit state machine:

- "IDLE"
- "CONNECTING"
- "LISTENING"
- "PROCESSING"
- "AWAITING_APPROVAL"
- "EXECUTING"
- "SPEAKING"
- "ERROR"
- "ENDING"

Define valid transitions and prevent duplicate audio sessions.

Speech input

Implement microphone permission requests and a real audio-input path.

Preferred initial approach:

1. Capture speech using Android's available speech-recognition facilities.
2. Detect final and, where supported, partial transcripts.
3. Display the transcript.
4. Send the transcript to the AI provider.
5. Support a configurable transcription API as an alternative.

Do not assume speech recognition works offline or is supported by every installed device service. Detect availability and explain failures.

Do not repeatedly upload every partial transcript to the AI API. Debounce updates and submit a completed utterance unless the selected streaming implementation genuinely supports incremental processing.

Speech output

Use Android TextToSpeech initially.

Requirements:

- Speak AI responses as they arrive, in meaningful chunks.
- Handle initialization and language errors.
- Release audio resources properly.
- Avoid overlapping speech sessions.
- Stop speech on interruption or call termination.

Interruptions and cancellation

When the user speaks the interrupt command or presses the interruption control:

1. Immediately stop or pause speech output.
2. Cancel the previous response stream where supported.
3. Cancel the previous task only if it has not reached an irreversible operation or if the tool supports safe cancellation.
4. Begin listening again.
5. Prevent old responses from being spoken after the interruption.
6. Maintain a request identifier or generation token to ignore stale results.

Ending a call must immediately stop playback, cancel cancellable requests, close audio resources, and transition to "ENDING" followed by "IDLE".

Never report that a remote operation was cancelled unless cancellation was confirmed. If a non-cancellable operation is already running, report its actual state.

The first version may use a turn-based voice loop. Mark continuous full-duplex conversation as a future enhancement unless it is genuinely implemented.

6. AI provider integration and free usage

Create a provider abstraction such as:

- "AiProvider"
- "TranscriptionProvider"
- "SpeechOutputProvider"
- "WebSearchProvider"

Implement at least one working text-generation provider using an account-configurable API endpoint and model identifier.

Potential free-tier options include OpenRouter's currently available free models and Groq's available API services. Their limits, model availability, and eligible features may change.

Requirements:

- Never hard-code a particular free model as permanently available.
- Allow provider, model, and endpoint configuration through Settings.
- Read current model identifiers from provider configuration rather than assuming old identifiers remain valid.
- Handle HTTP 401, 403, 404, 429, 5xx, network failures, and timeouts.
- Implement bounded retries with exponential backoff and respect "Retry-After".
- Avoid retrying requests that may have already executed a task unless execution is idempotent.
- Show quota or provider failures clearly.
- Never claim unlimited free use.
- Never silently switch to a paid model.
- Never require a paid subscription merely to open the application.
- Do not send sensitive user files or screen content to remote services without appropriate disclosure and consent.

Add a Provider Settings screen showing:

- Selected provider.
- Selected model.
- Connection-test result.
- Whether credentials are configured.
- Last request status.
- Usage information when the provider exposes it.
- Clear instructions when the free quota is exhausted.

API-key security

Preferred deployment: an optional backend proxy that stores API secrets in server-side environment variables and authenticates the user's application.

For a personal, direct-API prototype, allow the user to enter their own key through Settings, but disclose that secrets stored on a client device cannot be guaranteed safe against a sufficiently compromised device or reverse engineering.

Use Android Keystore-backed encryption for locally stored user credentials where appropriate. Do not claim this makes an embedded or client-held key impossible to extract.

Never:

- Commit API keys to source control.
- Include private keys in "BuildConfig", assets, source files, or the APK.
- Print credentials in logs or crash reports.
- Send credentials to an unrelated endpoint.
- Collect developer credentials from the user.
- Create a hidden proxy or relay using another person's account.

Provide a ".env.example" only for server-side configuration and ensure real ".env" files are excluded from version control.

7. AI agent and task execution engine

The agent must perform actual operations using an explicit tool registry.

Define a tool interface with:

- Tool identifier.
- Human-readable description.
- Validated parameter schema.
- Required permission.
- Risk classification.
- Execution function.
- Cancellation support, where applicable.
- Structured result.
- Error handling.

Use strict JSON or equivalent schema validation for tool arguments.

The AI must never execute arbitrary shell commands, arbitrary Kotlin code, arbitrary URLs with unrestricted privileges, or unvalidated file paths.

Implement an agent loop:

1. Receive the user's instruction.
2. Determine whether it requires a direct response or an action.
3. Select an available tool when necessary.
4. Validate the tool name and arguments.
5. Check permissions and risk.
6. Request confirmation if required.
7. Execute the tool.
8. Validate the tool result.
9. Give the result back to the AI for a follow-up response if necessary.
10. Speak the result and record the task status.

Prevent infinite tool loops, excessive API requests, duplicate actions, and unbounded task execution.

Set sensible limits on tool calls per task and maximum execution duration.

Keep tool output separate from AI-generated text. Never interpret untrusted webpage content, document content, or tool results as privileged instructions.

8. File creation

Implement real TXT, Markdown, and PDF generation.

Expose the following tools:

- "create_text_file"
- "create_markdown_file"
- "create_pdf_file"
- "list_app_generated_files"
- "open_generated_file"

For each file-creation request:

1. Determine the content and filename.
2. Validate the filename and remove path traversal components.
3. Show a preview or summary for substantial documents.
4. Use the Android document picker or an explicit app-controlled storage destination.
5. Write the actual file.
6. Verify that writing completed successfully.
7. Record the file URI, filename, MIME type, creation time, and result.
8. Return a working open/share action.

Use Storage Access Framework APIs such as "ACTION_CREATE_DOCUMENT" for user-selected save locations. Do not request broad storage access unnecessarily.

For PDFs:

- Generate readable layouts with proper margins.
- Handle multiple pages.
- Prevent clipped or missing text.
- Include page numbers where appropriate.
- Use suitable fonts.
- Handle long paragraphs and Unicode text.
- Close and release PDF resources reliably.

Do not display a fake success message before file writing completes.

The AI should be able to fulfil instructions such as:

"Create a Class 10 revision plan in Markdown and save a printable PDF copy."

The application should generate both files, verify them, and provide access to the resulting documents.

9. Web research

Implement an actual research pipeline using a configurable search provider.

Separate:

- Search query creation.
- Search-provider requests.
- Result parsing.
- Page retrieval.
- Content extraction.
- Source attribution.
- AI summarisation.

Do not assume that an LLM can browse the web merely because it has an Internet connection.

Use documented search APIs or other permitted retrieval methods. If an external API key is required, explain this and allow a supported free alternative where available.

Requirements:

- Display source titles and clickable URLs.
- Attribute factual claims to retrieved sources.
- Distinguish verified source information from the AI's interpretation.
- Report when sources could not be accessed.
- Handle consent, cookie walls, rate limits, and robots restrictions appropriately.
- Do not bypass logins, paywalls, access controls, or website restrictions.
- Treat retrieved pages as untrusted content.
- Never follow instructions found inside a webpage that attempt to override JARVIS's permissions or reveal credentials.

Provide a working tool such as "search_web" and a retrieval tool such as "fetch_public_page", with domain and URL validation.

When the search provider is not configured, tell the user that live research is unavailable instead of inventing sources.

10. App task management

Create a task manager that records:

- Task ID.
- User instruction.
- Task type.
- Creation timestamp.
- Current status.
- Progress.
- Tool calls.
- Required approvals.
- Result summary.
- Generated file references.
- Errors and cancellation status.

Suggested statuses:

- Queued.
- Running.
- Waiting for permission.
- Waiting for approval.
- Completed.
- Failed.
- Cancelled.

Implement the following in the initial version:

- File generation.
- Web research.
- Opening an approved generated file.
- Sharing a generated file through Android's share sheet.
- Reading and managing JARVIS's own conversation and task history.

Use idempotency keys for operations where repeated requests could create duplicates.

If the user says, "Make a PDF of that research," JARVIS should reuse the relevant research result when possible rather than starting the entire task again.

Do not automatically send emails, messages, or publish content. Those integrations require separate implementation and user confirmation.

11. Screen assistance

Implement screen assistance as a permission-gated module.

The user must explicitly start a screen-sharing session and approve Android's system consent dialog.

Use MediaProjection and the required foreground-service declarations appropriate to the target Android version.

Requirements:

- Explain what will be captured and whether any content will leave the device.
- Request permission through the Android system flow.
- Display a persistent, visible sharing indicator.
- Provide an immediate stop-sharing control.
- Stop capture when the session ends or the user revokes access.
- Release projection, image, and encoder resources.
- Do not start screen capture silently in the background.
- Do not attempt to bypass protected or secure screen restrictions.
- Do not upload screenshots without the user's informed approval.

Initially support a consent-based capture workflow:

1. User requests screen help.
2. JARVIS explains the capture and processing options.
3. User grants screen-capture permission.
4. The application captures a frame or selected sequence.
5. The user confirms whether the captured content can be processed locally or sent to a configured AI provider.
6. The assistant analyses the permitted content.
7. The result is spoken or displayed.

If image understanding is unavailable through the selected AI provider, report that limitation. Do not pretend that screenshots have been analysed.

Treat continuous screen streaming and real-time visual assistance as later milestones unless a working implementation is available.

12. System-wide Android integrations

Design an extensible integration framework but do not pretend Android grants unrestricted device control.

Implement only integrations that are available through documented Android APIs and explicit permissions.

Possible future integrations include:

- Opening approved apps using intents.
- Launching Android settings pages.
- Creating reminders through a supported integration.
- Sharing files through Android's share sheet.
- Opening supported URLs.
- Reading user-selected documents.
- Creating home-screen shortcuts.
- Integrating with supported calendar or productivity providers.

For every integration, document:

- Required permissions.
- Supported Android versions.
- User-visible consent flow.
- What data is accessed.
- What actions can be performed.
- How the integration can be disabled.

AccessibilityService-based automation, if considered later, must be optional, user-enabled through Android Settings, transparent about the capabilities it grants, and restricted to legitimate user-directed accessibility or automation workflows. Never enable it silently or use it to bypass security restrictions.

Do not promise arbitrary control over every application, notification, message, or protected system screen.

Use explicit confirmations for consequential actions, including deletion, sending communications, modifying important settings, or sharing private information.

13. Privacy and security

Implement the following safeguards:

- Request permissions only when a feature needs them.
- Explain microphone and screen-capture use.
- Provide a visible microphone and sharing status.
- Offer controls to delete conversation and task history.
- Redact credentials from logs.
- Avoid collecting unnecessary personal information.
- Keep app-generated files within user-approved storage.
- Validate all external URLs and tool arguments.
- Protect against prompt injection from retrieved content.
- Prevent untrusted AI output from directly becoming executable code.
- Never bypass Android's permission dialogs.
- Never silently capture audio, video, or screen content.
- Do not transmit private content without appropriate consent.
- Add a privacy screen explaining which information remains local and which is transmitted to configured AI or search providers.
14. User interface and design
Use a polished, modern interface with a dark theme option, clear typography, accessible contrast, and restrained animations.
Required screens:
Welcome and setup.
Main JARVIS contact screen.
Active voice call.
Conversation transcript.
Task list and task details.
Generated files.
Research results.
Provider and model settings.
API-key configuration.
Permission explanations.
Screen-assistance controls.
Privacy and data management.
Diagnostics and implementation status.
The interface must support loading, empty, permission-denied, offline, API-limit, and error states.
Never leave a button decorative. Every enabled button must perform a real action or be explicitly marked as unavailable.
15. Testing and acceptance criteria
Create unit tests, integration tests, and instrumentation tests where appropriate.
Voice
Permission denial produces a clear explanation.
Speech input produces a real transcript.
A response is spoken using the configured speech engine.
Interrupting speech stops the previous output.
Ending the call releases audio resources.
Old responses are not spoken after a newer request begins.
Offline and API failure states are handled correctly.
AI and tools
A valid tool request executes the intended operation.
Invalid tool arguments are rejected.
Unknown tools are rejected.
An action requiring approval cannot execute before approval.
A failed operation is never reported as successful.
Duplicate requests do not accidentally repeat sensitive operations.
Rate limits are handled without infinite retries.
Files
TXT, Markdown, and PDF files are actually written.
Generated documents contain the expected text.
Long documents render across multiple pages.
Unicode text is preserved as far as the selected font supports it.
Invalid filenames are safely handled.
Permission denial or a failed write is reported honestly.
Files can be opened or shared through supported Android APIs.
Research
Real search results are displayed.
Each result includes its source URL.
Unavailable sources are reported accurately.
Unconfigured search providers do not produce fabricated results.
Untrusted page instructions cannot invoke privileged tools.
Screen assistance
Capture cannot begin before system consent.
The user can stop capture immediately.
Resources are released after stopping.
Remote processing requires appropriate consent.
Unsupported capture or analysis is reported honestly.
Build and delivery
Gradle compilation succeeds.
The app launches on a compatible Android device or emulator.
No critical crashes occur during the main workflows.
A debug APK is generated if the environment supports Android builds.
Installation and testing instructions are documented.
Build errors are fixed rather than hidden.
Any feature not verified on a real device is labelled accordingly.
16. Implementation milestones
Build in the following order.
Milestone 1: Native app foundation
Create the Android project, Compose navigation, JARVIS home screen, call UI, settings, and task-history structure. Ensure the project compiles.
Milestone 2: Real voice loop
Implement microphone permission, speech input, AI response generation, spoken output, interruption, and call termination.
Milestone 3: Secure AI configuration
Implement provider configuration, user-supplied credentials, safe networking, timeouts, and rate-limit handling.
Milestone 4: Real agent tools
Implement the tool registry, argument validation, permission checks, approval flow, execution records, and cancellation.
Milestone 5: File generation
Implement real TXT, Markdown, and PDF creation with user-selected destinations and working file-opening actions.
Milestone 6: Web research
Implement actual search, source retrieval, source attribution, summarisation, and failure handling.
Milestone 7: Screen assistance
Implement MediaProjection consent, the sharing indicator, a working capture flow, and appropriate cleanup.
Milestone 8: Supported Android integrations
Implement safe app launching, sharing, shortcuts, and other selected integrations.
Milestone 9: Verification and delivery
Run the available tests, fix failures, generate the APK where possible, and finish the documentation.
At the end of each milestone, update IMPLEMENTATION_STATUS.md and run the relevant tests.
17. Final deliverables
Provide:
Complete Android Studio project.
Kotlin source code.
Gradle build configuration.
Manifest and permission declarations.
AI provider abstraction and working provider integration.
Voice interaction implementation.
Real file-generation implementation.
Working task-execution engine.
Configurable research integration.
Screen-assistance module with explicit consent.
Unit and instrumentation tests.
README.md with setup and build instructions.
SECURITY.md explaining API-key handling and permissions.
ARCHITECTURE.md describing the application modules.
TESTING.md documenting tested and untested behaviour.
Debug APK, if the build environment supports generating one.
A list of required user-supplied API keys, if any.
A list of external services that may impose usage limits or charges.
Do not include real API keys in the deliverables.
18. Completion report
When implementation is finished, report:
What was built.
Which features work end-to-end.
Which features are partial or future work.
Which tests passed and failed.
Whether the APK was successfully generated.
How to install and run the application.
How to configure a free-tier provider.
What remains necessary for real-time WebRTC calls or broader system integrations.
Any expected external service limitations or possible costs.
Final requirement: Prioritise a smaller application that genuinely listens, responds, creates files, and executes validated tasks over a large application whose features only appear to work. Do not mark a feature complete until its actual execution path has been implemented and verified.
