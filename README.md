const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');

const app = express();
const PAGE_ACCESS_TOKEN = 'YOUR_PAGE_ACCESS_TOKEN';

app.use(bodyParser.json());

// Verify webhook
app.get('/webhook', (req, res) => {
  const VERIFY_TOKEN = 'your_verify_token';
  const mode = req.query['hub.mode'];
  const token = req.query['hub.verify_token'];
  const challenge = req.query['hub.challenge'];

  if (mode && token === VERIFY_TOKEN) {
    res.status(200).send(challenge);
  } else {
    res.sendStatus(403);
  }
});

// Handle messages
app.post('/webhook', async (req, res) => {
  const body = req.body;

  if (body.object === 'page') {
    body.entry.forEach(async function(entry) {
      const webhookEvent = entry.messaging[0];
      const senderId = webhookEvent.sender.id;
      const message = webhookEvent.message.text;

      await axios.post(`https://graph.facebook.com/v18.0/me/messages?access_token=${PAGE_ACCESS_TOKEN}`, {
        recipient: { id: senderId },
        message: { text: `You said: "${message}"` }
      });
    });

    res.status(200).send('EVENT_RECEIVED');
  } else {
    res.sendStatus(404);
  }
});

app.listen(process.env.PORT || 3000, () => console.log('Bot is running'));
