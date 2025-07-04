## **Project: Mental Health VR Assistant with AI + Blockchain + AWS + MongoDB (TypeScript)**

**Folder Structure Overview**
```bash 
mental-vr-assistant/
├── backend/
│   ├── controllers/
│   ├── routes/
│   ├── services/
│   ├── models/
│   ├── utils/
│   └── server.ts
├── ai-api/
│   └── openaiClient.ts
├── blockchain/
│   ├── storeHashToBlockchain.ts
│   └── SmartContract.sol
├── database/
│   └── mongoClient.ts
├── scripts/
│   └── deployContract.ts
├── vr-client/
│   ├── Assets/
│   │   ├── Scripts/
│   │   │   ├── VRInteraction.cs
│   │   │   └── VoiceCapture.cs
│   │   ├── Scenes/
│   │   │   └── MainScene.unity
│   │   └── Prefabs/
│   │       └── DigitalCreature.prefab
│   ├── Packages/
│   ├── ProjectSettings/
│   └── WebGLTemplates/
├── .env
├── tsconfig.json
└── README.md
```

## **It's VR app where users can talk or type their feelings, and a digital creature like an ai therapist**

- listens to them,
- understands their mental state,
- sends that information to a backend,
- uses AI to analyze it,
- saves the result securely,
- and protects it using blockchain

**🔄 FLOW**

**1. User Interaction (Frontend: Unity / WebXR)**
  - The user **types or speaks** in the VR environment.
  - If speaking, your `VoiceCapture.cs` script records voice and turns it into **text** using speech recognition.
  - Unity sends the user’s text message (e.g., “I feel anxious”) to the backend API (`/analyze` endpoint).

📦 Data sent looks like
    ```json
    {"userId": "demo-user", "text": "I feel anxious and can’t sleep"}
    ```

**2. Backend Receives Message (server.ts)**

- The backend server receives the text and:

1. Calls OpenAI to **understand the message**
2. Saves it in **MongoDB**.
3. Creates a **hash** of the message and stores it on the **blockchain** (as proof).

**3. AI Mental Health Analysis (openaiClient.ts)**

- The AI (OpenAI GPT-4) is told:

 > “You are a mental health assistant. Classify this message.”

- It reads the text and gives a result like:

 > “This message shows signs of moderate anxiety.”

🧠 This is your **mental health diagnosis step**.

**4. Save Data Securely (MongoDB)**

- The backend saves this full session in MongoDB:

```json
{
  "userId": "demo-user",
  "text": "I feel anxious and can’t sleep",
  "result": "moderate anxiety",
  "timestamp": "2025-07-04T..."
}
```

**5. Prove It Was Not Changed (Blockchain: Solidity Smart Contract)**

- The backend creates a **hash** of the full session data (like a digital fingerprint).

- Then it calls a **blockchain smart contract** to record that hash.

📦 The blockchain stores:

```bash
0xa73ef4567a...   // unique proof of data
```
This proves the session was recorded and not changed later.

**6. Send Back to User (VR)**

- The server sends back:

 ```json
{
  "success": true,
  "result": "moderate anxiety",
  "hash": "0xa73ef4567a..."
}
```
- Unity receives this and shows:

 > “You seem to have moderate anxiety. Talking to someone may help.”

**🔐 SECURITY CONCEPTS**

- MongoDB stores full data, but it’s secured and private.

- Blockchain stores only a **hash**, so even if someone sees it, they can’t read the message.

- No real personal information is exposed.

**📦 TECH SUMMARY**

| Component  | Technology                      | Purpose                                 |
| ---------- | ------------------------------- | --------------------------------------- |
| Frontend   | Unity + WebXR                   | 3D VR world with voice and text input   |
| Backend    | Node.js + TypeScript            | Handles messages and analysis           |
| AI         | OpenAI GPT-4                    | Understands and classifies mental stage |
| Database   | MongoDB                         | Saves all messages and results          |
| Blockchain | Ethereum (Solidity + Ethers.js) | Verifies integrity of records           |


**🧪 Real Example Flow**

1. 🧍‍♂️ User talks: “I feel tired and stressed.”

2. 🕹️ Unity → converts voice to text → sends to backend.

3. 🧠 OpenAI says: “Signs of early burnout.”

4. 📊 MongoDB saves message and result.

5. 🔐 A hash of that session is stored on blockchain.

6. 📢 User sees: “You may be feeling burnout. Here’s a suggestion…”




## ✅ Conclusion

This project combines **VR**, **AI**, and **blockchain** to create a secure, interactive mental health assistant.  
Users can talk or type, receive real-time emotional insights, and trust their data is safe and untampered.  
It's a simple step toward smarter, more supportive mental wellness tools.



























