
# Ain-js Trigger Router for Ainize
The "Ain-js Trigger Router for Ainize" is an intermediary between the [Ainize](https://github.com/ainize-team/ainize-js?tab=readme-ov-file#deploy) and the [ain-js library](https://github.com/ainblockchain/ain-js?tab=readme-ov-file#function-call). It processes requests and routes events originating from the AI Network blockchain, and forwards them to the models.

You can choose to deploy your models on the [AI Network GPU Service](https://gpu.ainetwork.ai).

![image](/public/sample_structure.png)


## Requirements

node >= 18


## Install

Clone this repository.
```
git clone git@github.com:ainize-team/ain-js-Trigger-Router-for-Ainize.git
```

## Set envs
```JS
BLOCKCHAIN_NETWORK= // AI Network network. mainnet = '1', testnet = '0'. 
PRIVATE_KEY= // AI Network private key to send Transactions for your AI Service.
PORT= // Port number to run this server. (optional, default: 3000)
```
## usage

## Ready to get POST request from AI Network trigger function

A trigger function in AI Network automatically sends POST requests to a specified URL whenever a specific value in the blockchain database changes. For more details, watch this! 👉  [What is AI Network Trigger?](https://docs.ainetwork.ai/ain-blockchain/developer-guide/tools/ainize-trigger)

Requests from trigger functions include complex data structures.
```js
{
  fid: 'function-id',
  function: {
    function_type: 'REST',
    function_url: 'https://function_url.ainetwork.ai/',
    function_id: 'function-id'
  },
  valuePath: [
    'apps',
    'app_name',
    'sub_path',
    '0xaddress...', // Path variable matched with functionPath.
    ...
    ],
  functionPath: [
    'apps',
    'app_name',
    'sub_path',
    '$address', // Path variable name. Start with '$'
    ...
  ],
  value: <ANY_DATA_TO_WRITE_ON_BLOCKCHAIN>,
  ...
  params: {
    address: '0xaddress...' // Path variable.
  },
  ...
  transaction: {
    tx_body: { ... },
    signature: '0xsignature...',
    ...
  },
  ...
}
```

The ain-js Trigger Router for Ainize simplifies handling these requests with built-in utilities.

### Middle ware to check It is from trigger function.
Use the middleware `blockchainTriggerFilter` to verify requests originate from a trigger function.
```JS
import Middleware from './middlewares/middleware';
const middleware = new Middleware();

app.post(
  ...
  middleware.blockchainTriggerFilter,
  ...
)
```
### Extracting the required datas from the request
Easily extract key datas using helper functions.
```JS
import { extractDataFromModelRequest } from './utils/extractor';

const { 
  appName, 
  requesterAddress,
  requestData, 
  requestKey 
} = extractDataFromModelRequest(req);
```

### Connect your inference service.

To integrate your AI service, modify `src/inference.ts`. This file allows you to process the incoming data, format it appropriately, and send requests to your inference service.
```JS
import { Request } from 'express'

export const inference = async (req: Request): Promise<any> =>{
  const { 
    appName, 
    requesterAddress,
    requestData, 
    requestKey 
  } = extractDataFromModelRequest(req);

  ////// Insert your AI Service's Inference Code. //////

  // return the result
}
```

## Example: Simple AI Service Integration

This is a simple example provided to help you understand how to connect an AI service. Modify it according to your specific requirements.
```JS
export const inference = async (req: Request): Promise<any> =>{
  const { 
    appName, 
    requesterAddress,
    requestData, 
    requestKey 
  } = extractDataFromModelRequest(req);

  ////// Insert your AI Service's Inference Code. //////
  const inferenceUrl = process.env.INFERENCE_URL as string; // https://llama_vision.ainize.xyz/chat/completions (sample url. not working)
  const modelName = process.env.MODEL_NAME as string; // meta-llama/Llama-3.2-11B-Vision-Instruct
  const apiKey = process.env.API_KEY as string;

  const prompt = requestData.prompt;

  const response = await fetch(
    inferenceUrl,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`
      },
      body: JSON.stringify({
        model: modelName,
        messages: [
          {
            role: 'user',
            content: prompt
          }
        ]
      })
    }
  )
  if (!response.ok) {
    throw new Error(`Fail to inference: ${JSON.stringify(await response.json())}`);
  }
  const data = await response.json();

  // return the result
  return data.choices[0].message.content;
}

```
