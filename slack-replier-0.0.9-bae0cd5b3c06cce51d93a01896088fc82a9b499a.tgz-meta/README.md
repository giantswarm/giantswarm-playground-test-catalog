[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/slack-replier/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/slack-replier/tree/main)

# slack-replier chart

**What is this app?**

An AI bot that auto replies to all questions asked in #ask-me-anything channel. This project kicked-off during Giant Swarm AI Hackathon of 2023.

**Why did we add it?**

- Avoid people distraction for usual doubts.
- Not reply twice to same question.
- Have better internal docs.

**Who can use it?**
- All Giant Swarm Staff.

**How it works?**

The bot app works as follows:
1. Starts an HTTP server waiting for POST request on route `/slack/events`.
2. Slack sends an event for new messages posted on #ask-me-anything.
3. The bot receives the incoming POST request and fetches the message content.
4. The bot then create a thread for the message and posts another message directed towards @Ask InKeep bot.
5. @Ask Inkeep bot then replies to the message question using trained knowledge from the Intranet handbook.

## Installing

### Create and configure the Slack app

Go the the [App page in Slack API](https://api.slack.com/apps/) and create a new app.

1) Give `channels:history`, `channels:read` and `chat:write` scopes to the app in the permissions page
2) Install the App in the workspace
3) Configure the `Event subscription` URL with the ingress domain of your running app
4) Copy the bot token and the verification token in the values provided to the Helm install command.
5) Deploy the app in the cluster
