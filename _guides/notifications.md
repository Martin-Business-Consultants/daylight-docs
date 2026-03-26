---
title: Notifications
nav_order: 8
description: Configure email and Slack notifications for new and recurring errors.
---

Daylight can notify you when new errors appear or when previously resolved errors recur.

## Notification triggers

Notifications are sent when:

- A **new error** is captured for the first time
- A **resolved error** re-opens due to a new occurrence

Notifications are rate-limited to a maximum of once per error per hour to prevent alert fatigue.

## Email notifications

Configure recipient email addresses in the dashboard Settings page as a comma-separated list.

When triggered, Daylight sends an email with:

- **Subject**: `[Daylight] ErrorClass: error message`
- **Body**: Error details, occurrence count, severity, and backtrace summary

{% include alert.html type="info" content="Email notifications use your application's existing ActionMailer configuration. Make sure your mail settings are working before enabling Daylight email alerts." %}

## Slack notifications

Enter a Slack incoming webhook URL in the dashboard Settings page.

Daylight posts a formatted message with:

- Red color indicator
- Error class and message
- Occurrence count, severity, and status fields

### Creating a Slack webhook

1. Go to [api.slack.com/apps](https://api.slack.com/apps) and create a new app
2. Enable **Incoming Webhooks**
3. Add a webhook to your desired channel
4. Copy the webhook URL into Daylight Settings

## Testing notifications

Use the **Test Notification** button in Settings to send a test alert through all configured channels. This verifies your email and Slack configuration without waiting for a real error.

## Graceful failure

Notification delivery never interferes with your application. If email or Slack delivery fails, the error is silently logged — your app continues running normally.
