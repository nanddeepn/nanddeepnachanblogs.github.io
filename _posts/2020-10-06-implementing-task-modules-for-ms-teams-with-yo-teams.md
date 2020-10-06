---
title: "Implementing Task Modules for MS Teams with Yo Teams"
date: "2020-10-06"
share: true
header:
  image: media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/09.gif
  teaser: media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/09.gif
categories:
  - MS Teams
tags:
  - "2020"
  - October 2020
last_modified_at: 2020-10-06T00:00:00-00:00
---

## Overview

Task modules enable us to implement modal popup experiences in MS Teams applications.
In this article, we will create task modules in a custom Microsoft Teams app using Yo Teams.


## Task Modules overview

Task modules help to implement popup in MS Teams applications. Inside the popup you can run any of the below:

- Custom HTML and JavaScript code
- Iframe based widgets
- Adaptive cards

From the MS Teams&#39; point of view, task modules can be considered as a tab inside a popup. The task modules can be invoked from any of the below:

- **Channel and personal tab:** Any button or link can invoke a task module by opening a popup.
- **Bot:** Button on cards sent from the bot can invoke a task module.
- **Deeplink:** You can compose a deeplink to invoke a task module.


## Prerequisites

Install below to get started:

- [Node.js](https://nodejs.org/) - v10.* (or higher)
- NPM (installed with Node.js) - v6.* (or higher)
- [Gulp](https://gulpjs.com/) - v4.* (or higher)
- [Yeoman](https://yeoman.io/) - v3.* (or higher)
- [Yeoman Generator for Microsoft Teams](https://github.com/OfficeDev/generator-teams) - v2.13.0 (or higher)
- [Visual Studio Code](https://code.visualstudio.com/)

Install Yeoman, Gulp global command-line interface, and Typescript compiler globally using NPM.

```
npm install yo gulp-cli typescript --global
```

Install generator-teams globally using NPM.

```
npm install generator-teams --global
```

To install preview versions of the generator, run the below command.

```
npm install generator-teams@preview --global
```


## Implement task modules with Yo Teams

We will generate a solution using Yo Teams.

1. On the command prompt, type below command.

    ```
    yo teams
    ```

2. Follow the wizard to set up the solution.

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/01.png)

3. On the command prompt, type code . to open the solution in visual studio code.


## Use the latest version of Teams manifest and SDK

Run the below command to ensure the project is using the latest version of Teams manifest and SDK.

```
npm i @microsoft/teams-js
````

**Changes to manifest.json**

Under the manifest folder, open the manifest.json file and make below changes:

- Change the `$schema` property to `https://developer.microsoft.com/en-us/json-schemas/teams/v1.7/MicrosoftTeams.schema.json`
- Change the `manifestVersion` property to `1.7`.

**Changes to gulp.config.js**

Under the root folder, open the gulp.config.js file and make below changes:

Add the following to the SCHEMAS property.

```json
{
    version: "1.7",
    schema: "https://developer.microsoft.com/en-us/json-schemas/teams/v1.7/MicrosoftTeams.schema.json"
}
```


## Test the personal tab

Before customizing the tab, let us test it.

1. From the command line, navigate to the root folder for the project and execute the below command:

    ```
    gulp ngrok-serve
    ```

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/02.png)

2. Open a browser and navigate to the ngrok URL displayed in the console:

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/03.png)

3. Update the URL in the browser to load the tab created from the scaffolding process.

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/04.png)

4. Now, let us try to load the tab in MS Teams. In the browser, navigate to [https://teams.microsoft.com](https://teams.microsoft.com/)

5. Using the app bar navigation menu, select the **More added apps** button. Then select **More apps**.

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/05.png)

6. Select **Upload a custom app** > **Upload for me or my teams**.
7. Upload the app package (a ZIP file) from the project&#39;s ./package folder.

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/06.png)

8. Click **Add**.

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/07.png)

9. The app will be displayed in MS Teams as below:

    ![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/08.png)


## Adaptive Cards

Adaptive Cards are a cross-product specification for various Microsoft products including Bots, Outlook, Teams, and Windows. An Adaptive Card is represented as a JSON object. The JSON string defines all the controls, text, and actions that the hosting application will use to render the card.

Below is a JSON example of an Adaptive Card that contains an input box and a single submit button:

```json
{
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.0",
    "body": [
      {
        "type": "Container",
        "items": [
          {
            "type": "TextBlock",
            "text": "YouTube Video Selector",
            "weight": "bolder",
            "size": "extraLarge"
          }
        ]
      },
      {
        "type": "Container",
        "items": [
          {
            "type": "TextBlock",
            "text": "Enter the ID of a YouTube video to show in the task module player.",
            "wrap": true
          },
          {
            "type": "Input.Text",
            "id": "youTubeVideoId",
            "value": ""
          }
        ]
      }
    ],
    "actions": [
      {
        "type": "Action.Submit",
        "title": "Update"
      }
    ]
}
```

Add above json content by creating a file at src\app\scripts\youTubePlayerTab\YouTubeSelectorCard.json


## Using Adaptive Cards in Microsoft Teams task modules

