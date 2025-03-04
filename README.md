# Spotify Link Getter for Google Sheets

This project allows users to retrieve Spotify track links directly into their Google Sheets using a custom Google Apps Script.

see the demo of this feature in this video: [https://youtu.be/R0VlONnZ1DI](https://youtu.be/R0VlONnZ1DI)

## Prerequisites

1. A Google account.
2. A Google Sheet where the script will be added.

## Setting Up the Google Apps Script

1. **Create a new Google Sheet** or open an existing one.
2. **Go to Extensions > Apps Script**.
3. **Delete any existing code in the script editor** and **paste the following code**:

    ```javascript
    // Spotify API credentials
    const clientId = 'YOUR_CLIENT_ID';
    const clientSecret = 'YOUR_CLIENT_SECRET';
    const tokenUrl = 'https://accounts.spotify.com/api/token';

    function getSuggestedLink(song, composer) {
      var response = UrlFetchApp.fetch(tokenUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        payload: `grant_type=client_credentials&client_id=${clientId}&client_secret=${clientSecret}`
      });

      var json = JSON.parse(response.getContentText());
      var accessToken = json.access_token;

      var searchResponse = UrlFetchApp.fetch(`https://api.spotify.com/v1/search?q=${encodeURIComponent(song)}+${composer}&type=track`, {
        headers: {
          'Authorization': `Bearer ${accessToken}`
        }
      });

      var result = JSON.parse(searchResponse.getContentText());
      if (result.tracks && result.tracks.items.length > 0) {
        return result.tracks.items[0].external_urls.spotify;
      } else {
        return 'No results found';
      }
    }

    function onOpen() {
      var ui = SpreadsheetApp.getUi();
      ui.createMenu('Spotify Link')
        .addItem('Get Suggested Link', 'getSpotifyLink')
        .addToUi();
    }

    function getSpotifyLink() {
      var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
      var activeCell = sheet.getActiveCell();
      var row = activeCell.getRow();
      var song = sheet.getRange(row, 1).getValue();  // Assuming Song is in column 1
      var composer = sheet.getRange(row, 2).getValue();  // Assuming Composer is in column 2

      if (song && composer) {
        var spotifyLink = getSuggestedLink(song, composer);
        sheet.getRange(row, 4).setValue(spotifyLink);  // Assuming Link is in column 4
        SpreadsheetApp.getUi().alert('Spotify link suggested and added to the Link column.');
      } else {
        SpreadsheetApp.getUi().alert('Please make sure both Song and Composer are filled in.');
      }
    }
    ```

## Customization

1. **Replace `YOUR_CLIENT_ID` and `YOUR_CLIENT_SECRET`** with the Client ID and Client Secret I provided.
2. **Save the script** and give it a name.

## Using the Script in Google Sheets

1. **In your Google Sheet**, enter the track name in a cell (e.g., `A1`).
2. **Enter the composer in another cell** (e.g., `B1`).
3. **In another cell**, use the custom function `=SPOTIFY_LINK(A1, B1)` to get the Spotify track link.

## Getting Access to My Spotify App

In Spotify Developer console the app is called LinkFinder. I tis in dev mode right now so I don't think it isdiscoverable but I am allowed to provsion access to 25 users (24 including myself). To get access to the App, reach out to me so I can get your name and email to add you as a user.

## Optional: Setting Up Your Own Spotify Developer App

If you prefer to set up your own Spotify Developer App using their Web API for this feature in Google Sheets, follow these steps:

1. **Create a Spotify Developer Account**:
   - Go to the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/).
   - Log in with your Spotify account.

2. **Create a New App**:
   - Click on **Create an App**.
   - Fill in the required fields:
     - App name.
     - App description.
   - Redirect URI: Add any valid URL (e.g., `http://localhost`), this won't be used but it's required by Spotify.

3. **Note Your Client ID and Client Secret**:
   - Once your app is created, you'll see your **Client ID** and **Client Secret**. 

4. **Update the Google Apps Script**:
   - Replace `YOUR_CLIENT_ID` and `YOUR_CLIENT_SECRET` in the script with your own Spotify Developer credentials.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
