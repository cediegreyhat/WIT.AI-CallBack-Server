const express = require('express');
const bodyParser = require('body-parser');
const request = require('request');

const WIT_TOKEN = 'ZZNLYWFMP24Z3C4MAYW2DE5HXI25X2TL';
const FB_PAGE_ACCESS_TOKEN = 'EAANPZBFp0XYQBAPrJVRK3LLPiIHZBDVXmsCDshujVcFibOW8Vd7FOPowDXWBDm2r5H4mtjgO1YZCcgPhzRnqpn5peSGMtXmGMBFWxbLfPZBqSbrvyJQcL3SSLZBaNEYekBeiqefUaZC1Xd3cAKWzZCxgeZBBVwRQzpNt9LaTNe7ldRWLQnFtYn2h';
const FB_VERIFY_TOKEN = 'rikudesu';

const app = express();

app.use(bodyParser.json());

// Facebook Webhook
app.get('/webhook', (req, res) => {
  if (req.query['hub.mode'] === 'subscribe' && req.query['hub.verify_token'] === FB_VERIFY_TOKEN) {
    res.status(200).send(req.query['hub.challenge']);
  } else {
    res.sendStatus(403);
  }
});

// Handle incoming messages from Facebook
app.post('/webhook', (req, res) => {
  const entries = req.body.entry;

  entries.forEach((entry) => {
    const messaging = entry.messaging;

    messaging.forEach((message) => {
      if (message.message && message.message.text) {
        // Send message to Wit.ai for NLU
        const messageText = message.message.text;
        const senderId = message.sender.id;
        const witUrl = `https://api.wit.ai/message?v=2022-02-28&q=${encodeURIComponent(messageText)}`;
        const witHeaders = {
          'Authorization': `Bearer ${WIT_TOKEN}`
        };

        request({
          url: witUrl,
          headers: witHeaders
        }, (err, witRes, witBody) => {
          if (err) {
            console.error(`Error sending message to Wit.ai: ${err}`);
          } else {
            const witData = JSON.parse(witBody);

            // Handle intent from Wit.ai
            if (witData.intents.length > 0) {
              const intent = witData.intents[0].name;
              const confidence = witData.intents[0].confidence;

              // Handle intent based on confidence
              if (confidence > 0.7) {
                switch (intent) {
                  case 'greeting':
                    sendTextMessage(senderId, 'Hello!');
                    break;
                  case 'bye':
                    sendTextMessage(senderId, 'Goodbye!');
                    break;
                  // Add more cases for other intents
                  default:
                    sendTextMessage(senderId, 'I am not sure what you mean.');
                    break;
                }
              } else {
                sendTextMessage(senderId, 'I am not sure what you mean.');
              }
            } else {
              sendTextMessage(senderId, 'I am not sure what you mean.');
            }
          }
        });
      }
    });
  });

  res.sendStatus(200);
});

// Send text message to Facebook Messenger
function sendTextMessage(senderId, messageText) {
  const messageData = {
    text: messageText
  };

  request({
    url: 'https://graph.facebook.com/v13.0/me/messages',
    qs: { access_token: FB_PAGE_ACCESS_TOKEN },
    method: 'POST',
    json: {
      recipient: { id: senderId },
      message: messageData
    }
  }, (err, res, body) => {
    if (err) {
      console.error(`Error sending message to Facebook: ${err}`);
    }
  });
}

// Start server
app.listen(process.env.PORT || 3000, () => {
  console.log('App listening on port 3000!');
});