Once we have an Adaptive card ready, we will create a task module to display it and handle the submission action.

1. Open the file `src\app\scripts\youTubePlayerTab\YouTubePlayerTab.tsx`.
2. Add `youTubeVideoId` to the state.

    ```typescript
    export interface IYouTubePlayerTabState extends ITeamsBaseComponentState {
        entityId?: string;
        name?: string;
        error?: string;
        youTubeVideoId?: string;
    }
    ```

3. Update the render method as below:

    ```typescript
    public render() {
        return (
            <Provider theme={this.state.theme}>
                <Flex column gap="gap.smaller">
                    <Header>Task Module Demo</Header>
                    <Text>YouTube Video ID:</Text>
                    <Input value={this.state.youTubeVideoId} disabled></Input>
                    <Button content="Change Video ID (AdaptiveCard)" onClick={this.onChangeVideoAdaptiveCard}></Button>
                    <Button content="Show Video" primary onClick={this.onShowVideo}></Button>
                    <Text content="(C) Copyright Contoso" size="smallest"></Text>
                </Flex>
            </Provider>
        );
    }
    ```

4. Implement onChangeVideoAdaptiveCard method as below:

    ```typescript
    private onChangeVideoAdaptiveCard = (event: React.MouseEvent<HTMLButtonElement>): void => {
        // load adaptive card
        const adaptiveCard: any = require("./YouTubeSelectorCard.json");
        // update card with current video ID
        adaptiveCard.body.forEach((container: any) => {
            if (container.type === "Container") {
                container.items.forEach((item: any) => {
                    if (item.id && item.id === "youTubeVideoId") {
                        item.value = this.state.youTubeVideoId;
                    }
                });
            }
        });

        const taskModuleInfo = {
            title: "YouTube Video Selector",
            card: adaptiveCard,
            width: 350,
            height: 250
        };

        const submitHandler = (err: string, result: any): void => {
            this.setState(Object.assign({}, this.state, {
                youTubeVideoId: result.youTubeVideoId
            }));
        };

        microsoftTeams.tasks.startTask(taskModuleInfo, submitHandler);
    }
    ```

5. Implement onShowVideo method as below:

    ```typescript
    private onShowVideo = (event: React.MouseEvent<HTMLButtonElement>): void => {
        const taskModuleInfo = {
            title: "YouTube Player",
            url: this.appRoot() + `/youTubePlayerTab/player.html?vid=${this.state.youTubeVideoId}`,
            width: 1000,
            height: 700
        };
        microsoftTeams.tasks.startTask(taskModuleInfo);
    }
    ```

6. The helper appRoot() method is as below:

    ```typescript
    private appRoot(): string {
        if (typeof window === "undefined") {
            return "https://{{HOSTNAME}}";
        } else {
            return window.location.protocol + "//" + window.location.host;
        }
    }
    ```

7. Add an HTML page `src\app\web\youTubePlayerTab\player.html` to solution to render the YouTube video.

    ```html
    <!DOCTYPE html>
    <html lang="en">

        <head>
            <title>YouTube Player Task Module</title>
            <style>
                #embed-container iframe {
                    position: absolute;
                    top: 0;
                    left: 0;
                    width: 95%;
                    height: 95%;
                    padding-left: 20px;
                    padding-right: 20px;
                    padding-top: 10px;
                    padding-bottom: 10px;
                    border-style: none;
                }
            </style>
        </head>

        <body>
            <div id="embed-container"></div>
            
            <script>
                function getUrlParameter(name) {
                    name = name.replace(/[\[]/, '\\[').replace(/[\]]/, '\\]');
                    var regex = new RegExp('[\\?&]' + name + '=([^&#]*)');
                    var results = regex.exec(location.search);
                    return results === null ? '' : decodeURIComponent(results[1].replace(/\+/g, ' '));
                };

                var element = document.createElement("iframe");
                element.src = "https://www.youtube.com/embed/" + getUrlParameter("vid");
                element.width = "1000";
                element.height = "700";
                element.frameborder = "0";
                element.allow = "autoplay; encrypted-media";
                element.allowfullscreen = "";

                document.getElementById("embed-container").appendChild(element);
            </script>
        </body>

    </html>
    ```

## Test the solution

On the command prompt, type below command.

```
gulp ngrok-serve
```

![](/media/2020-10-06-implementing-task-modules-for-ms-teams-with-yo-teams/09.gif)


## Summary

Task modules enable us to implement modal popup experiences in MS Teams applications. Yo Teams helps to build task modules with ease.


## References

- [Collect Input in Microsoft Teams with Task Modules](https://docs.microsoft.com/en-us/learn/modules/msteams-task-modules?WT.mc_id=M365-MVP-5003693)
- [Yo Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/tutorials/get-started-yeoman?WT.mc_id=M365-MVP-5003693)


## Code Download

The code developed during this article can be found at: [https://github.com/nanddeepn/code-samples/tree/master/MSTeams/task-modules-tab/YouTubePlayer_AdaptiveCards](https://github.com/nanddeepn/code-samples/tree/master/MSTeams/task-modules-tab/YouTubePlayer_AdaptiveCards)