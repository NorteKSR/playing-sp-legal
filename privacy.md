# Playing SP: Privacy Policy

**Effective date:** 2026-09-26 <!-- update to the date you publish, if later -->
**Contact:** paws.epsilon-97@icloud.com

> This policy describes how the Playing SP Twitch Extension ("Playing SP", "the extension", "we") handles data. It is a plain-language description of how the software works. It is not legal advice. If you need legal certainty for your jurisdiction, have a lawyer review it.

Playing SP is a Twitch Extension. It shows viewers the song or podcast episode a broadcaster is currently playing on Spotify. It shows **information only**. It never plays, streams or records any audio.

Playing SP is an independent project. It is **not affiliated with, endorsed by or sponsored by Spotify AB or Twitch Interactive, Inc.** Spotify and Twitch are trademarks of their respective owners.

---

## 1. Who this policy covers

- **Broadcasters** who install the extension and connect a Spotify account.
- **Viewers** who see the extension on a channel.

## 2. Data we store about broadcasters

When a broadcaster clicks **Connect Spotify** on the extension's configuration page and approves access on Spotify's own website, our backend stores one record, keyed by the broadcaster's **Twitch channel ID**:

| Data | Why | How it's protected |
|---|---|---|
| Spotify **access token** and **refresh token** | To ask Spotify what is currently playing on the connected account | **Encrypted at rest** with AES-256-GCM. The key is kept separately as a server secret. |
| Twitch **channel ID** (a number Twitch assigns to the channel) | To link the Spotify connection to the right channel | Stored in the protected data file (see section 5) |
| Spotify **user ID** and **display name** of the connected account | To show the broadcaster which Spotify account is connected | Stored in the protected data file, not separately encrypted |
| Granted Spotify **scopes**, link time, last-updated time, connection status and the last error code (e.g. "token revoked") | To run the connection and show the broadcaster its status | Stored in the protected data file |

While the broadcaster is completing the Spotify sign-in, a short-lived **sign-in session** is also stored. It holds a random one-time value, the channel ID and a PKCE verifier, all encrypted, and it **expires after 10 minutes**.

We do **not** receive or store the broadcaster's Spotify password, email address, payment details, playlists, listening history or followers.

### Spotify permissions we request

