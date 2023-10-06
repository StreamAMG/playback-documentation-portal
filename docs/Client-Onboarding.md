---
internal: true
---

# Client Onboarding

The playback service and features available to clients are configured via the [Playback Configuration API](../reference/Playback-Configuration-API.yaml).

Client onboarding is managed through client configuration which is managed in the AWS DynamoDB table 'playback-api-production-clients-configs' and holds
information for a client specific for playback such as:

- player configuration e.g. for the 'bitmovin' video player
- player MUX analytics keys
- player resume capability (on/off)
- global adverts configuration
- player licences

Here is our [Client Onboarding Guide](https://streamuk.atlassian.net/wiki/spaces/DEV/pages/3792502785/Client+Onboarding).
