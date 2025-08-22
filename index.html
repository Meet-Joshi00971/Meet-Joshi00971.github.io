import express from "express";
import axios from "axios";
import { appendToSheet, readSheet } from "./google-sheets-api.js";

const app = express();
app.use(express.json());

app.get('/', (req, res) => {
  const { 'hub.mode': mode, 'hub.challenge': challenge, 'hub.verify_token': token } = req.query;

  if (mode === 'subscribe' && token === verifyToken) {
    console.log('WEBHOOK VERIFIED');
    res.status(200).send(challenge);
  } else {
    res.status(403).end();
  }
});

// Route for POST requests
app.post('/', (req, res) => {
  const timestamp = new Date().toISOString().replace('T', ' ').slice(0, 19);
  console.log(`\n\nWebhook received ${timestamp}\n`);
  console.log(JSON.stringify(req.body, null, 2));
  res.status(200).end();
});

const { WEBHOOK_VERIFY_TOKEN, GRAPH_API_TOKEN, PORT } = process.env;

// --- In-memory Store ---
let productList = [];
const userState = new Map();

// --- Image URL Mapping ---
// You MUST replace these placeholder URLs with your actual converted Google Drive URLs
const productImages = {
  "Pure Nicotine USP/ EP 99.5+": "https://drive.google.com/uc?id=1e5U5tEFnQ9xmou9JAvpKbQ44qL-W5k3A",
  "Nicotine Salt": "https://drive.google.com/uc?id=1a-pTJY03_w6ovDF9dm_72N1_kYU68T2V",
  "Nicotine Bitartrate Dihydrate": "https://drive.google.com/uc?id=1WVwRDa6gJRW8a8ZZv6Tg6C3r4gfSkvAK",
  "Nicotine Dilutions (VG/PG)": "https://drive.google.com/uc?id=1ppaCnyHTXSvKWse4bk5CKeSTQ-2NrRIB",
  "Nicotine Polacrilex USP / EP": "https://drive.google.com/uc?id=1YKxwPiytirOno8Di0Ybmc7O8wLIB_glY",
};

// --- Main Webhook Logic ---
app.post("/webhook", async (req, res) => {
  const body = req.body;
  const message = body.entry?.[0]?.changes?.[0]?.value?.messages?.[0];
  const phone_number_id =
    body.entry?.[0]?.changes?.[0]?.value?.metadata?.phone_number_id;

  if (!message) return res.sendStatus(200);

  const from = message.from;
  const messageId = message.id;

  let msgBody;
  if (message.type === "text") {
    msgBody = message.text.body;
  } else if (message.type === "interactive" && message.interactive.type === "button_reply") {
    msgBody = message.interactive.button_reply.id;
  } else {
    return res.sendStatus(200);
  }
  
  const lowerCaseMsgBody = msgBody.toLowerCase();
  const currentUserState = userState.get(from);

  // ✅ --- NEW: Handle a "done" command to end the session at any time ---
  if (currentUserState && (lowerCaseMsgBody === 'done' || lowerCaseMsgBody === 'stop')) {
    // Save the last viewed product
    const lastViewedProduct = productList[currentUserState.productIndex];
    currentUserState.data.products = lastViewedProduct.name;
    
    await sendTextMessage(phone_number_id, from, "Thank you! Your enquiry has been saved. Our team will get back to you shortly.");
    await appendToSheet(currentUserState.data);
    userState.delete(from); // End conversation
    
    await markMessageAsRead(phone_number_id, messageId);
    return res.sendStatus(200);
  }


  if (!currentUserState) {
    if (lowerCaseMsgBody.includes("hello")) {
      userState.set(from, {
        current_question: "ask_name",
        data: { phoneNumber: from },
        productIndex: 0,
      });
      await axios.post(
      `https://graph.facebook.com/v19.0/${phone_number_id}/messages`,
       {messaging_product: "whatsapp",
       to: from,
       type: "template",
       template: {name:"welcome_custom", language:{code:"en_US"}}},
      { headers: { Authorization: `Bearer ${GRAPH_API_TOKEN}` } }
    );
  }
  } else {
    // We are in an active conversation
    switch (currentUserState.current_question) {
      case "ask_name":
        currentUserState.data.fullName = msgBody;
        currentUserState.current_question = "ask_country";
        await sendTextMessage(phone_number_id, from, "Thank you! Which country are you from?");
        break;
      case "ask_country":
        currentUserState.data.country = msgBody;
        currentUserState.current_question = "ask_company";
        await sendTextMessage(phone_number_id, from, "Great. What is your company name? (You can type 'N/A' if not applicable)");
        break;
      case "ask_company":
        currentUserState.data.companyName = msgBody;
        currentUserState.current_question = "ask_email";
        await sendTextMessage(phone_number_id, from, "What is your E-mail ID?");
        break;
      case "ask_email":
        currentUserState.data.email = msgBody;
        currentUserState.current_question = "product_menu";
        // ✅ --- NEW: Updated instructions for the user ---
        await sendTextMessage(phone_number_id, from, "Thank you. Please browse our products below. Type 'done' when you are finished.");
        await sendProductMessage(phone_number_id, from, 0); // Show the first product
        break;
      case "product_menu":
        // This case now only handles the next/prev buttons
        if (lowerCaseMsgBody === 'next_product' || lowerCaseMsgBody === 'prev_product') {
          let newIndex = currentUserState.productIndex;
          if (lowerCaseMsgBody === 'next_product') {
            newIndex = (currentUserState.productIndex + 1) % productList.length;
          } else {
            newIndex = (currentUserState.productIndex - 1 + productList.length) % productList.length;
          }
          currentUserState.productIndex = newIndex;
          await sendProductMessage(phone_number_id, from, newIndex);
        }
        // ✅ --- REMOVED: The 'select_product' logic is gone ---
        break;
    }
    // Update the state
    userState.set(from, currentUserState);
  }

  await markMessageAsRead(phone_number_id, messageId);
  res.sendStatus(200);
});

