# Blender GPT Builder

Blender GPT Builder is a Blender 4.x add-on that turns natural-language prompts
into practical Blender help or executable Blender Python. It brings an OpenAI
workflow directly into the 3D Viewport so a user can ask about the current scene,
generate geometry, and keep working while the model responds.

Current release: **1.0.8**

Installable package: [blender_gpt_builder-1.0.8.zip]

## What it does

- Adds a **GPT Builder** tab to the 3D Viewport sidebar.
- Uses the OpenAI Responses API for chat and Blender Python generation.
- Uses `gpt-5.6-sol` by default, with a model refresh/selection control.
- Exposes `low`, `medium`, `high`, and `xhigh` reasoning settings for GPT-5.6.
- Can include a compact summary of the active scene, selected objects, and mode.
- Runs network requests on a background thread so Blender stays responsive.
- Saves complete chat output and generated code in Blender text blocks.
- Creates an undo checkpoint before automatically running generated code.
- Uses only Python's standard library and Blender's built-in Python API; there is
  no separate server or Python package installation.

## Quick demo

After completing the setup below:

1. Open Blender's 3D Viewport and press **N**.
2. Select the **GPT Builder** tab.
3. Enter: `Create a double helix, or create a planet with rings, or create a medieval watchtower`
4. Leave **Include scene context** enabled.
5. Select **Medium** reasoning and click **Generate**.
6. Wait for the background request to finish. The generated script is saved to
   the `BlenderGPT_GeneratedCode` text block and then executed.
7. Press **Ctrl+Z** if you want to revert the generated scene changes.

For a non-destructive first test, use **Ask** with:

```text
Explain what is selected in my scene and suggest three ways to improve it.
```

The full answer is saved to the `BlenderGPT_Response` text block.

## Requirements

- Blender 4.0 or newer
- Internet access from Blender
- An OpenAI API key with access to a compatible GPT model

API usage is charged to the API account associated with the key. Never commit,
publish, or include an API key in a `.blend` file or distributable ZIP.

## Installation and setup

1. Download `blender_gpt_builder-1.0.8.zip`.
2. Do **not** extract the ZIP.
3. Open Blender and go to **Edit > Preferences > Add-ons**.
4. Click **Install...**, or open the upper-right menu and choose
   **Install from Disk**, depending on the Blender version.
5. Select `blender_gpt_builder-1.0.8.zip`.
6. Search for **Blender GPT Builder** and enable it if needed.
7. Create an API key at <https://platform.openai.com/api-keys>.
8. Expand the add-on's preferences and enter the key.
9. Click **Save Clipboard**. For a long `sk-proj-...` key, copy the complete key and click
   **Save** to bypass Blender's text-field length limit.
10. Return to the 3D Viewport, press **N**, and open **GPT Builder**.
11. Expand **Connection** and click **Test**.

See [`INSTALLATION.md`](INSTALLATION.md) for updating, troubleshooting, key
storage, and removal instructions.


## Running the project

There is no standalone web server or command-line service. The project runs
inside Blender:

1. Open any `.blend` file or a new default scene.
2. Open **3D Viewport > Sidebar (N) > GPT Builder**.
3. Under **Model & Quality**, choose a model, reasoning effort, and response
   timeout. The timeout accepts 30 to 3,600 seconds and defaults to 600.
4. Enter a short prompt in the panel. For a longer prompt, click **Text Editor**
   and edit the `BlenderGPT_Prompt` text block.
5. Choose an action:
   - **Ask** returns guidance without changing the scene.
   - **Generate** requests Blender Python, saves it, creates an undo checkpoint,
     and executes it automatically.
6. Read the short status in the panel. Open Blender's Text Editor for the full
   answer or generated source.

Only one OpenAI request runs at a time. Blender remains usable while the network
request is in progress, but generated code executes on Blender's main thread when
the response returns.


## How the system works

```text
Prompt + optional scene summary
              |
              v
  OpenAI Responses API (/v1/responses)
              |
       +------+------+
       |             |
      Ask         Generate
       |             |
       v             v
 Response text   Blender Python
       |             |
       v             v
 Blender text    Save source -> undo checkpoint -> execute
 block
```

The add-on sends requests with `urllib.request`, so it does not require the
OpenAI Python SDK inside Blender. Network work happens in a daemon thread. A
Blender timer polls job state and performs UI updates and Blender data changes
back on the main thread.

When scene context is enabled, the request includes:

- Scene name and object count
- Active object's name, type, transform, and scale
- Up to 20 selected objects with names, types, and locations
- Current Blender interaction mode

The current flow is intentionally stateless: each request receives the prompt
and current scene summary, but previous chat responses are not automatically
sent back to the model.

## How GPT-5.6 is used

GPT-5.6 is the runtime reasoning and code-generation engine, not just a tool used
to write the project.

- `gpt-5.6-sol` is the default model for quality-first Blender and Python tasks.
- **Ask** uses a Blender-assistant prompt that requests direct guidance and
  precise UI or `bpy` steps.
- **Generate** uses a separate deterministic-code prompt that requests raw
  Blender 4.x Python, preserves unrelated scene objects by default, and leaves
  the main result selected.
