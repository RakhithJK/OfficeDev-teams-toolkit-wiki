
Teams Toolkit offers the ability to start debugging the application using the Teams Desktop Client

## Benefits
* Better performance and reliability
* Reduce time-to-F5
* Improve coverage of debug targets

## How to use the feature
The debug in Teams Desktop Client feature is built in for the following application templates scaffolded by Teams Toolkit:
- Copilot Plugin
- Custom Copilot
- Bot
- Message Extension

Step 1: Login to your Microsoft 365 account from Teams Toolkit:

![image](https://github.com/OfficeDev/teams-toolkit/assets/11220663/14ce6f25-465f-4038-afee-fe943146d037)

Step 2: Select the `Debug in Teams (Desktop)` from the `Run and Debug` panel and click the green arrow:

![image](https://github.com/OfficeDev/teams-toolkit/assets/11220663/abf3de64-2c9a-4212-b086-8a6084cf687b)

Step 3: Wait for the resource provision and you will notice a pop up that reminds you to make sure the account you used in Teams Toolkit matches the account you used in Teams desktop client. Click continue to proceed.

![image](https://github.com/OfficeDev/teams-toolkit/assets/11220663/2f3b53cd-557f-4b16-b1b3-b76b56f59df6)

Step 4: Add the app in the app installation flyout

![image](https://github.com/OfficeDev/teams-toolkit/assets/11220663/998925b0-bd46-4475-aba9-01ac3edee69f)

Step 5: Debug works the same as if it's in the Teams web client. You can put break points and hot reload your changes.

![image](https://github.com/OfficeDev/teams-toolkit/assets/11220663/0cbf3ff1-5348-4b6d-abf6-74b67de36d70)

## Important notes
1. Before proceeding, make sure your Teams desktop login matches your current Microsoft 365 account used in Teams Toolkit, otherwise you will see an `app not found` error.
2. The system-level notification will pop up only once per project. Next time you click `Debug in Desktop` Teams Toolkit will remind you about the account using notifications from VSCode.

![image](https://github.com/OfficeDev/teams-toolkit/assets/11220663/bd8cd459-da24-43fa-9c24-a42bbbeb1f77)
