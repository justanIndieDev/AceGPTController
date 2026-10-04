# AceGPT Controller

A small GitHub Pages dashboard for the AceGPT Discord bot. It can show connected Discord servers, summon/dismiss AceGPT from voice channels, and clear a text channel's conversation history.

**The page is only the dashboard.** AceGPT still runs on your own PC. To control it over the internet, its local authenticated controller API must be running and exposed through a temporary HTTPS tunnel.

## Published dashboard

After GitHub Pages is enabled and the Actions deployment succeeds:

<https://justanindiedev.github.io/AceGPTController/>

## What you need

- Windows computer with AceGPT installed, Discord access, and Python dependencies installed.
- A Discord bot that is already invited to your server. For voice controls it needs View Channel, Connect, and Speak in the selected voice channel, plus View Channel and Send Messages in the selected text channel.
- cloudflared installed on the computer running AceGPT.
- Your private CONTROLLER_TOKEN, stored only in AceGPT's local .env and entered into the dashboard when connecting.

## First-time GitHub Pages setup

1. Open the repository's **Settings → Pages**.
2. Under **Build and deployment**, choose **GitHub Actions** as the source.
3. Open **Actions** and confirm **Deploy controller to GitHub Pages** completes successfully.
4. Visit the published dashboard link above.

The workflow publishes only the site/ directory. It does not deploy the bot, copy .env, or contain Discord or AI credentials.

## Configure AceGPT's local controller

In the AceGPT folder, add these settings to the local .env file:

    CONTROLLER_TOKEN=replace_with_a_long_random_private_value
    CONTROLLER_PORT=8765
    CONTROLLER_ALLOWED_ORIGIN=https://justanindiedev.github.io

Generate a strong value for CONTROLLER_TOKEN locally, for example with Python's secrets.token_urlsafe(32). Do not use a Discord bot token or AI provider key here. Do not paste this controller token into GitHub, Discord, chat messages, or screenshots.

Make sure the AceGPT source includes its local controller API and that aiohttp is installed (the Discord library uses it, but the controller imports it directly). Restart AceGPT after changing .env.

## Start a temporary secure tunnel

With AceGPT running, open PowerShell in the AceGPT folder and run:

    cloudflared tunnel --url http://127.0.0.1:8765

Cloudflare prints an HTTPS URL ending in trycloudflare.com. Keep that PowerShell window open. In the dashboard, enter that full HTTPS address and the local CONTROLLER_TOKEN, then press **Connect**.

Quick Tunnels create a temporary public URL; stopping the tunnel ends access and a later run usually gets a different URL. Your bot computer, AceGPT, and the tunnel must remain online while you use the dashboard. Do not share the URL and token together.

## Use the dashboard

- **Connect:** verifies the address and token, then lists the Discord servers AceGPT can see.
- **GPT Summon:** select a server, voice channel, and text channel, then summon AceGPT.
- **GPT Dismiss:** disconnects AceGPT from voice in the selected server.
- **Clear channel history:** resets that text channel's AI conversation history after confirmation.
- **Refresh status:** fetches current server and voice-session status.

The address and controller token are kept in the current browser tab's session storage and can be cleared with **Forget saved connection** or by closing the tab. They are not written to this repository or sent to GitHub.

## Troubleshooting

- **Connection failed:** confirm AceGPT is running, the tunnel window is still open, and you copied the current HTTPS tunnel URL without extra characters.
- **401 / incorrect token:** compare the dashboard token with CONTROLLER_TOKEN in the local .env, then restart AceGPT after any change.
- **403 website not allowed:** set CONTROLLER_ALLOWED_ORIGIN=https://justanindiedev.github.io in .env and restart.
- **No servers or channels listed:** ensure AceGPT is online in Discord and has access to the server and channels.
- **Cannot summon:** grant the voice and text channel permissions listed above.
- **Dashboard does not load:** check the Pages workflow under the repository's Actions tab and Pages settings.

## Security and limitations

This repository contains only a static web dashboard and its deployment workflow. The controller token is never embedded in the published page; it is provided by you in the browser at connection time. The local controller binds to loopback and expects an authenticated request. The HTTPS tunnel makes that local API reachable from the public internet, so protect both the tunnel URL and token. Anyone who has both can control the bot while the tunnel is active. Stop the tunnel when you are finished.

This setup is for a single local bot process and a temporary Quick Tunnel. GitHub Pages does not host the Discord gateway bot or keep it running.