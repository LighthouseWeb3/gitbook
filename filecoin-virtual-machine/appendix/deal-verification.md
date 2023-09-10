# Deal Verification

Deals and file inclusion can be verified using the following steps.

#### Step 1: Use the Contract to Verify:

{% hint style="success" %}
the successful return of calling `complete` (i.e. the call doesn’t revert) means that the deal’s proof is verified successfully by the smart contract and the mapping of the deals between cid and (miner\_id, deal\_id) are updated in the smart contract&#x20;
{% endhint %}

* You can call the **complete function** located here: `0x6ec8722e6543fB5976a547434c8644b51e24785b` to verify the successful inclusion of your in the live filecoin deal. Code for this smart contract can be found [here](https://github.com/lighthouse-web3/raas-starter-kit/blob/main/contracts/DealStatus.sol)
* Submit your proof details, specifically: proofSubtree, proofIndex, and verifierData. <mark style="background-color:yellow;">\[Todo: add code here calling dealstatus.complete () contract function here and show actual response commP and that its not reverting in response]</mark>
* What you'll get in return are two things: "commPa" and "sizePa". Think of these as your file's unique fingerprints.
* These fingerprints (pieceCID and pieceSize) might look like regular text. You'll need to change them into a code-like format, known as hex, before you compare.

{% hint style="info" %}
Use the Bearer Authorization token (signed message) for authenticating API requests. Always renew the signed message if it expires or is invalidated.
{% endhint %}

#### Step 2: Double-Check Your File's Data:

* This verifier data is like a mixed salad of your file combined with others. The first step confirmed that the salad has the right ingredients.
* Now, it's time to be certain that the main ingredient in the salad is indeed your file. Do this by grabbing your file's unique PieceCid and comparing it with the verifier data given.

#### Step 3: Check Live Deal on Explorer:

* You're almost done! You've just ensured your file is part of a bigger bundle handed to a miner.
* Need more peace of mind? Head on over to [FilFox](https://calibration.filfox.info/en/deal/133652). Here, make sure the package's unique tag (the piece cid) aligns with what you've been provided earlier.