Only these two read-only [Spotify scopes](https://developer.spotify.com/documentation/web-api/concepts/scopes):

- `user-read-currently-playing`: read the track or episode currently playing.
- `user-read-playback-state`: read whether playback is playing or paused, and its progress.

We also read the connected account's Spotify user ID and display name from Spotify's profile endpoint (`/me`), which needs no extra scope. We cannot control playback, change playlists or see any other account data.

## 3. Current-track information

While a broadcaster is connected and their channel has viewers, our backend asks Spotify every few seconds what is playing. It then shares that information with the channel's viewers through Twitch's Extension messaging (PubSub). The information is:

- Track or episode title, artist(s) or show name, album name
- Album or episode artwork URL (hosted by Spotify at `i.scdn.co`) and the Spotify link to the item
- Whether the item is marked explicit, playback progress, duration, play/pause state and timestamps
- The Twitch channel ID it belongs to

This information is **public by nature**: it is shown to everyone watching the channel. We keep the latest value **in memory only**. It is not written to disk and is gone when the server restarts or the broadcaster disconnects.

## 4. Viewers

Viewers do not sign in to Playing SP and we do not ask them for anything.

When a viewer's browser loads the extension, Twitch gives it a signed token (JWT). The token holds an **opaque user ID**: a random identifier that Twitch generates and that doesn't reveal who the viewer is. It also holds the channel ID and the viewer's role (viewer or broadcaster). It holds the real Twitch user ID only if the viewer has chosen to share their identity with extensions. Playing SP never asks viewers to share their identity.

- We **verify** this token on each request to confirm it really came from Twitch. We **do not store** viewer IDs in our database.
- For abuse protection, our server keeps a short-lived **in-memory rate-limit counter** keyed by the channel ID and the viewer's opaque ID (or the connecting IP address if no ID is present). The counter resets every minute, is never written to disk and is lost on restart.
- Our backend keeps an in-memory, per-channel "someone is watching" timestamp for about 10 minutes, so it can stop asking Spotify when nobody is watching. It does not record who is watching.
- Our application **does not log** viewer opaque IDs, Twitch user IDs or IP addresses.
- We don't use cookies, analytics, advertising trackers or fingerprinting.
- Album artwork is loaded by the viewer's browser directly from Spotify's image server (`i.scdn.co`). If a viewer clicks **Listen on Spotify**, they leave the extension and go to `open.spotify.com`. Spotify's own privacy policy applies there.

## 5. Where data is stored, and security

- The backend runs on **[Fly.io](https://fly.io)**. Broadcaster records are stored in one data file on a Fly.io storage volume. Only our server process can read the file (file mode 0600). Fly.io also encrypts volumes at rest.
- Spotify tokens inside that file are additionally encrypted with AES-256-GCM, as described above.
- All connections use HTTPS.
- Tokens are **never written to logs**.

No system is perfectly secure. If we discover a breach that affects broadcaster data, we will disconnect affected accounts and notify affected broadcasters where we can.

## 6. Logs

Our server writes operational logs, which Fly.io collects. They may contain:

- Twitch channel IDs (e.g. "polling started for channel …")
- When a broadcaster connects: the **Spotify user ID** that was linked to a channel ID
- Error messages from Spotify or Twitch (shortened), e.g. when a token has been revoked

The logs contain no tokens, viewer IDs or IP addresses from our application. As our hosting provider, Fly.io may process connection metadata such as IP addresses at its network edge under its own policies. Logs are kept only as long as our Fly.io log retention allows, and we do not export them to other services. <!-- If you add a log drain or longer log retention, update this section. -->

## 7. Third parties

We do not sell, rent or share data for advertising. Data goes only to the services needed to run the extension:

| Service | What it receives | Their policy |
|---|---|---|
| **Spotify** (Spotify AB) | The broadcaster's sign-in (on Spotify's site), then our requests for current playback using the broadcaster's token. Viewers' browsers load artwork from Spotify. | https://www.spotify.com/legal/privacy-policy/ |
| **Twitch** (Twitch Interactive, Inc.) | The extension runs inside Twitch. We send the current-track information to Twitch for delivery to viewers, and may read or set the extension's configuration state for the channel. | https://www.twitch.tv/p/legal/privacy-notice/ |
| **Fly.io** (Fly.io, Inc.) | Hosts our backend, stores the data file and collects logs | https://fly.io/legal/privacy-policy |

We may disclose data if the law requires us to.

## 8. Retention

| Data | How long we keep it |
|---|---|
| Broadcaster record (encrypted tokens, channel ID, Spotify user ID and display name, status) | Until the broadcaster **disconnects** (or asks us to delete it). If Spotify revokes access, the tokens are wiped right away. The remaining status record (channel ID, Spotify ID and display name, error code) is kept until the broadcaster disconnects or asks for deletion. |
| Sign-in sessions | 10 minutes at most |
| Current track, viewer-activity timestamps, rate-limit counters | In memory only. Minutes at most, and lost on restart. |
| Server logs | According to Fly.io's log retention. We don't keep separate copies. |
| Volume snapshots (backups) | Fly.io takes automatic daily volume snapshots, kept for **5 days** by default. A deleted record may therefore remain in an encrypted snapshot for up to about 5 days. <!-- confirm your Fly.io snapshot retention setting --> |

## 9. Disconnecting and deleting your data

**Broadcasters can disconnect at any time:**

1. Open the Twitch Creator Dashboard → **Extensions** → **My Extensions** → Playing SP → **Configure**.
2. Click **Disconnect** and confirm.

Disconnecting stops polling right away and **permanently deletes the channel's whole record** (including the encrypted tokens) from our data store.

Spotify doesn't let apps revoke their own access. To also remove Playing SP's access on Spotify's side, go to **https://www.spotify.com/account/apps/** and remove the app. If you do that without disconnecting first, our tokens stop working and are wiped automatically. Click Disconnect as well to remove the rest of the record.

Uninstalling the extension on Twitch doesn't automatically notify us, so **please click Disconnect before uninstalling**, or email us.

**Deletion or access requests:** email **paws.epsilon-97@icloud.com** from any address and include your Twitch channel name. We'll confirm the channel is yours (for example by asking you to click Disconnect, or to post a short code on your channel). We will delete the data, or tell you what we hold, within 30 days. Viewers: we hold no viewer data that we could identify you by, but you may still contact us.

## 10. Children

Playing SP is not directed at children under 13 (or the minimum age for Twitch in your country), and we do not knowingly collect personal information from children. Twitch's own rules require users to be at least 13. If you believe a child has connected an account, contact us and we will delete it.

## 11. International users

Our servers are operated through Fly.io and may be located outside your country. By using the extension you understand that the limited data described here may be processed there.

## 12. Changes to this policy

We may update this policy when the extension changes. We'll change the effective date above. For significant changes we'll also note them in the extension's release notes or on the configuration page. Continuing to use the extension after a change means you accept the updated policy.

## 13. Contact

Questions, requests or concerns: **paws.epsilon-97@icloud.com**
