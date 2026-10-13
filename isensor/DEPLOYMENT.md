# SWFGS i-Sensor pilot deployment

This is a supervised **pilot**, not approved for rollout across centres. It contains the WebHID implementation used in the successful experimental transfer, plus automatic same-origin firmware download using a manifest.

## Local setup (VS Code)

1. Copy this `isensor` directory into your existing SWFGS Git working tree.
2. Obtain permission to host/redistribute Autogem firmware. Copy your authorised, already-tested HEX to `isensor/firmware/Autogem_Combo_Master_V5.4_1.73_2026_07.hex`.
3. Check the SHA-256 matches `isensor/firmware/manifest.json`. Never alter a checksum to silence a mismatch.
4. Test via HTTPS (or localhost for development); WebHID requires a secure context and Chrome/Edge.
5. Commit and push using your existing Git workflow. On VPS: `cd /var/www/swfgs && git pull --ff-only`.
6. Check `https://swfgs.co.uk/isensor/firmware/manifest.json` and verify a missing HEX returns 404, not the SWFGS homepage.

## Nginx recommended

Your existing `location / { try_files $uri $uri/ /index.html; }` serves the SWFGS homepage for missing files. Add this inside the SWFGS HTTPS server block (before deployment), then `sudo nginx -t && sudo systemctl reload nginx`:

```nginx
location ^~ /isensor/firmware/ {
    try_files $uri =404;
    add_header Cache-Control "no-store" always;
    add_header X-Content-Type-Options nosniff always;
}
```

Do not remove the hidden-file access block protecting `.git`. WebHID uses the `hid` permissions feature; `usb=()` does not itself disable WebHID. Test WebHID on the deployed origin.

## Automatic updates — scope

The page automatically fetches the approved manifest and firmware from **your SWFGS server**. It does **not** yet discover or download newly published Autogem releases. An official stable upstream source, redistribution permission and compatible-format validation are needed before enabling that safely. The current parser still checks the tested firmware layout; a new firmware may be rejected even if the manifest is updated. This is intentional.

## Safety

- This is not independently validated on a second device.
- The final proprietary completion command lacks a device-confirmed success reply.
- A tab-close warning is advisory and cannot prevent forced termination, crashes, sleep or unplugging.
- Do not run production fleet updates until hardware recovery and additional device tests are established.
- Firmware bytes and logs remain local to the browser during programming; the server serves static assets and can still have standard Nginx access logs.
