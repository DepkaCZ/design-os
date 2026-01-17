# Screenshot Screen Design

You are helping the user capture a screenshot of a screen design they've created. The screenshot will be saved to the product folder for documentation purposes.

## Prerequisites: Check for Playwright MCP

Before proceeding, verify that you have access to the Playwright MCP tool. Look for a tool named `browser_take_screenshot` or `mcp__playwright__browser_take_screenshot`.

If the Playwright MCP tool is not available, output this EXACT message to the user (copy it verbatim, do not modify or "correct" it):

---
To capture screenshots, I need the Playwright MCP server installed. Please run:

```
claude mcp add playwright npx @playwright/mcp@latest
```

Then restart this Claude Code session and run `/screenshot-design` again.
---

Do not substitute different package names or modify the command. Output it exactly as written above.

Do not proceed with the rest of this command if Playwright MCP is not available.

## Step 1: Identify the Screen Design

First, determine which screen design to screenshot.

Read `/product/product-roadmap.md` to get the list of available sections, then check `src/sections/` to see what screen designs exist.

If only one screen design exists across all sections, auto-select it.

If multiple screen designs exist, use the AskUserQuestion tool to ask which one to screenshot:

"Which screen design would you like to screenshot?"

Present the available screen designs as options, grouped by section:
- [Section Name] / [ScreenDesignName]
- [Section Name] / [ScreenDesignName]

## Step 2: Start the Dev Server

Start the dev server yourself using Bash. Run `npm run dev` in the background so you can continue with the screenshot capture.

Do NOT ask the user if the server is running or tell them to start it. You must start it yourself.

After starting the server, wait a few seconds for it to be ready before navigating to the screen design URL.

## Step 3: Capture the Screenshot

Use the Playwright MCP tool to navigate to the screen design and capture a screenshot.

The screen design URL pattern is: `http://localhost:3000/sections/[section-id]/screen-designs/[screen-design-name]/fullscreen`

Note: Use the `/fullscreen` suffix to get the screen design with the app shell but without the Design OS navigation chrome.

### Screenshot Capture Process

1. **Navigate to the screen design URL** using `browser_navigate`

2. **Wait for page to load** - wait 2 seconds after navigation for content to render

3. **CRITICAL: Hide the scrollbar before capturing**

   The browser reserves space for the scrollbar (typically 15px) which can cause content to appear cut off on the right edge. Before each screenshot, inject CSS to hide the scrollbar:

   ```javascript
   // Use browser_evaluate to inject this script
   () => {
     let style = document.getElementById('screenshot-scrollbar-fix');
     if (!style) {
       style = document.createElement('style');
       style.id = 'screenshot-scrollbar-fix';
       style.textContent = `
         html, body {
           scrollbar-width: none !important;
           -ms-overflow-style: none !important;
         }
         html::-webkit-scrollbar, body::-webkit-scrollbar {
           display: none !important;
           width: 0 !important;
           height: 0 !important;
         }
       `;
       document.head.appendChild(style);
     }
   }
   ```

4. **Set the theme (light/dark)** before capturing:

   ```javascript
   // For light mode:
   () => {
     localStorage.setItem('theme', 'light');
     document.documentElement.classList.remove('dark');
   }

   // For dark mode:
   () => {
     localStorage.setItem('theme', 'dark');
     document.documentElement.classList.add('dark');
   }
   ```

5. **Capture the screenshot** using `browser_take_screenshot` with `fullPage: true`

6. **Verify the screenshot** - check for visual issues like missing elements, text overlap, cut-off content on the right edge, and proper alignment

### Screenshot Specifications

**Viewport sizes:**
- Mobile: 375x812 (iPhone-sized)
- Desktop: 1280x800

**Capture all 4 variants for each screen:**
1. `[ScreenDesignName]-mobile-light.png`
2. `[ScreenDesignName]-mobile-dark.png`
3. `[ScreenDesignName]-desktop-light.png`
4. `[ScreenDesignName]-desktop-dark.png`

**Settings:**
- Use `fullPage: true` to capture the entire scrollable content
- PNG format for best quality
- Always inject the scrollbar-hide CSS before capturing

When using `browser_take_screenshot`, set `fullPage: true` to capture the entire page including content below the fold.

## Step 4: Save the Screenshot

The Playwright MCP tool can only save screenshots to its default output directory (`.playwright-mcp/`). You must save the screenshot there first, then copy it to the product folder.

1. **First**, use `browser_take_screenshot` with just a filename (no path):
   - Use a simple filename like `dashboard.png` or `invoice-list.png`
   - The file will be saved to `.playwright-mcp/[filename].png`

2. **Then**, copy the file to the product folder using Bash:
   ```bash
   cp .playwright-mcp/[filename].png product/sections/[section-id]/[filename].png
   ```

**Naming convention:** `[screen-design-name]-[variant].png`

Examples:
- `invoice-list.png` (main view)
- `invoice-list-dark.png` (dark mode variant)
- `invoice-detail.png`
- `invoice-form-empty.png` (empty state)

If the user wants both light and dark mode screenshots, capture both.

## Step 5: Confirm Completion

Let the user know:

"I've saved the screenshot to `product/sections/[section-id]/[filename].png`.

The screenshot captures the **[ScreenDesignName]** screen design for the **[Section Title]** section."

If they want additional screenshots (e.g., dark mode, different states):

"Would you like me to capture any additional screenshots? For example:
- Different states (empty, loading, etc.)"

## Important Notes

- If there are any dev servers started, kill them before capturing screenshots
- Start the dev server yourself - do not ask the user to do it
- Screenshots are saved to `product/sections/[section-id]/` alongside spec.md and data.json
- Use descriptive filenames that indicate the screen design and any variant (dark mode, mobile, etc.)
- Capture at a consistent viewport width for documentation consistency
- Always capture full page screenshots to include all scrollable content
- After you're done, you may kill the dev server if you started it
