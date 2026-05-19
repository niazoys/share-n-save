Topic-wise Research Reading List App

Files:
- index.html: the browsable list with Add item modal, filters, search, export/import, and Android share-target handling.
- manifest.webmanifest: PWA metadata and Web Share Target configuration.
- sw.js: service worker needed for install/offline behavior.
- icon.svg: app icon.

How to use locally:
1. Open index.html in a browser.
2. Click “+ Add item”.
3. Choose Category and Subcategory.
4. Save.
5. Use Export to back up added items.

How to make Android Share from X/LinkedIn work:
1. Host this whole folder over HTTPS. GitHub Pages, Netlify, Vercel, or your university webspace works.
2. Open the hosted URL in Chrome on Android.
3. Use Chrome menu → Add to Home screen / Install app.
4. Open X or LinkedIn, tap Share on a post, and choose “Research List”.
5. The add-item window opens with the shared link/text prefilled. Choose category/subcategory and save.

Important:
- Android share-target cannot work from a plain downloaded file:// HTML page.
- Custom additions are saved in that browser/device’s localStorage, so use Export for backup.
