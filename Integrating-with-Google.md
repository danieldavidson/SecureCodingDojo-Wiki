You will need a Google developer account and obtain the OAuth credentials from the Google API console: https://console.developers.google.com

In the same place you will setup your domain and authorized redirect URIs. For example for a localhost setup the redirect url would be: http://localhost:8081/public/google/callback

NOTE:
> Everyone with a Google account will be able to login to your app unless you setup IP restrictions.

Once you have the OAuth credentials configure the training portal as follows:
- Edit `encryptConfigs.js` and set the value for `googleSecret`
- Run `encryptConfig.js`
- Update the value for the `config.encGoogleClientSecret` variable in `config.js`
- Update the value for `config.googleClientId` in `config.js`