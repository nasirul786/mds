## Bot dialogues

These are the exact message texts the bot sends to users. Bold formatting (`<b>…</b>`) is rendered as HTML by Telegram and is included verbatim so the source is easy to grep.

### `/start` (welcome + application kick-off)

When a user has no prior application (or has a rejected one with `can_apply_again = true`):

```
<b>Welcome!</b>

Apply for the <b>Dollars Capital Circle Online Mentorship</b>.

Your journey begins with live online learning sessions covering everything from basic to advanced trading concepts. This is followed by 6 months of guided live trading, helping you apply strategies in real market conditions. Includes weekly weekend doubt sessions and trading psychology coaching to build discipline and consistency.

Tap <b>Continue</b> to begin your application.
```

Inline buttons: `Continue` and `Learn more about the program`.

When the user already has a **pending** application:

```
<b>Application already submitted</b>

Your mentorship application is currently <b>pending</b> review. You cannot submit another request.
```

When the user already has an **approved** application:

```
<b>Application approved</b>

Your mentorship application has already been <b>approved</b>. You cannot submit another request.
```

When the user has a **rejected** application and is not allowed to re-apply:

```
<b>Application rejected</b>

Your previous application was <b>rejected</b> and you are not allowed to apply again yet.

<b>Reason:</b> {rejection reason if any}
```

If the bot cannot read the Telegram profile (rare):

```
<b>Error:</b> Could not read your Telegram profile.
```

### Tapping the `Continue` button

```
<b>Step 1 of 3: Phone number</b>

Please enter your valid phone number.
```

### `/start request-recordings` (deep link from the bot's "request recordings" button)

Not approved:

```
<b>Not eligible</b>

Only approved mentorship applicants can request the last year's recordings.
```

Already delivered:

```
<b>Already delivered</b>

You have already received the video.
```

Already in queue:

```
<b>Already applied</b>

You have already applied for the last year's mentorship recordings.

Your queue number is <b>{N}</b>.
```

Request accepted:

```
<b>Request received</b>

You have successfully requested for the last year's mentorship recordings.

The video will be delivered to you in this chat shortly. Please allow us some time to process the video for you.

Your queue number is <b>{N}</b>.
```

### `/cancel`

Nothing in progress:

```
<b>Nothing to cancel.</b> No application is in progress.
```

Cancelled mid-flow:

```
<b>Application cancelled.</b>

Send /start when you want to apply again.
```

### Application step messages

After a valid phone number is accepted:

```
<b>Step 2 of 3: Email address</b>

Please send your email address.

The email address must be registred with ZOOM, if you do not have created ZOOM account, please create one and use the same email address for the application. <a href='https://zoom.us/signup'>Click here to create zoom account</a>
```

After a valid email is accepted:

```
<b>Step 3 of 3: Payment screenshot</b>

Please send a photo of your payment screenshot.
```

Invalid phone:

```
<b>Invalid phone number</b>

Please enter your valid phone number.
```

Invalid email:

```
<b>Invalid email address</b>

Please send a valid email address.
```

Unrelated command sent during the flow:

```
<b>Application in progress</b>

Please complete the current step, or send /cancel to stop.
```

Session lost (e.g., bot restarted between steps):

```
<b>Session expired</b>

Please send /start to begin again.
```

Application saved successfully:

```
<b>Application submitted successfully!</b>

Thank you for applying to the <b>Dollars Capital Circle Online Mentorship</b>.

Your application status is <b>pending</b>. We will review it and get back to you.
```

Save failed:

```
<b>Something went wrong</b>

We could not save your application. Please try again in a moment.
```

### Admin actions → user DMs (`src/telegram-notify.ts`)

On **approve**, when `COMMUNITY_CHAT_ID` is **not** configured:

```
<b>Application approved</b>

Your mentorship application for the <b>Dollars Capital Circle Online Mentorship</b> has been <b>approved</b>.
```

On **approve**, when `COMMUNITY_CHAT_ID` is configured (one or more community chats), the same message plus:

```
<b>Join the private community</b>

Use the link below to join the community chat. This link is one-time-use and expires in 7 days, please do not share it.

<a href="{invite_link}">{invite_link}</a>
```

On **reject** (with reapply allowed):

```
<b>Application rejected</b>

Your mentorship application was <b>rejected</b>.

<b>Reason:</b> {reason}

You are allowed to <b>apply again</b>. Send /start when you are ready.
```

On **reject** (no reapply):

```
<b>Application rejected</b>

Your mentorship application was <b>rejected</b>.

<b>Reason:</b> {reason}

You are <b>not allowed</b> to apply again at this time.
```

### Recordings channel handler (`src/recordings.ts`)

When the worker posts a video to `RECORDINGS_CHANNEL_ID`, the bot replies in the channel to indicate delivery status.

Caption missing the `::userid` suffix:

```
<b>Failed to send this video to user.</b>

Recipient user id not found in caption. Expected format: <code>...::12345678</code>
```

Delivered successfully to the user:

```
<b>Successfully sent this video to the user.</b>
```

Delivery failed (e.g., user blocked the bot):

```
<b>Failed to send this video to user.</b>

Error message: {Telegram API error description}
```

### Community guard (`src/community-guard.ts`)

When a non-approved user joins a configured community chat they are removed and the bot posts in that chat:

```
<b>Removed unauthorized member</b>

{mention of user} was removed because {reason}.
```

`{reason}` is one of:

- `they have not applied for the <b>Dollars Capital Circle Online Mentorship</b>.`
- `their mentorship application is still <b>pending</b> review by the admin.`
- `their mentorship application was <b>rejected</b> by the admin.` (optionally followed by `Reason: {admin-provided reason}`)