- The reasoning control lets the user trade latency and API usage for more model
  work on difficult geometry or scripting requests.
- The model list can be refreshed from the API to cache compatible GPT model
  IDs while retaining a compact fallback list.

The implementation uses OpenAI's recommended
[Responses API](https://developers.openai.com/api/docs/guides/migrate-to-responses)
and follows the current [GPT-5.6 model guidance](https://developers.openai.com/api/docs/guides/latest-model).

## How Codex accelerated development

Codex served as the main build enviroment for creation, debugging, and verification across
the add-on's refinement cycle. Its contribution was invaluable
where diagnosis required moving between Blender UI behavior, Python source,
packaging, and API behavior:

- Audited the original single-file add-on and proposed a reliability-focused
  refinement plan while preserving the requested one-file architecture.
- Refactored blocking API calls into background jobs while keeping Blender API
  work on the main thread.
- Improved prompt design, response parsing, error sanitization, status feedback,
  and the collapsible sidebar workflow.
- Built versioned ZIP and direct `.py` installers, then exercised install and
  enable flows with isolated Blender profiles rather than relying only on static
  Python checks.
- Diagnosed repeated HTTP 401 failures from screenshots and request behavior.
  Codex traced the failure to Blender truncating a long project key at 127
  characters, then implemented clipboard-based saving, authoritative local
  configuration, masked previews, and key fingerprints.
- Added configurable 30-to-3,600-second request timeouts after advanced reasoning
  requests exceeded the earlier fixed limit.
- Iterated the generated-code workflow through review mode, policy validation,
  automatic execution, undo support, and a runtime limit as product decisions
  changed.
- Packaged and integrity-checked each release, making rapid user-test-feedback
  loops possible without manually reconstructing the add-on archive.

In short, Codex compressed a sequence of code review, implementation, Blender
integration testing, log analysis, and release packaging into tight conversational
iterations. The product direction remained human-led; Codex made each decision
faster to implement and verify.

## Key product and engineering decisions

I made these decisions by testing the add-on in Blender and paying attention to
what felt useful, what slowed me down, and what made the setup harder than it
needed to be. Codex helped me implement and test the changes, but I made the
final product choices.

- **Keep the add-on simple to install.** I kept the runtime in a single
  `__init__.py` file so the add-on is easy to package, share, and inspect.
- **Avoid extra dependencies.** I used Python's built-in HTTP tools instead of
  requiring the OpenAI SDK inside Blender. Both **Ask** and **Generate** connect
  through the Responses API.
- **Keep Blender responsive.** Longer requests made Blender feel frozen during
  early tests, so I moved the network work off the main thread. Blender now
  remains usable while the add-on waits for a response.
- **Support both quick and complicated prompts.** I added four reasoning levels
  and an adjustable timeout from 30 to 3,600 seconds.
- **Save long API keys correctly.** Blender can cut off long project keys in its
  preference field. The saved configuration now takes priority, and
  **Save Clipboard** preserves the complete key.
- **Provide useful scene context.** The add-on sends a compact scene summary and
  information about up to 20 selected objects. This gives the model useful
  context without uploading the entire `.blend` file.
- **Make prompt-to-scene generation immediate.** I chose to run generated code
  automatically so I can describe an object, click **Generate**, and see the
  result in Blender without a separate script step.
- **Keep the interface focused.** I removed the reference-image controls from
  version 1.0.8 because they made the panel feel crowded. The final release
  focuses on text prompts and the scene already open in Blender.

Automatic execution was the biggest tradeoff I made. It creates a much faster
creative loop, but it also means model-written Python runs inside Blender. To
make that behavior clearer and easier to recover from, the add-on saves the
generated source, creates an undo checkpoint, and applies a runtime limit before
execution begins.


### Generated code

**Generate automatically executes model-produced Python.** The generation prompt
asks for Blender-focused code and the add-on applies a best-effort 15-second
Python tracing limit, but this is not a security sandbox. Generated code has the
permissions of the Blender process and can modify the open scene or access local
resources.

### API keys and scene data

- A saved key is written to Blender's local configuration directory with
  owner-only permissions where supported.
- `OPENAI_API_KEY` can be used instead of local storage.
- Error messages redact strings that resemble API keys.
- If scene context is enabled, the summarized scene/object metadata is sent to
  the OpenAI API with the prompt.


## Known limitations

- Generated code is automatically executed and is not sandboxed.
- Requests do not currently stream partial output.
- There is no automatic retry or exponential backoff for transient API errors.
- Chat history is not carried between requests.
- Model availability depends on the user's OpenAI API project.
- API latency and usage increase with reasoning effort and task complexity.

## Next steps

- Add an opt-in review-before-run mode without slowing the default demo path.
- Add retry/backoff and optional Responses API background processing.
- Add streaming status for long generation requests.
- Build a small Blender-native regression suite for registration, key handling,
  scene context, response parsing, and execution rollback.
- Add a permission profile or external worker process for stronger generated-code
  isolation.

## Author

Created by **MellowMonday**, with GPT-5.6 powering the in-Blender experience and
Codex used as the development and verification collaborator.
