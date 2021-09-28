🌅 Using window.ethereum()
----------------------

So, in order for our website to talk to the blockchain, we need to somehow connect our wallet to it. Once we connect our wallet to our website, our website will have permission to call smart contracts on our behalf. **Remember, it's just like authenticating into a website.**

Head over Replit and go to `App.js` under `src`, this is where we'll be doing all our work.

If we're logged in to Metamask, it will automatically inject a special object named `ethereum` into our window that has some magical methods. Let's check if we have that first.

```jsx
import './styles/App.css';
import twitterLogo from './assets/twitter-logo.svg';
import React, { useEffect } from "react";

// Constants
const TWITTER_HANDLE = '_buildspace';
const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;
const OPENSEA_LINK = '';
const TOTAL_MINT_COUNT = 50;

const App = () => {

    const checkIfWalletIsConnected = () => {
    /*
    * First make sure we have access to window.ethereum
    */
    const { ethereum } = window;

    if (!ethereum) {
      console.log("Make sure you have metamask!");
      return;
    } else {
      console.log("We have the ethereum object", ethereum);
    }
  }

  /*
  * This runs our function when the page loads.
  */
  useEffect(() => {
    checkIfWalletIsConnected();
  }, [])

  const renderNotConnectedContainer = () => (
    <button className="cta-button connect-wallet-button">
      Connect to Wallet
    </button>
  );

  return (
		// blah blah all your html here
  );
};

export default App;
```

🔒 See if we can access the user's account
----------------------

So when you run this, you should see that line "We have the Ethereum object" printed in the console of the website when you go to inspect it.

**NICE.**

Next, we need to actually check if we're authorized to actually access the user's wallet. Once we have access to this, we can call our smart contract!

Basically, Metamask doesn't just give our wallet credentials to every website we go to. It only gives it to websites we authorize. Again, it's just like logging in! But, what we're doing here is **checking if we're "logged in".** 

Check out the code below.

```jsx
import './styles/App.css';
import twitterLogo from './assets/twitter-logo.svg';
import React, { useEffect, useState } from "react";

// Constants
const TWITTER_HANDLE = '_buildspace';
const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;
const OPENSEA_LINK = '';
const TOTAL_MINT_COUNT = 50;

const App = () => {

    /*
    * Just a state variable we use to store our user's public wallet. Don't forget to import useState.
    */
    const [currentAccount, setCurrentAccount] = useState("");
    
    /*
    * Gotta make sure this is async.
    */
    const checkIfWalletIsConnected = async () => {
      const { ethereum } = window;

      if (!ethereum) {
          console.log("Make sure you have metamask!");
          return;
      } else {
          console.log("We have the ethereum object", ethereum);
      }

      /*
      * Check if we're authorized to access the user's wallet
      */
      const accounts = await ethereum.request({ method: 'eth_accounts' });

      /*
      * User can have multiple authorized accounts, we grab the first one if its there!
      */
      if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
          setCurrentAccount(account)
      } else {
          console.log("No authorized account found")
      }
  }

  useEffect(() => {
    checkIfWalletIsConnected();
  }, [])

  const renderNotConnectedContainer = () => (
    <button className="cta-button connect-wallet-button">
      Connect to Wallet
    </button>
  );

  return (
		 // blah blah all your html here
  );
};

export default App;
```

🛍 Build a connect wallet button
----------------------

When you run the above code, the console.log that prints should be `No authorized account found`. Why? Well because we never explicitly told Metamask, *"hey Metamask, please give this website access to my wallet".*

We need to create a `connectWallet` button. In the world of Web3, connecting your wallet is literally a "Login" button for your user.

Ready for the easiest "login" experience for your life :)? Check it out:

```jsx
import './styles/App.css';
import twitterLogo from './assets/twitter-logo.svg';
import React, { useEffect, useState } from "react";

const TWITTER_HANDLE = '_buildspace';
const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;
const OPENSEA_LINK = '';
const TOTAL_MINT_COUNT = 50;

const App = () => {

    const [currentAccount, setCurrentAccount] = useState("");
    
    const checkIfWalletIsConnected = async () => {
      const { ethereum } = window;

      if (!ethereum) {
          console.log("Make sure you have metamask!");
          return;
      } else {
          console.log("We have the ethereum object", ethereum);
      }

      const accounts = await ethereum.request({ method: 'eth_accounts' });

      if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
          setCurrentAccount(account)
      } else {
          console.log("No authorized account found")
      }
  }

  /*
  * Implement your connectWallet method here
  */
  const connectWallet = async () => {
    try {
      const { ethereum } = window;

      if (!ethereum) {
        alert("Get MetaMask!");
        return;
      }

      /*
      * Fancy method to request access to account.
      */
      const accounts = await ethereum.request({ method: "eth_requestAccounts" });

      /*
      * Boom! This should print out public address once we authorize Metamask.
      */
      console.log("Connected", accounts[0]);
      setCurrentAccount(accounts[0]); 
    } catch (error) {
      console.log(error)
    }
  }

  useEffect(() => {
    checkIfWalletIsConnected();
  }, [])

  /*
  * We added a simple onClick event here.
  */
  const renderNotConnectedContainer = () => (
    <button onClick={connectWallet} className="cta-button connect-wallet-button">
      Connect to Wallet
    </button>
  );

  /*
  * We want the "Connect to Wallet" button to dissapear if they've already connected their wallet!
  */
  const renderMintUI = () => (
    <button onClick={null} className="cta-button connect-wallet-button">
      Mint NFT
    </button>
  )

  /*
  * Added a conditional render! We don't want to show Connect to Wallet if we're already conencted :).
  */
  return (
    <div className="App">
      <div className="container">
        <div className="header-container">
          <p className="header gradient-text">My NFT Collection</p>
          <p className="sub-text">
            Each unique. Each beautiful. Discover your NFT today.
          </p>
          {currentAccount === "" ? renderNotConnectedContainer() : renderMintUI()}
        </div>
        <div className="footer-container">
          <img alt="Twitter Logo" className="twitter-logo" src={twitterLogo} />
          <a
            className="footer-text"
            href={TWITTER_LINK}
            target="_blank"
            rel="noreferrer"
          >{`built on @${TWITTER_HANDLE}`}</a>
        </div>
      </div>
    </div>
  );
};

export default App;
```


🚨Progress report.
------------------------
Post a screenshot of your website in #progress!