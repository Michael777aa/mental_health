## **Project: Mental Health VR Assistant with AI + Blockchain + AWS + MongoDB (TypeScript)**

**1. Folder Structure Overview**
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

**2. server.ts**

```bash
import express, { Request, Response } from 'express';
import dotenv from 'dotenv';
import mongoClient from './database/mongoClient';
import { classifyMentalStage } from './ai-api/openaiClient';
import { storeHashToBlockchain } from './blockchain/storeHashToBlockchain';

dotenv.config();
const app = express();
app.use(express.json());

app.post('/analyze', async (req: Request, res: Response) => {
  try {
    const { userId, text } = req.body;
    const result = await classifyMentalStage(text);
    const collection = mongoClient.db('vr_mental_health').collection('sessions');
    const session = { userId, text, result, timestamp: new Date() };
    await collection.insertOne(session);
    const hash = await storeHashToBlockchain(JSON.stringify(session));
    res.json({ success: true, result, hash });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

mongoClient.connect().then(() => {
  app.listen(3000, () => console.log('Server running on port 3000'));
});
```

**3. openaiClient.ts**
```bash
import { OpenAI } from 'openai';
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY! });

export const classifyMentalStage = async (text: string): Promise<string> => {
  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      { role: 'system', content: 'You are a mental health assistant classifying stages.' },
      { role: 'user', content: text }
    ]
  });
  return response.choices[0].message.content;
};
```

**4. storeHashToBlockchain.ts**
```bash
import { ethers } from 'ethers';
import contractAbi from './MentalHealthVerifierABI.json';

const provider = new ethers.JsonRpcProvider(process.env.BLOCKCHAIN_RPC_URL);
const wallet = new ethers.Wallet(process.env.BLOCKCHAIN_PRIVATE_KEY!, provider);
const contractAddress = process.env.CONTRACT_ADDRESS!;
const contract = new ethers.Contract(contractAddress, contractAbi, wallet);

export const storeHashToBlockchain = async (data: string): Promise<string> => {
  const hash = ethers.id(data);
  const tx = await contract.recordHash(hash);
  await tx.wait();
  console.log('Hash stored on blockchain:', hash);
  return hash;
};
```

**5. SmartContract.sol**
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MentalHealthVerifier {
    event HashRecorded(address indexed sender, bytes32 hash);
    mapping(bytes32 => bool) public recordedHashes;

    function recordHash(bytes32 hash) external {
        require(!recordedHashes[hash], "Hash already recorded");
        recordedHashes[hash] = true;
        emit HashRecorded(msg.sender, hash);
    }

    function verifyHash(bytes32 hash) external view returns (bool) {
        return recordedHashes[hash];
    }
}
```

**6. mongoClient.ts**
```bash
import { MongoClient } from 'mongodb';
const client = new MongoClient(process.env.MONGODB_URI!);
export default client;
```

**7. deployContract.ts**
```bash
import { ethers } from 'ethers';
import fs from 'fs';
import path from 'path';
const contractSource = fs.readFileSync(path.join(__dirname, '../blockchain/SmartContract.sol'), 'utf8');

async function deploy() {
  const provider = new ethers.JsonRpcProvider(process.env.BLOCKCHAIN_RPC_URL);
  const wallet = new ethers.Wallet(process.env.BLOCKCHAIN_PRIVATE_KEY!, provider);
  const solc = require('solc');
  const input = {
    language: 'Solidity',
    sources: {
      'SmartContract.sol': {
        content: contractSource
      }
    },
    settings: {
      outputSelection: {
        '*': {
          '*': ['abi', 'evm.bytecode']
        }
      }
    }
  };

  const output = JSON.parse(solc.compile(JSON.stringify(input)));
  const contractFile = output.contracts['SmartContract.sol'].MentalHealthVerifier;
  const abi = contractFile.abi;
  const bytecode = contractFile.evm.bytecode.object;

  const factory = new ethers.ContractFactory(abi, bytecode, wallet);
  const contract = await factory.deploy();
  console.log('Deployed contract address:', contract.address);
}

deploy();
```

**8. vr-client/Assets/Scripts/VoiceCapture.cs**
```bash
using UnityEngine;
using UnityEngine.Networking;
using TMPro;
using System.Collections;

public class VoiceCapture : MonoBehaviour
{
    public TextMeshProUGUI resultText;

    public void OnVoiceInputComplete(string transcript)
    {
        StartCoroutine(SendTextToBackend(transcript));
    }

    IEnumerator SendTextToBackend(string userText)
    {
        string jsonData = JsonUtility.ToJson(new { userId = "demo-user", text = userText });
        UnityWebRequest request = UnityWebRequest.Put("https://your-backend-url.com/analyze", jsonData);
        request.method = UnityWebRequest.kHttpVerbPOST;
        request.SetRequestHeader("Content-Type", "application/json");

        yield return request.SendWebRequest();

        if (request.result == UnityWebRequest.Result.Success)
        {
            string responseText = request.downloadHandler.text;
            resultText.text = "Result: " + responseText;
        }
        else
        {
            resultText.text = "Error: " + request.error;
        }
    }
}

// This script enables speech-based interaction: user speaks → voice converted to text → sent to backend → AI responds.
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



























