Share n' Save

Files:
- index.html: the browsable list with Add item modal, filters, search, export/import, editable colored categories, and Android share-target handling.
- manifest.webmanifest: PWA metadata and Web Share Target configuration.
- sw.js: service worker needed for install/offline behavior.
- icon.svg: app icon.
- guide_for_sync_setup.md: Supabase setup for free automatic sync across devices.

How to use locally:
1. Open index.html in a browser.
2. Click "+ Add item".
3. Choose Category and Subcategory.
4. Save.
5. Use Sync for automatic cross-device sync or Export for manual backup.

How to make Android Share from X/LinkedIn work:
1. Host this whole folder over HTTPS. GitHub Pages, Netlify, Vercel, or your university webspace works.
2. Open the hosted URL in Chrome on Android.
3. Use Chrome menu -> Add to Home screen / Install app.
4. Open X or LinkedIn, tap Share on a post, and choose "Share n' Save".
5. The add-item window opens with the shared link/text prefilled. Choose category/subcategory and save.
