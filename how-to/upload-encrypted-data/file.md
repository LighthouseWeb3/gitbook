---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 📁 File

This example demonstrates uploading files to Lighthouse, which are encrypted at the user's end while it's getting uploaded. The encryption key is split into five parts using BLS cryptography and stored independently in five nodes.

{% tabs %}
{% tab title="JS SDK" %}
#### Method 1: Node JS

```javascript
import {ethers} from "ethers"
import lighthouse from '@lighthouse-web3/sdk'
import kavach from "@lighthouse-web3/kavach"

const signAuthMessage = async(privateKey) =>{
  const signer = new ethers.Wallet(privateKey)
  const authMessage = await kavach.getAuthMessage(signer.address)
  const signedMessage = await signer.signMessage(authMessage.message)
  const { JWT, error } = await kavach.getJWT(signer.address, signedMessage)
  return(JWT)
}

const uploadEncrypted = async() =>{
  /**
   * This function lets you upload a file to Lighthouse with encryption enabled.
   * 
   * @param {string} path - Location of your file.
   * @param {string} apiKey - Your unique Lighthouse API key.
   * @param {string} publicKey - User's public key for encryption.
   * @param {string} signedMessage - A signed message or JWT used for authentication at encryption nodes.
   * 
   * @return {object} - Returns details of the encrypted uploaded file.
   */
  
  const pathToFile = '/home/cosmos/Desktop/wow.jpg'
  const apiKey = 'YOUR_API_KEY_HERE'
  const publicKey = 'YOUR_PUBLIC_KEY_HERE'
  const privateKey= 'YOUR_PRIVATE_KEY_HERE'
  const signedMessage = await signAuthMessage(privateKey)
  
  const response = await lighthouse.uploadEncrypted(pathToFile, apiKey, publicKey, signedMessage)
  console.log(response)
  /* Sample Response
  {
    data: [
      {
        Name: 'decrypt.js',
        Hash: 'QmeLFQxitPyEeF9XQEEpMot3gfUgsizmXbLha8F5DLH1ta',
        Size: '1198'
      }
    ]
  }
  */
}

uploadEncrypted()
```

#### Method 2: Browser

```javascript
import React, { useState } from "react"
import lighthouse from "@lighthouse-web3/sdk"

function App() {
  const [file, setFile] = useState(null)

  // Define your API Key (should be replaced with secure environment variables in production)
  const apiKey = process.env.REACT_APP_API_KEY

//progressCallback in percentage
  const progressCallback = (progressData) => {
    let percentageDone =
      100 - (progressData?.total / progressData?.uploaded)?.toFixed(2)
    console.log(percentageDone)
  }

  // Function to sign the authentication message using Wallet
  const signAuthMessage = async () => {
    if (window.ethereum) {
      try {
        const accounts = await window.ethereum.request({
          method: "eth_requestAccounts",
        })
        if (accounts.length === 0) {
          throw new Error("No accounts returned from Wallet.")
        }
        const signerAddress = accounts[0]
        const { message } = (await lighthouse.getAuthMessage(signerAddress)).data
        const signature = await window.ethereum.request({
          method: "personal_sign",
          params: [message, signerAddress],
        })
        return { signature, signerAddress }
      } catch (error) {
        console.error("Error signing message with Wallet", error)
        return null
      }
    } else {
      console.log("Please install Wallet!")
      return null
    }
  }

  // Function to upload the encrypted file
  const uploadEncryptedFile = async () => {
    if (!file) {
      console.error("No file selected.")
      return
    }

    try {
      // This signature is used for authentication with encryption nodes
      // If you want to avoid signatures on every upload refer to JWT part of encryption authentication section
      const encryptionAuth = await signAuthMessage()
      if (!encryptionAuth) {
        console.error("Failed to sign the message.")
        return
      }

      const { signature, signerAddress } = encryptionAuth

      // Upload file with encryption
      const output = await lighthouse.uploadEncrypted(
        file,
        apiKey,
        signerAddress,
        signature,
        progressCallback
      )
      console.log("Encrypted File Status:", output)
      /* Sample Response
        {
          data: [
            Hash: "QmbMkjvpG4LjE5obPCcE6p79tqnfy6bzgYLBoeWx5PAcso",
            Name: "izanami.jpeg",
            Size: "174111"
          ]
        }
      */
      // If successful, log the URL for accessing the file
      console.log(
        `Decrypt at https://decrypt.mesh3.network/evm/${output.data[0].Hash}`
      )
    } catch (error) {
      console.error("Error uploading encrypted file:", error)
    }
  }

  // Function to handle file selection
  const handleFileChange = (e) => {
    const selectedFile = e.target.files
    if (selectedFile) {
      setFile(selectedFile)
    }
  }

  return (
    <div className="App">
      <input type="file" onChange={handleFileChange} />
      <button onClick={uploadEncryptedFile} disabled={!file}>
        Upload Encrypted File
      </button>
    </div>
  )
}

export default App
```
{% endtab %}

{% tab title="CLI" %}
```bash
lighthouse-web3 upload-encrypted <path>
```
{% endtab %}
{% endtabs %}
