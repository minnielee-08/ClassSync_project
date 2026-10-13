# ClassSync

ClassSync is a Chrome extension that helps students sync upcoming Canvas assignments with their personal calendar workflow. The extension reads assignment events from the Western Washington University Canvas API, shows those events in the extension popup, and can export them as an `.ics` calendar file.

The repository also includes a small instructional website that explains how to install and use the extension.

## Project Structure

```text
ClassSync-main/
+-- README.md
+-- extension/
|   +-- manifest.json
|   +-- package.json
|   +-- package-lock.json
|   +-- css/
|   |   +-- styles.css
|   +-- html/
|   |   +-- popup.html
|   +-- icons/
|   |   +-- icon16.png
|   |   +-- icon32.png
|   +-- js/
|       +-- background.js
|       +-- popup.js
+-- site/
    +-- assets/
    |   +-- icons/
    |   |   +-- icon16.png
    |   |   +-- icon32.png
    |   +-- videos/
    |       +-- classSyncDemo.mp4
    +-- css/
    |   +-- styles.css
    +-- html/
    |   +-- instructions.html
    +-- js/
        +-- tabbing.js
```

## How the Project Works

ClassSync has two main parts:

1. The `extension/` folder contains the Chrome extension that users install in their browser.
2. The `site/` folder contains a simple instructional webpage with written setup steps and a video demo.

The Chrome extension is the working product. When a user opens the extension popup, ClassSync checks whether a Canvas API token has already been saved. If not, it shows a token form. Once the token is saved, the popup shows controls for syncing Canvas calendar events and downloading those events as an `.ics` file.

When the user clicks the sync button, the popup sends a message to the extension background service worker. The background script uses the Canvas token to call the Canvas API. It first gets the current Canvas user, then gets the user's active courses, then builds a Canvas calendar events request for the user and those courses. The returned events are sent back to the popup, where they can be displayed or converted into calendar file content.

## Extension Setup

1. Clone or download this repository.
2. Open Chrome.
3. Go to `chrome://extensions/`.
4. Turn on Developer Mode.
5. Click **Load unpacked**.
6. Select the `extension/` folder from this project.
7. Go to Canvas settings and create a new Canvas API access token.
8. Open the ClassSync extension popup.
9. Paste the Canvas API token and click **Save Token**.
10. Use **Sync Calendar** to view events or **Download .ics** to export them.

## Canvas API Token

ClassSync needs a Canvas API token so it can request calendar and assignment data from Canvas on the user's behalf.

In the current code, the extension is configured for Western Washington University's Canvas domain:

```text
https://wwu.instructure.com
```

If this extension is adapted for another school, the Canvas URLs in the extension code and the host permissions in `manifest.json` must be updated.

## Extension Files

### `extension/manifest.json`

This is the Chrome extension manifest. It tells Chrome how the extension should be loaded and what permissions it needs.

Important sections:

- `manifest_version`: Uses Manifest V3.
- `name`, `version`, and `description`: Define the extension metadata shown by Chrome.
- `permissions`: Gives the extension access to Chrome identity, local storage, and the active tab.
- `host_permissions`: Allows network requests to the WWU Canvas API and Microsoft Graph.
- `background`: Registers `js/background.js` as the background service worker.
- `action`: Defines the popup page and toolbar icons.
- `icons`: Defines extension icons.
- `oauth2`: Contains a placeholder Microsoft client ID and a calendar scope.

The Microsoft Graph permission appears to be intended for future Outlook calendar integration. The current popup code exports `.ics` files but does not directly create Outlook calendar events through Microsoft Graph.

### `extension/html/popup.html`

This file defines the popup users see when they click the ClassSync extension icon.

It contains two main interface sections:

- `#token-form`: A form with a Canvas token input and save button.
- `#main-content`: The main extension controls shown after a token has been saved.

The main controls include:

- **Sync Calendar**: Fetches Canvas events and lists them in the popup.
- **Download .ics**: Fetches Canvas events and downloads them as a calendar file.
- `#event-list`: A list where synced Canvas events are displayed.

At the bottom, the file loads `../js/popup.js`, which gives the popup its behavior.

### `extension/css/styles.css`

This stylesheet controls the appearance of the extension popup.

It handles:

- Resetting default spacing with a universal selector.
- Setting the popup size, background color, font, and layout.
- Centering the token form and main content.
- Styling the Canvas token input.
- Styling the buttons used for saving, syncing, and downloading.
- Adding hover, focus, and disabled button states.
- Styling the event list.
- Customizing scrollbars for a cleaner popup appearance.

The popup uses a dark blue background with bright blue action buttons, matching the instructional site.

### `extension/js/popup.js`

This is the main client-side script for the extension popup.

It is responsible for:

- Waiting for the popup document to load.
- Checking Chrome local storage for an existing `canvasToken`.
- Showing the token form when no token is saved.
- Showing the main sync controls when a token exists.
- Saving the Canvas API token to `chrome.storage.local`.
- Handling the **Sync Calendar** button.
- Handling the **Download .ics** button.
- Asking the background service worker to fetch Canvas events.
- Displaying fetched events in the popup.
- Generating `.ics` calendar content.
- Downloading the generated calendar file.

Key functions:

- `fetchCanvasEventsFromToday()`: Reads the saved token and sends a message to `background.js` requesting Canvas events from the current date forward.
- `displayEvents(events)`: Clears the existing event list and renders each event title and start date.
- `generateICSContent(events)`: Converts Canvas event objects into iCalendar text.
- `decodeHTMLEntities(text)`: Converts encoded HTML entities back into readable text for event descriptions.
- `downloadICSFile(icsContent)`: Creates a downloadable calendar file named `canvas_events.ics`.

