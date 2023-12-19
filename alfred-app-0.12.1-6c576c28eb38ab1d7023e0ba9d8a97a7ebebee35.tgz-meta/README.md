# Alfred - Your slack companion

Alfred is an app that enhances support workflows in Slack.

## Features

- Create internal support threads automatically when oncaller marks customer thread as `ACK`.
- Keep in memory the list of support threads.
- Mark support threads as complete when customer support message is masked as `DONE`.
- Report daily about open/closed threads.

## Development

We have a [dev Alfred bot](https://api.slack.com/apps/A03HSNFMLJD) create in Slack for development. In order to develop the bot you need to do a couple of actions

1) Modify a secret.yaml in working directory to add the real settings from Slack

```yaml
  slack:
    reportingChannel: C015X4E70BY #test-afred
    verificationToken: <token> # Copy it from the app config https://api.slack.com/apps/A03HSNFMLJD/general?
    slackToken: <token> # Copy it from the app config https://api.slack.com/apps/A03HSNFMLJD/install-on-team?
```

2) Use [ngrok](https://ngrok.com/download) to make your local Alfred instance routable from Slack and configure the [event URL in Slack app](https://api.slack.com/apps/A03HSNFMLJD/event-subscriptions?)

```bash
ngrok http 8080
```

![events](./docs/events.png)

Now you can use bot and see reactions in #test-alfred channel. The bot by now is installed in #support-test only

## Metrics

Alfred expose some metrics to enable the support monitoring. This is the list of metrics availables now:

- `support_alfred_threads_total` It is a counter with the number of closed support threads (marked with emoji `DONE`) and it contains team and customer in labels
- `support_alfred_threads_duration_seconds` is a histogram of the number of seconds per closed thread in buckets (60, 120, 240, 480, ...). 

Those metrics are pushed to Grafana Cloud using a prometheus agent sidecar running in the same alfred deployment. The configuration can be found [here](https://github.com/giantswarm/alfred-app/blob/main/helm/alfred-app/templates/configmap-agent.yaml).

By now, there is a dashboard called `Customer Success` in Grafana Cloud showing some nice visualizations from the data collected by Alfred.
