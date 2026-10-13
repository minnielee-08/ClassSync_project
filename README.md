# ClassSync

ClassSync is a Chrome extension that helps students sync upcoming Canvas assignments with their personal calendar workflow. The extension reads assignment events from the Western Washington University Canvas API, shows those events in the extension popup, and can export them as an `.ics` calendar file.

The repository also includes a small instructional website that explains how to install and use the extension.


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

