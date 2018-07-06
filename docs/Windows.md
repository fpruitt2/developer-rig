This guide assumes you have completed on-boarding and nothing else.

1.  Install all dependencies.
    1.  [Node LTS](https://nodejs.org/en/download/).  If you already have Node installed, it must be at least version 6.
    2.  [Yarn](https://yarnpkg.com/lang/en/docs/install).
    3.  [Python 2](https://www.python.org/downloads/release/python-2715/).
    4.  [Git for Windows](https://github.com/git-for-windows/git/releases/download/v2.17.1.windows.2/Git-2.17.1.2-64-bit.exe).  Its shell is used in subsequent steps.
2.  Add `127.0.0.1 localhost.rig.twitch.tv` to `/etc/hosts`.
    1.  Press the Windows key.
    2.  Type `notepad`.
    3.  In the search results, right-click **Notepad** and select **Run as administrator**.  Accept the elevation prompt, if any.
    4.  In **Notepad**, open `%SystemRoot%\System32\drivers\etc\hosts`.  This is usually `C:\Windows\System32\drivers\etc\hosts`.
    5.  Add `127.0.0.1 localhost.rig.twitch.tv` to the bottom of the file.
    6.  Press Ctrl+S to save your changes.
3.  Create the extension at Twitch.
    1.  Visit [https://dev.twitch.tv/dashboard/extensions](https://dev.twitch.tv/dashboard/extensions).
    2.  Click **Create Extension**.
    3.  Provide all requested information about your new extension and click **Create Extension**.
    4.  Keep this browser session open since you'll need some of the data on it to create a configuration file later.  
        Note that you didn't actually create an extension.  What you did was create Twitch's awareness of your extension (i.e., an internal manifest for your extension).
4.  Open **Git Bash** and execute these commands in a directory of your choosing.
    1.  `git clone https://github.com/twitchdev/developer-rig`
    2.  `cd developer-rig`
    3.  `yarn install` # This takes about half a minute.
    4.  `yarn extension-init -l ../my-extension`  
        \# You may replace `my-extension` with a different directory name here and in subsequent steps.
    5.  `node scripts/ssl.js`
    7.  `mkdir ../my-extension/conf`
    8.  `mv ssl/selfsigned.crt ../my-extension/conf/server.crt`
    9.  `mv ssl/selfsigned.key ../my-extension/conf/server.key`
    9.  `yarn host -l ../my-extension/public -p 8080`
5.  Visit [https://localhost.rig.twitch.tv:8080/viewer.html](https://localhost.rig.twitch.tv:8080/viewer.html).  Allow the certificate.  You will see **Hello, World!** in the browser window.
6.  Open another **Git Bash** and execute these commands in the same directory as above.
    1.  `cd my-extension`
    2.  `npm install`
    3.  `curl -H 'Client-ID: `_your client ID_`' -X GET 'https://api.twitch.tv/helix/users?login=`_your Twitch user name_`' | sed -e 's/.*"id":"//' -e 's/".*//'`  
        \# This will print your Twitch user ID for the next step.
        \# Your client ID is available in the **Overview** tab of your extension.  Your Twitch user name is displayed in the top-right corner of twitch.tv when you are signed in.  The channel name must be alphabetic.  For instance, if your Twitch user name is `dallas`, use the channel name `RIGdallas`.
    4.  `node services/backend -c '`_your client ID_`' -s '`_your extension secret_`' -o '`_your Twitch user ID_`'`  
        Your extension secret is in the **Secret Keys** section in the **Settings** tab of your extension.
7.  Visit [https://localhost:8081](https://localhost:8081).  Allow the certificate.  You will see some JSON describing a 404 response.
8.  Open another **Git Bash** and execute these commands in the same directory as above.
    1.  `cd developer-rig`
    2.  `echo '{
          "clientID": "`_your client ID_`",
          "version": "0.0.1",
          "channel": "RIG`_your Twitch user name_`",
          "ownerName": "`_your Twitch user name_`"
        }' > ../config.json`  
    3.  `yarn start -s '`_your extension secret_`' -c ../config.json`
9.  This will open [https://localhost.rig.twitch.tv:3000/](https://localhost.rig.twitch.tv:3000/).  Allow the certificate.  You will see the developer rig with no extension views.
9.  Make the required configuration changes.
    1.  Go back to your dev.twitch.tv browser session.
    2.  Click the **Versions** tab then click **Manage**.  If there is no such button, you're already on the right page.
    3.  Click **Asset Hosting**.
    4.  Change **Testing Base URI** from `https://localhost:8080/` to `https://localhost.rig.twitch.tv:8080/`.
    6.  Click **Save Changes**.
    7.  Go back to your developer rig browser session.
    8.  Click **Configurations**.  The **Dev Rig Configurations** dialog will open.
    9.  Verify the `viewer_url` is `https://localhost.rig.twitch.tv:8080/viewer.html`.  Since it might take a while for the change to take effect as it propagates through Twitch's internal systems, click **Refresh** until you see the change.
    9.  Click **Cancel**.
9.  Verify the rig is working.
    1.  Click the **+** button.  The **Add a new view** panel will appear.
    2.  Select the **Broadcaster** viewer type and click **Save**.  The Broadcaster view will appear.
    3.  Click **Yes, I would**.  Verify the color changes and there is output to match that request in the second Git Bash shell.
    4.  Click the **+** button again.  The **Add a new view** panel will appear.
    5.  Select the **Logged-Out Viewer** viewer type and click **Save**.  The Logged-Out Viewer view will appear.
    6.  Click **Yes, I would**.  Verify the color changes in both views and there is output to match that request in the second Git Bash shell.

### Issues

Some browser extensions (e.g. noscript) prevent these endpoints from working.  Configure them as necessary to allow access to these endpoints.  If you are using Firefox, consider creating a profile specifically for the developer rig that has no browser extensions.

_I see a browser certificate warning._  
You need to allow the certificate.  How you do so varies by browser.  Consider allowing it permanently so you won't have to allow it every time you run the rig.

_When I click the **+** button and add the **Broadcaster** viewer type, I see an empty box._  
You need to allow the certificate for the viewer.  See step 5 above.

_When I add the **Broadcaster** viewer type or when I click the **Yes, I would** button, I see **EBS request returned (error)** messages in the rig console._  
You need to allow the certificate for the EBS.  See step 7 above.

_The color updates only in the view in which I clicked the **Yes, I would** button._  
Verify your channel name is alphabetic.  It may not contain any other characters, such as hyphens, numbers, and underscores.

_I changed the **channel** in the configuration and now the color updates only in the view in which I clicked the **Yes, I would** button._  
This is a known issue.  Currently, once you create your extension and start using a channel, you may not change it to a different one.  If, after changing it back, you still experience this issue, clear the local storage for the developer rig (https://localhost.rig.twitch.tv:3000/) in your browser.  This will necessitate recreating the views in the rig.

_I see **EBS request returned Too Many Requests (error)** messages in the rig console._  
Twitch uses rate limiting to prevent overloading our systems.  Your EBS and your extension will need to account for this and other HTTP errors.
