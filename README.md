# WinTool Plugin Development Guide

Welcome, developer! This guide will walk you through creating your own plugins to extend the functionality of WinTool. The plugin system is designed to be simple, allowing you to build powerful system utilities using HTML, CSS, and JavaScript.

## Getting Started: Your First Plugin

Creating a plugin is straightforward. Here’s a quick overview of the process:

1.  **Create a Folder:** Add a new folder for your plugin inside this `src/plugins/` directory. The folder name will be your plugin's unique ID (e.g., `my-cool-plugin`).
2.  **Create the Manifest:** Inside your new folder, create a `plugin.json` file. This file tells WinTool about your plugin.
3.  **Build the UI:** Create an `index.html`, `styles.css`, and `script.js` file to define your plugin's user interface and logic.
4.  **Test:** Run WinTool. It will automatically discover and load your new plugin.

## Plugin Structure

Your plugin's folder must contain a few key files. Let's use the `system-uptime` plugin as an example structure:

```
/src/plugins/
└── system-uptime/
    ├── plugin.json     // (Required) Manifest file with metadata about your plugin.
    ├── index.html      // (Required) The HTML that defines the plugin's UI.
    ├── styles.css      // (Optional) The CSS for styling your plugin's UI.
    └── script.js       // (Optional) The JavaScript for your plugin's logic.
```

### The Manifest: `plugin.json`

This is the most important file. It tells WinTool how to load your plugin.

```json
{
  "name": "System Uptime",
  "version": "1.0.0",
  "description": "Displays how long the system has been running since the last restart.",
  "author": "Your Name",
  "main": "index.html",
  "icon": "fas fa-clock"
}
```

**Manifest Fields:**