// --- Helper Functions (No changes below) ---
async function sendProductMessage(phoneNumberId, to, productIndex) {
  const product = productList[productIndex];
  if (!product) return;
  const imageUrl = productImages[product.name];
  if (!imageUrl) {
    console.error(`Image URL not found for product: ${product.name}`);
    return;
  }
  const messageData = {
    messaging_product: "whatsapp",
    to,
    type: "template",
    template: {
      name: "product_display_template",
      language: { code: "en_US" },
      components: [
        { type: "header", parameters: [{ type: "image", image: { link: imageUrl } }] },
        { type: "body", parameters: [{ type: "text", text: product.name }, { type: "text", text: product.description }] },
      ],
    },
  };
  await sendMessage(phoneNumberId, messageData);
}

async function sendTextMessage(phoneNumberId, to, text) {
  const messageData = {
    messaging_product: "whatsapp",
    to,
    type: "text",
    text: { body: text },
  };
  await sendMessage(phoneNumberId, messageData);
}

async function sendMessage(phoneNumberId, messageData) {
  try {
    await axios.post(
      `https://graph.facebook.com/v19.0/${phoneNumberId}/messages`,
      messageData,
      { headers: { Authorization: `Bearer ${GRAPH_API_TOKEN}` } }
    );
  } catch (error) {
    console.error("Error sending message:", error.response?.data || error.message);
  }
}

async function markMessageAsRead(phoneNumberId, messageId) {
  try {
    await axios.post(
      `https://graph.facebook.com/v19.0/${phoneNumberId}/messages`,
      { messaging_product: "whatsapp", status: "read", message_id: messageId },
      { headers: { Authorization: `Bearer ${GRAPH_API_TOKEN}` } }
    );
  } catch (error) {
    console.error("Error marking message as read:", error.response?.data);
  }
}

// --- Server Setup ---
app.get("/webhook", (req, res) => {
  const mode = req.query["hub.mode"];
  const token = req.query["hub.verify_token"];
  const challenge = req.query["hub.challenge"];
  if (mode === "subscribe" && token === WEBHOOK_VERIFY_TOKEN) {
    res.status(200).send(challenge);
  } else {
    res.sendStatus(403);
  }
});

async function startServer() {
  try {
    productList = await readSheet();
    if (productList.length > 0) {
      console.log(`Loaded ${productList.length} products from Google Sheet.`);
    }
    app.listen(PORT || 3000, () => {
      console.log(`Server is listening on port: ${PORT || 3000}`);
    });
  } catch (error) {
    console.error("Failed to initialize server:", error);
    process.exit(1);
  }
}

startServer();
