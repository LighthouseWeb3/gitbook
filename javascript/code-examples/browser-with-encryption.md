# 🔐 Browser Encryption Upload

**Step 1:** **Follow this** [**React documentation**](https://reactjs.org/docs/create-a-new-react-app.html) **and Create a new react app using the following command**&#x20;

```
npx create-react-app lighthouse-app
```

and go into the new repository using

```
cd lighthouse-app
```

**Step 2.1:** **Install the Lighthouse SDK**&#x20;

```
npm i @lighthouse-web3/sdk
```

**Step 2.2:** **Install the ethers library**&#x20;

Also, install the ethers library to trigger wallet-related functions like signing.

```
npm i ethers
```

**Step 3: Copy and paste the following code example into src/App.js file**

Now the following code and be used to push files to Lighthouse from the browser with encryption. You need to have a browser wallet like Metamask to sign from your wallet

**Step 3.1:** Get the API Key from [Lighthouse Files Dapp](https://files.lighthouse.storage/) and insert it into the `uploadEncrypted` function as a parameter below.

Note: for production use, set the API Key to the .env file and don't publish your API Key publically



```javascript
import React, { useState } from "react";
import { ethers } from "ethers";
import lighthouse from "@lighthouse-web3/sdk";

function App() {

  const ConnectWalletButton = () => {
    const [isWalletConnected, setAccount] = useState(null);

    const handleClick = async () => {
      if (window.ethereum) {
        try {
          const accounts = await window.ethereum.request({
            method: "eth_requestAccounts",
          });
          setAccount(accounts[0]);
        } catch (error) {
          console.error(error);
        }
      } else {
        console.error("Wallet is not installed on this browser");
      }
    };
    return (
      <button onClick={handleClick} disabled={isWalletConnected}>
        Connect Wallet
      </button>
    );
  };

  const encryptionSignature = async () => {
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();
    const address = await signer.getAddress();
    const messageRequested = (await lighthouse.getAuthMessage(address)).data
      .message;
    const signedMessage = await signer.signMessage(messageRequested);
    return {
      signedMessage: signedMessage,
      publicKey: address,
    };
  };

  const progressCallback = (progressData) => {
    let percentageDone =
      100 - (progressData?.total / progressData?.uploaded)?.toFixed(2);
    console.log(percentageDone);
  };

  /* Deploy file along with encryption */
  const deployEncrypted = async (e) => {
    /*
       uploadEncrypted(e, publicKey, accessToken, uploadProgressCallback)
       - e: js event
       - publicKey: wallets public key
       - accessToken: your api key
       - signedMessage: message signed by the owner of publicKey
       - uploadProgressCallback: function to get progress (optional)
    */
    const sig = await encryptionSignature();
    const response = await lighthouse.uploadEncrypted(
      e,
      sig.publicKey,
      "YOUR_API_KEY",
      sig.signedMessage,
      progressCallback
    );
    console.log(response);
    /*
      output:
        {
          Name: "c04b017b6b9d1c189e15e6559aeb3ca8.png",
          Size: "318557",
          Hash: "QmcuuAtmYqbPYmPx3vhJvPDi61zMxYvJbfENMjBQjq7aM3"
        }
      Note: Hash in response is CID.
    */
  };

  return (
    <div className="App">
      <input onChange={(e) => deployEncrypted(e)} type="file" />
      <ConnectWalletButton />
    </div>
  );
}

export default App;
```

**Step 4: As a final step, run the following command to view your react site in the browser**

```
npm start
```

You can now upload a file by clicking the upload button and viewing the IPFS hash (CID) to the file in the browser console in the data object, as shown below -&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2023-01-21 at 3.05.41 AM.png" alt=""><figcaption><p>data obect in the browser console</p></figcaption></figure>

The Hash above is the IPFS CID of the encrypted file. If you try downloading it from the public IPFS network, it will be an encrypted file and can be decrypted only by the Authorised wallet address. Check the "Browser Decrypt File" Code example on how to decrypt the file.

**Step 5: View the file**

To view the uploaded file, go to the following link and sign in from your wallet **https://files.lighthouse.storage/viewFile/\<cid>**  (insert cid here)