*   `name` (string, required): The human-readable name of your plugin, which will appear in the sidebar and the Plugins management tab.
*   `version` (string, required): The version number of your plugin (e.g., `"1.0.0"`).
*   `description` (string, optional): A brief description of what your plugin does.
*   `author` (string, optional): Your name or username.
*   `main` (string, required): The entry point for your plugin's UI. This should almost always be `"index.html"`.
*   `icon` (string, required): The [Font Awesome](https://fontawesome.com/v5/search?m=free) icon class to display next to your plugin's name in the sidebar (e.g., `"fas fa-cogs"`).

## The `wintoolAPI` Bridge

For security, plugins run in a sandboxed environment and cannot access Node.js or Electron directly. To interact with the system, you must use the secure `wintoolAPI` object exposed to the window.

Here are the currently available functions:

### `wintoolAPI.getSystemInfo(type)`

Fetches specific categories of system information. This is an asynchronous function that returns a Promise.

*   **`type`** (string): The category of information you want. Examples include:
    *   `'time'`: Returns an object with system time details, including `uptime`.
    *   `'cpu'`: Returns detailed CPU information.
    *   `'mem'`: Returns memory usage details.
    *   *For a full list of available types, refer to the [systeminformation](https://systeminformation.io/docs/api.html) library documentation.*

**Example:**
```javascript
// In your plugin's script.js
window.wintoolAPI.getSystemInfo('time')
    .then(time => {
        const uptimeInSeconds = time.uptime;
        console.log(`System has been up for ${uptimeInSeconds} seconds.`);
    })
    .catch(err => {
        console.error("Failed to get system time:", err);
    });
```

### `wintoolAPI.runPluginScript(pluginId, scriptPath)`

Securely executes a PowerShell script that is bundled *inside your plugin's folder*. This is the recommended way to perform custom system actions.

*   **`pluginId`** (string): Your plugin's folder name (its unique ID). This is required for security to ensure you aren't trying to run scripts from other plugins.
*   **`scriptPath`** (string): The relative path to the script inside your plugin's folder (e.g., `"scripts/my-script.ps1"`). Path traversal (`../`) is not allowed.

**Example:**
Imagine your plugin has a script at `my-cool-plugin/scripts/get-processes.ps1`.

```javascript
// In your plugin's script.js
window.wintoolAPI.runPluginScript('my-cool-plugin', 'scripts/get-processes.ps1')
    .then(output => {
        console.log("Script output:", output);
    })
    .catch(err => {
        console.error("Script execution failed:", err);
    });
```

### `wintoolAPI.showNotification({ title, body, type })`

Displays a native-style notification in the corner of the screen.

*   **`options`** (object): An object containing the notification details.
    *   `title` (string): The title of the notification (currently unused in UI, but good practice).
    *   `body` (string): The main content of the notification.
    *   `type` (string, optional): The style of the notification. Can be `'info'`, `'success'`, `'warning'`, or `'error'`. Defaults to `'info'`.

**Example:**
```javascript
window.wintoolAPI.showNotification({
    title: 'Scan Complete',
    body: 'Your scan finished successfully!',
    type: 'success'
});
```

### `wintoolAPI.storage.set(pluginId, key, value)` and `wintoolAPI.storage.get(pluginId, key)`

Provides a simple and secure way for your plugin to save and retrieve its own settings or data. The storage is automatically namespaced to your `pluginId` to prevent conflicts with other plugins.

**Example:**
```javascript
const pluginId = 'my-cool-plugin'; // Your plugin's folder name

// Save a setting
async function saveUserPreference(theme) {
    await window.wintoolAPI.storage.set(pluginId, 'userTheme', theme);
}

// Retrieve a setting
async function loadUserPreference() {
    const savedTheme = await window.wintoolAPI.storage.get(pluginId, 'userTheme');
    if (savedTheme) {
        // Apply the theme
    }
}
```

### `wintoolAPI.dialog.showOpenDialog(options)` and `wintoolAPI.dialog.showSaveDialog(options, content)`

Allows your plugin to securely interact with the user's file system by opening the native open/save dialogs.

**Example: Open a text file**
```javascript
async function openConfigFile() {
    const options = {
        title: 'Open Configuration File',
        filters: [{ name: 'JSON Files', extensions: ['json'] }]
    };
    const result = await window.wintoolAPI.dialog.showOpenDialog(options);
    if (!result.canceled && result.file) {
        console.log('File path:', result.file.path);
        console.log('File content:', result.file.content);
        // Do something with the file content
    }
}
```

**Example: Save a text file**
```javascript
async function saveConfigFile(settingsObject) {
    const options = {
        title: 'Save Configuration File',
        defaultPath: 'my-settings.json',
        filters: [{ name: 'JSON Files', extensions: ['json'] }]
    };
    const content = JSON.stringify(settingsObject, null, 2);
        const result = await window.wintoolAPI.dialog.showSaveDialog(options, content);
    if (!result.canceled && result.path) {
        console.log('File saved to:', result.path);
    }
}
```

## Advanced: Creating a Plugin Backend

For plugins that require access to Node.js modules (like `fs` or `axios`) or need to perform heavy, persistent tasks, you can create a `backend.js` script. This script runs in Electron's main process and communicates securely with your frontend UI.

**Structure:**
```
/my-advanced-plugin/
├── backend.js      <-- Your new backend script
├── plugin.json
├── index.html
└── script.js
```

### Using Node.js Dependencies in Your Backend

Your plugin's backend can have its own Node.js dependencies. WinTool will automatically install them for you.

1.  **Create a `package.json`:** In your plugin's root folder, create a `package.json` file and list your dependencies there.

    **Example `package.json`:**
    ```json
    {
      "name": "my-advanced-plugin",
      "version": "1.0.0",
      "description": "Dependencies for my advanced plugin.",
      "dependencies": {
        "axios": "^1.10.0"
      }
    }
    ```

2.  **Require Dependencies in `backend.js`:** WinTool provides a secure `backendApi.require` function to load your plugin's dependencies. This ensures that your plugin can only access modules from its own `node_modules` folder.

    **Example `backend.js`:**
    ```javascript
    // my-advanced-plugin/backend.js

    function initialize(backendApi) {
        console.log('Initializing my advanced plugin backend!');

        // Load the 'axios' library from this plugin's node_modules folder
        const axios = backendApi.require('axios');

        // Register a handler that uses the library
        backendApi.registerHandler('fetch-data', async (url) => {
            try {
                const response = await axios.get(url);
                return response.data;
            } catch (error) {
                console.error('API request failed:', error);
                throw new Error(error.message);
            }
        });
    }

    module.exports = { initialize };
    ```
When WinTool starts, it will detect your plugin's `package.json` and automatically run `npm install` inside your plugin's folder if it hasn't been run before. This makes your plugin self-contained and easy to distribute.

### Calling Your Backend from Your Frontend

Use the `wintoolAPI.invoke()` function in your frontend `script.js` to call the handlers you registered in your backend.

**Example `script.js`:**
```javascript
// my-advanced-plugin/script.js
const pluginId = 'my-advanced-plugin';

async function onButtonClick() {
    try {
        const data = await window.wintoolAPI.invoke(pluginId, 'fetch-data', 'https://api.github.com/users/MTech-CMD');
        console.log('Fetched data:', data);
    } catch (error) {
        console.error('Backend invocation failed:', error);
    }
}
```

## Using Third-Party Libraries (Frontend)

Your plugin can use third-party **frontend** libraries (e.g., for charts, animations, date formatting). Because plugins are sandboxed for security, you must include them directly.

### Method 1: CDN (Easiest)

If the library is hosted on a CDN, you can add it via a `<script>` tag in your plugin's `index.html`. This requires an internet connection.

**Example:**
```html
<!-- In index.html -->
<head>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
```

### Method 2: Local Files (Offline Support)

For offline support, download the library's JavaScript file, place it in your plugin's folder, and reference it using a relative path.

**Example:**
```
/my-plugin/
├── index.html
├── script.js
└── lib/chart.min.js
```
```html
<!-- In index.html -->
<head>
    <script src="./lib/chart.min.js"></script>
</head>
```

### `wintoolAPI.tabs.on(eventName, callback)` and `wintoolAPI.tabs.off(eventName, callback)`

Allows your plugin to react to events happening in the main application UI, making it more efficient and context-aware.

*   **`eventName`** (string): Currently, the only supported event is `'tab-switched'`.
*   **`callback`** (function): A function that will be executed when the event fires. The function will receive an `event` object, and the relevant data will be in `event.detail`.

**Example: Pause background tasks when your plugin is not visible**
```javascript
const pluginId = 'my-cool-plugin';
let myIntervalId = null;

function handleTabSwitch(event) {
    const { newTabId, previousTabId } = event.detail;

    // Check if the user is navigating away from our plugin
    if (previousTabId === pluginId && newTabId !== pluginId) {
        console.log('Pausing background tasks.');
        clearInterval(myIntervalId);
        myIntervalId = null;
    }
    // Check if the user is navigating back to our plugin
    else if (newTabId === pluginId && previousTabId !== pluginId) {
        console.log('Resuming background tasks.');
        myIntervalId = setInterval(() => {
            // Do some work...
        }, 5000);
    }
}

// Start listening
window.wintoolAPI.tabs.on('tab-switched', handleTabSwitch);

// It's also good practice to have a way to clean up the listener if your plugin has a "disable" or "destroy" function.
function cleanup() {
    window.wintoolAPI.tabs.off('tab-switched', handleTabSwitch);
    clearInterval(myIntervalId);
}
```

## Development Workflow & Best Practices

1.  **Standard Layout:** Always create a standard tab header in your `index.html` to match the look and feel of the built-in tabs.
    ```html
    <div class="tab-header">
        <h1><i class="fas fa-your-icon"></i> Your Plugin Name</h1>
        <p>A short description of your plugin.</p>
    </div>
    ```
2.  **Styling:** Use the application's CSS variables in your `styles.css` to ensure your plugin's UI is consistent with the user's selected theme.
    *   `var(--background-card)`: Background for card-like elements.
    *   `var(--primary-color)`: The main accent color.
    *   `var(--border-color)`: For borders and dividers.
3.  **Asynchronous Operations:** All `wintoolAPI` calls are asynchronous. Always use `.then()/.catch()` or `async/await` to handle them properly.
4.  **Error Handling:** Your plugin should gracefully handle cases where API calls fail. Display a user-friendly error message instead of leaving the UI in a broken state.
5.  **Signaling Readiness:** To ensure WinTool's loading progress bar is accurate, signal that your plugin has finished initializing by calling `window.markTabAsReady('your-plugin-id')` at the end of your main script logic.

## Distributing Your Plugin

To share your plugin with others, simply zip the **contents** of your plugin's folder. The `plugin.json` file should be at the root of the zip archive.

**Correct Structure for `my-cool-plugin.zip`:**
```
my-cool-plugin.zip
├── plugin.json
├── index.html
├── styles.css
└── script.js
```

A user can then install it using the "Install Plugin from File" button in WinTool's "Plugins" tab.

Happy coding!
