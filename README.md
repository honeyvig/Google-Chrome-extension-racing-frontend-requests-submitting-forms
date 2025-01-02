# Google-Chrome-extension-racing-frontend-requests-submitting-forms
Creating a Chrome extension that automates tasks like tracing frontend requests and submitting forms requires a combination of Chrome extension APIs, JavaScript, and content scripts. Below is a step-by-step guide for building a Chrome extension that traces frontend requests (network requests) and automatically submits forms based on user configuration.
Objective:

    Trace frontend network requests: Monitor outgoing and incoming requests made by the web page.
    Automatically submit forms: Fill and submit forms with user-specified data.

Chrome Extension Structure

chrome-extension/
├── manifest.json
├── background.js
├── content.js
├── popup.html
├── popup.js
└── icons/
    └── icon.png

1. Manifest File: manifest.json

This file defines the extension's settings, permissions, and resources.

{
  "manifest_version": 3,
  "name": "Automate Requests & Form Submission",
  "description": "Automates frontend request tracing and form submission.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "webRequest",
    "webRequestBlocking",
    "storage",
    "https://*/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_end"
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon.png",
      "48": "icons/icon.png",
      "128": "icons/icon.png"
    }
  }
}

2. Background Script: background.js

The background script will listen for network requests and form submissions.

// background.js

// Listener to trace network requests
chrome.webRequest.onBeforeRequest.addListener(
  function (details) {
    console.log("Request URL: ", details.url);
    // Handle the request URL, maybe save data or track specific requests
  },
  { urls: ["<all_urls>"] },  // Listen to all URLs (can be customized for specific URLs)
  ["blocking"]
);

// Listening for requests to send custom data or form submissions
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "submitForm") {
    // Simulate form submission
    const { formData, formAction } = message;
    const form = new FormData();
    Object.keys(formData).forEach(key => form.append(key, formData[key]));

    fetch(formAction, {
      method: 'POST',
      body: form
    })
      .then(response => response.json())
      .then(data => {
        sendResponse({ success: true, data });
      })
      .catch(error => {
        sendResponse({ success: false, error: error });
      });

    return true;  // Keep the response open for async
  }
});

3. Content Script: content.js

This script interacts with the webpage to trace requests and automate form submissions.

// content.js

// Function to trace frontend requests (when a user clicks a button or initiates an action)
function traceRequest(url) {
  console.log("Frontend Request Traced: ", url);
  // You can also send data to the background script here
  chrome.runtime.sendMessage({
    action: "traceRequest",
    url: url
  });
}

// Automatically fill and submit forms (example)
function autoSubmitForm() {
  const form = document.querySelector('form'); // You can specify the form here
  if (form) {
    const formData = new FormData(form);
    const formAction = form.action;

    // Automatically fill the form (you can configure this part with custom data)
    formData.set('username', 'testUser');
    formData.set('password', 'testPassword');

    // Send the form data to background.js for submission
    chrome.runtime.sendMessage({
      action: "submitForm",
      formData: Object.fromEntries(formData.entries()),
      formAction: formAction
    });
  }
}

// Listen for specific events to trigger actions
document.addEventListener('DOMContentLoaded', function () {
  traceRequest(window.location.href);  // Trace the current URL
  autoSubmitForm();  // Automatically submit the first form on the page
});

4. Popup HTML: popup.html

This file will allow users to interact with the extension to enable/disable the automation.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Automate Requests & Form Submission</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      width: 300px;
      padding: 10px;
    }
    h3 {
      margin-bottom: 10px;
    }
    button {
      padding: 10px;
      font-size: 14px;
      cursor: pointer;
      background-color: #4CAF50;
      color: white;
      border: none;
    }
    button:hover {
      background-color: #45a049;
    }
  </style>
</head>
<body>
  <h3>Automate Tasks</h3>
  <button id="enableAutomation">Enable Automation</button>
  <button id="disableAutomation">Disable Automation</button>

  <script src="popup.js"></script>
</body>
</html>

5. Popup Script: popup.js

This script handles interaction with the popup UI, allowing users to enable or disable the automation features.

// popup.js

document.getElementById('enableAutomation').addEventListener('click', function () {
  chrome.storage.sync.set({ automationEnabled: true }, function () {
    alert("Automation enabled!");
  });
});

document.getElementById('disableAutomation').addEventListener('click', function () {
  chrome.storage.sync.set({ automationEnabled: false }, function () {
    alert("Automation disabled!");
  });
});

6. Icons Folder:

For simplicity, you can create a folder named icons and place an icon image in it (icon.png). This icon will represent your extension in the browser toolbar.
7. Testing the Extension

To test the extension:

    Install the Extension:
        Go to chrome://extensions/.
        Enable Developer mode.
        Click Load unpacked and select the extension's directory.
    Use the Extension:
        Navigate to a website that has a form.
        Click on the extension icon and enable automation.
        The form should be filled automatically, and network requests should be traced.

Conclusion

This Chrome extension traces frontend requests (like network calls made from the webpage) and automatically submits forms based on the data you configure. The background script handles both tasks, while the content script interacts with the page. You can expand this extension by adding more sophisticated form-filling logic, dynamic request monitoring, and more.
