You will need to create a Slack app for authentication.

#### Configuring the Slack app

- Go to https://api.slack.com/
- Hit Start Building and follow the prompts to create a Slack App in your Development Slack Team
- Once your app is created you will be able to get your App Credentials to configure your config.js
- Configure redirect urls under OAuth & Permissions. For example for a localhost setup the redirect url would be: http://localhost:8081/public/slack/callback

#### Configuring the training portal

- Edit `encryptConfigs.js` and set the value for `slackSecret`
- Run `encryptConfig.js`
- Update the value for the `encSlackClientSecret` variable in `config.js`
- Update the values for `slackClientId` and `slackTeamId`