### `extension/js/background.js`

This is the extension background service worker. It performs the Canvas API calls on behalf of the popup.

It listens for messages from `popup.js`. When it receives a message with the action `fetchCanvasEvents`, it:

1. Reads the Canvas token sent from the popup.
2. Calls the Canvas `users/self` endpoint to get the current user's Canvas ID.
3. Calls the Canvas `courses` endpoint to get active courses.
4. Builds a Canvas calendar events URL using the user context and course contexts.
5. Requests assignment-type calendar events from the current date through three months in the future.
6. Sends the fetched event data back to the popup.

This file keeps the API-fetching logic separate from the popup interface. That separation is useful because Chrome extension popups are temporary; they open and close often, while the background service worker is the better place to coordinate extension-level work.

## Instruction Site Files

### `site/html/instructions.html`

This file is the standalone instructional webpage for ClassSync.

It includes:

- Basic HTML metadata and responsive viewport settings.
- Favicon links using the icons in `site/assets/icons/`.
- A link to `../css/styles.css`.
- A page title and visible `ClassSync` heading.
- A tab bar with two tabs:
  - **Written Instructions**
  - **Video Demonstration**
- A written instructions section with setup steps.
- A video demonstration section that plays `classSyncDemo.mp4`.
- A script tag that loads `../js/tabbing.js`.

The written instructions mirror the basic extension setup flow: clone the repository, load the unpacked extension, create a Canvas API token, and save that token in the extension.

### `site/css/styles.css`

This stylesheet controls the instructional website.

It handles:

- Resetting spacing and box sizing.
- Creating a centered vertical page layout.
- Applying the dark blue ClassSync background.
- Positioning the `ClassSync` title in the top-right corner.
- Styling links with a bright blue glow.
- Styling inline code snippets.
- Creating the tab layout.
- Showing the active content panel while hiding inactive panels.
- Styling the written instructions list.
- Creating a responsive video container with a 16:9 aspect ratio.
- Making the demo video fill its container.
- Customizing scrollbars.

The stylesheet works closely with `tabbing.js`: inactive content sections are hidden with `display: none`, and the active content section becomes visible when the `active` class is applied.

### `site/js/tabbing.js`

This script powers the tabs on the instructional website.

When the page finishes loading, it:

1. Selects every element with the `.tab` class.
2. Selects every element with the `.content` class.
3. Adds a click listener to each tab.
4. Removes the `active` class from all tabs and content sections when a tab is clicked.
5. Adds the `active` class to the clicked tab.
6. Reads the clicked tab's `data-tab` value.
7. Finds the matching content section by ID.
8. Adds `active` to that content section so it becomes visible.

For example, the **Written Instructions** button has:

```html
data-tab="written-instructions"
```

That matches:

```html
<section id="written-instructions" class="content active">
```

This connection allows the script to switch between the written setup guide and the video demo without loading another page.

## Asset Files

### Icons

The project includes 16px and 32px PNG icons in both the extension and site folders:

- `extension/icons/icon16.png`
- `extension/icons/icon32.png`
- `site/assets/icons/icon16.png`
- `site/assets/icons/icon32.png`

The extension icons are used by Chrome for the toolbar action and extension metadata. The site icons are used as browser favicons for the instructions page.

### Demo Video

`site/assets/videos/classSyncDemo.mp4` is shown in the Video Demonstration tab on the instructional website.

## Data Flow

The main extension data flow looks like this:

```text
User opens popup
       |
       v
popup.js checks chrome.storage.local for canvasToken
       |
       v
User saves token or opens main controls
       |
       v
User clicks Sync Calendar or Download .ics
       |
       v
popup.js sends fetchCanvasEvents message to background.js
       |
       v
background.js calls Canvas API
       |
       v
Canvas returns assignment calendar events
       |
       v
background.js sends events back to popup.js
       |
       v
popup.js displays events or generates an .ics file
```

## Calendar Export

The `.ics` export is generated directly in `popup.js`.

For each Canvas event, ClassSync creates a `VEVENT` entry with:

- `UID`: The Canvas event ID.
- `SUMMARY`: The Canvas event title with `Due` appended.
- `DESCRIPTION`: The event description with HTML tags removed.
- `DTSTART`: The event start time.
- `DTEND`: The event end time.
- `STATUS`: Set to `CONFIRMED`.
- `LOCATION`: The Canvas event location, or `No location`.

The file is downloaded as:

```text
canvas_events.ics
```

This file can be imported into calendar applications that support iCalendar files.

## Notes for Developers

- The extension currently stores the Canvas token in Chrome local storage under the key `canvasToken`.
- The Canvas API domain is hard-coded for WWU.
- `background.js` fetches assignment calendar events for the next three months.
- The `package.json` includes the `ics` dependency, but the current code manually generates `.ics` text instead of importing that package.
- The Microsoft OAuth configuration in `manifest.json` contains a placeholder client ID. Direct Outlook calendar syncing would require a real Microsoft client ID and additional Microsoft Graph implementation.
- The extension uses Chrome extension APIs, so it should be tested in Chrome or another Chromium-based browser that supports Manifest V3 extensions.

## Possible Future Improvements

- Add a way to update or remove a saved Canvas token.
- Add loading states while events are being fetched.
- Improve error messages for expired or invalid Canvas tokens.
- Support Canvas instances outside WWU through configurable settings.
- Use Microsoft Graph to sync directly with Outlook instead of only exporting `.ics`.
- Use the installed `ics` package or remove it if manual `.ics` generation remains preferred.
- Add pagination handling if Canvas returns more events than fit in one response.
- Add tests for `.ics` generation and Canvas event formatting.
