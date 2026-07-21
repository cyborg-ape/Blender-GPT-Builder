# Blender GPT Builder — Installation Guide

Blender GPT Builder adds an OpenAI-powered chat and object-generation panel to
Blender's 3D Viewport.

## Requirements

- Blender 4.0 or newer
- An internet connection
- An OpenAI API key with access to a compatible GPT model

OpenAI API usage is associated with your OpenAI API account. Keep your API key
private and never include it in a `.blend` file, screenshot, or shared ZIP.

## Install the add-on

1. Download `blender_gpt_builder-1.0.8.zip`.
2. Do **not** extract the ZIP file.
3. Open Blender.
4. Go to **Edit > Preferences**.
5. Open **Add-ons**.
6. Click **Install...**.
7. Select `blender_gpt_builder-1.0.8.zip`.
8. Search for **Blender GPT Builder** and enable its checkbox if Blender does
   not enable it automatically.
9. Close Preferences. Blender normally saves this setting automatically; if
   your version shows **Save Preferences**, click it.

## Connect an OpenAI API key

1. Create or copy an API key from
   [OpenAI API keys](https://platform.openai.com/api-keys).
2. In Blender, go to **Edit > Preferences > Add-ons**.
3. Find and expand **Blender GPT Builder**.
4. Paste the key into **OpenAI API Key**.
5. Click **Save clipboard**.
6. Click ** Save**.

For a long `sk-proj-...` key, copy the complete key to your clipboard and click
**Save Clipboard**. This avoids truncation by Blender's text field.

The add-on stores a saved key in Blender's local configuration directory and
uses owner-only file permissions where the operating system supports them. For
stronger isolation, launch Blender with the `OPENAI_API_KEY` environment
variable instead of saving the key in the add-on.

## Verify the connection

1. Open a 3D Viewport.
2. Press **N** to show the sidebar.
3. Select the **GPT Builder** tab.
4. Expand **Connection** and click **Test**.
5. A successful test confirms that Blender can reach the OpenAI API and that
   the key is accepted.

You can then enter a prompt and choose:

- **Ask** for Blender help or an explanation.
- **Generate** to create and run Blender Python code for the requested scene
  changes.

## Safety notice

The **Generate** action automatically executes generated Python code with full
access to Blender and to the files and permissions available to the Blender
process. Save your `.blend` file first, keep backups, and use Generate only with
prompts and files you trust.

## Update the add-on

1. Download the newer `blender_gpt_builder-<version>.zip` file.
2. Open **Edit > Preferences > Add-ons**.
3. Disable and remove the existing **Blender GPT Builder** installation.
4. Install the new ZIP using the installation steps above.
5. Re-enable the add-on and click **Test** under **Connection**.

Restart Blender if the old version number remains visible after updating.

## Troubleshooting

### The GPT Builder tab is missing

- Confirm that **Blender GPT Builder** is enabled under Preferences > Add-ons.
- Make sure you are looking at a **3D Viewport**, press **N**, and select the
  **GPT Builder** tab.
- Restart Blender after installation.

### Authentication failed or HTTP 401 appears

- Copy a new, complete API key and save it again.
- Do not paste a masked key containing `*` characters.
- For long project keys, use **Save Clipboard**.
- Confirm that the API account is active and configured for API usage.

### The requested model is unavailable

- Expand **Model & Quality** and click **Refresh**.
- Select a model available to your API project, then click **Save**.

### Requests time out

- Confirm that Blender is allowed through your firewall or proxy.
- Increase **Response Timeout** under **Model & Quality**.
- Try a shorter prompt or a lower reasoning setting.

### Remove the saved API key

Open the add-on preferences and click **Remove**. This deletes the key from the
add-on's local settings file.

