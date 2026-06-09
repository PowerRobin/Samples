## Add Mailbox

Go to exchange admin center and create new shared mailbox
https://admin.cloud.microsoft/exchange#/mailboxes
Note down the mail-address of the mailbox, you will need it later :D

image1

Add users to shared mailbox with delegation permissions (to add it in outlook and send mails on behalf)

## Add Environment

Go to PPAC https://admin.powerplatform.microsoft.com/

Go to "Manage", "New", Select "Sandbox", select a region, give a cool name and head to the advanced settings

image2

Definitely select "Get new features early" and "Add a Dataverse Store" and hit "Next"

image3

For Security Group, select "Open Access", select a cool URL if you want, Enable D365 Apps (you can't at a later point, so better safe then sorry) and hit "Save"

image4

## Sync Mailbox

Now we gonna sync the mailbox to the environment and make all the mails automatically sync to dataverse.

After the environment is provisioned, click on the environment and then "Settings" on the top. Select "Queues" in the settings

image5

Add "New" Queue and provide your custom name and type in the Mail-Address from earlier and hit "Save"

image6

This has also created a new Mailbox in Dataverse automatically, scroll down and click on the linked Mailbox

image7

Click the buttons to Approve and Test&Enable the Mailbox. Refresh after a few minutes and check if the test was successful

image8

## Import the solution

Go to https://make.preview.powerapps.com/ and select your environment from the picker

image9

Head over to "Solutions" and import the provided solution

