# Automatic Birthday Card Sender

The goal of this project is to send automated greeting messages, e.g., a postcard as birthday card. The project is structured very modular and allow for multiple dispatcher (e.g. [Postcard Creator](https://www.post.ch/en/sending-letters/sending-letters/postcard-creator-app) or [Telegram API](https://core.telegram.org/)) as well as for multiple message sources (e.g. fetching data from the [CeviDB](https://github.com/hitobito/hitobito/blob/master/doc/development/05_rest_api.md) or from an CSV list).



## Requirements

- Google Gmail account with rights to send and receive mails. You need to setup a service worker and enable the [Gmail API](https://developers.google.com/gmail/api). At the moment we only plan to include Gmail accounts, but we try to keep the code modular to allow for other mail accounts as well, e.g. using SMTP.
- Server with ability to run docker containers. You can host the container on your own server or using a cloud based platform, e.g. [Google Cloud Run](https://cloud.google.com/run).



### Optional Requirements

- **Postcard Creator:** SwissPost account (setup with the native sign-in method or using SwissID without two-factor authentication). You must enable the Postcard creator API by signing in at least once to the [Postcard Creator App](https://www.post.ch/en/sending-letters/sending-letters/postcard-creator-app).
- **CeviDB fetcher:** You need a Personal OAuth access token to access the address data of the CeviDB.



## Main Components

Each component runs inside a separate Docker Container. 

- **Message Dispatcher:** Runs regularly, checks if new messages have been added to the dispatch queue (stored in the DB). If a greeting should be dispatched today / now, it will send it using the specified dispatcher.

  - Special for the [Postcard Creator Wrapper](https://github.com/abertschi/postcard_creator_wrapper). If multiple cards should be send at the same date, it will postpone cards according to a priority level. The priority of the postponed cards is increased and scheduled for the next day. Cards must be deferred to another day because the free tier only allows for one card to be sent on any given day.

- **Database (SQL + Data Storage):**

  - Message States are stored in an SQL database. Each entry represents a message, it's content, the recipient's address, phone number etc.

  - Images are stored on disk or in a cloud service like the [Google Cloud Bucket](https://cloud.google.com/storage/docs/creating-buckets). You can specify the image save location as argument for the message dispatcher. 

- **Message Source:** Runs regularly or on request: Adds new messages to the dispatch queue using one or multiple data sources (e.g. CeviDB, CSV file). 

  - CeviDB: Runs regularly and checks if anyone in the selected group has it's birthday in the upcoming days. Needs an access token: [Personal OAuth access token](https://github.com/hitobito/hitobito/blob/master/doc/development/05_rest_api.md).

- **Message Updater:** Runs regularly: For every newly added (unsend) message, it sends a preview via an email to the client. 
  You can respond to an email to change the image (including a single new image as attachment) or the text (writing a message in the main body of the mail). Emails will be send as plain text, the subject contains the message id to identify.
  Then checks for new emails and updates the pending messages accordantly.

  

### Optional Components

- **[For Postcard Creator] Card Tracker:** Regularly checks the email account for updates regarding the send status of a given card. You will be notified by email once you card was successfully submitted. You also get an email once a card is printend and ready for delivery. Then updates the status in the database.
- **Grafana Dashboard:** For monitoring send status, logs etc.

