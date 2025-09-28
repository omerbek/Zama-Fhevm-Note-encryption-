# 🔐 Blockchain Note Simulator  

Welcome to **Blockchain Note Simulator** – a demo dApp that shows how notes can be securely “stored” on-chain using hashed values.  

⚠️ **Disclaimer:** This is only a simulation. It does **not** use real Full Homomorphic Encryption (FHE).  

---

## ✏️ How It Works  

### Write a Note  
1. Type your note in the input box (e.g., `My secret password is 123456`).  
2. Click **Encrypt & Save to Blockchain**.  
3. Your note is hashed and stored on the **Sepolia Testnet** via a smart contract.  
4. The contract’s `setNote()` function saves it **permanently on-chain**.  

### Retrieve a Note  
- Click **Retrieve Note Information**.  
- You will **NOT** see the original note.  

👉 Why?  
- Only the **hash** is stored.  
- Original text is never saved.  
- Hashes (SHA-3) are **one-way functions**, impossible to reverse.  

---

## 🔍 Example  

**Your input:**  
```
"Meeting my friend today"
```  

**Stored on blockchain:**  
```
0x8f4a5b3c9d2e1f0a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5
```  

**What you see when retrieving:**  
```
Last updated: 12/25/2024, 14:30:45
```  

---

## 🔄 Simulation vs Real Zama FHE  

**Current Simulation**  
✅ Notes are hashed & secured on-chain  
✅ Nobody can read your note  
❌ You cannot read your own note  
❌ Not real encryption  

**With Real Zama FHE**  
✅ Notes are fully encrypted on-chain  
✅ Nobody can read your note  
✅ You can decrypt with your private key  
✅ True **Homomorphic Encryption**  

---

# 🚀 Zama-FHEVM Note Encryption  

## 📂 Create Project Directory  

```bash
mkdir zama
cd zama
```

## 📥 Clone Zama Template  

```bash
git clone https://github.com/zama-ai/fhevm-hardhat-template
```

## 📦 Install Dependencies & FHEVM SDK  

```bash
npm install
npm install @fhevm @fhevm-web3sdk/fhevm-web3sdk
```

---

## 📝 Smart Contract Development  

Create `contracts/EncryptedNotebook.sol`:  

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@fhevm/lib/TFHE.sol";

contract EncryptedNotebook {
    mapping(address => euint32) private userNotes;
    
    event NoteUpdated(address indexed user, string message);

    function setNote(euint32 calldata encryptedNote) public {
        userNotes[msg.sender] = encryptedNote;
        emit NoteUpdated(msg.sender, "Note encrypted and stored successfully!");
    }

    function getNote() public view returns (euint32) {
        return userNotes[msg.sender];
    }
    
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}
```

---

## ⚙️ Configure Hardhat  

Edit `hardhat.config.js`:  

```js
require("@nomicfoundation/hardhat-toolbox");

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: {
    version: "0.8.20",
    settings: {
      optimizer: { enabled: true, runs: 200 },
    },
  },
  networks: {
    // Zama Testnet
    zamatest: {
      url: "https://devnet.zama.ai",
      chainId: 8009,
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
    // Local development
    localhost: {
      url: "http://127.0.0.1:8545",
      chainId: 31337,
    }
  },
  paths: {
    sources: "./contracts",
    tests: "./test",
    cache: "./cache",
    artifacts: "./artifacts"
  },
};
```

---

## 📜 Create Deployment Script  

`scripts/deploy.js`:  

```js
const { ethers } = require("hardhat");
const { FhevmInstances } = require("@fhevm/web3sdk");

async function main() {
  console.log("🚀 Deploying Zama EncryptedNotebook dApp...");
  
  const EncryptedNotebook = await ethers.getContractFactory("EncryptedNotebook");
  const notebook = await EncryptedNotebook.deploy();
  await notebook.waitForDeployment();

  const address = await notebook.getAddress();
  console.log("✅ EncryptedNotebook deployed at:", address);
  console.log("📝 Tx hash:", notebook.deploymentTransaction().hash);
  
  const instance = await FhevmInstances.createInstance(ethers.provider);
  console.log("🔑 FHEVM Instance created successfully");
  
  return { notebook, instance, address };
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error("❌ Deployment failed:", error);
    process.exit(1);
  });
```

---

## 🔧 Environment Variables  

Create `.env`:  

```bash
# Sepolia RPC URL (Infura or Alchemy)
SEPOLIA_RPC_URL=https://eth-sepolia.g.alchemy.com

# Private Key (0x...)
PRIVATE_KEY=0x2fc6888

# Etherscan API Key (for contract verification)
ETHERSCAN_API_KEY=BBVX99PXCF

# Frontend port
FRONTEND_PORT=5300
```

---

## 🚀 Deploy Contract  

```bash
npx hardhat run scripts/deploy.js --network zamatest
```

---

## 🎨 Frontend Setup  

```bash
mkdir zama-frontend
cd zama-frontend
npx create-react-app .
npm install ethers @fhevm/web3sdk axios
```

## Then edit `src/App.js` with the provided frontend code.  

```bash
import React, { useState, useEffect, useCallback } from 'react';
import { ethers } from 'ethers';
import './App.css';

// SVG Icons
const WalletIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M21 18v1c0 1.1-.9 2-2 2H5c-1.11 0-2-.9-2-2V5c0-1.1.89-2 2-2h14c1.1 0 2 .9 2 2v1h-9c-1.11 0-2 .9-2 2v8c0 1.1.89 2 2 2h9zm-9-2h10V8H12v8zm4-2.5c-.83 0-1.5-.67-1.5-1.5s.67-1.5 1.5-1.5 1.5.67 1.5 1.5-.67 1.5-1.5 1.5z"/>
  </svg>
);

const LockIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M18 8h-1V6c0-2.76-2.24-5-5-5S7 3.24 7 6v2H6c-1.1 0-2 .9-2 2v10c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V10c0-1.1-.9-2-2-2zM12 17c-1.1 0-2-.9-2-2s.9-2 2-2 2 .9 2 2-.9 2-2 2zM15.1 8H8.9V6c0-1.71 1.39-3.1 3.1-3.1 1.71 0 3.1 1.39 3.1 3.1v2z"/>
  </svg>
);

const EyeIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M12 4.5C7 4.5 2.73 7.61 1 12c1.73 4.39 6 7.5 11 7.5s9.27-3.11 11-7.5c-1.73-4.39-6-7.5-11-7.5zM12 17c-2.76 0-5-2.24-5-5s2.24-5 5-5 5 2.24 5 5-2.24 5-5 5zm0-8c-1.66 0-3 1.34-3 3s1.34 3 3 3 3-1.34 3-3-1.34-3-3-3z"/>
  </svg>
);

const NetworkIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zM8 17.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5zM9.5 8c0-1.38 1.12-2.5 2.5-2.5s2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5S9.5 9.38 9.5 8zm6.5 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"/>
  </svg>
);

const ContractIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M14 2H6c-1.1 0-1.99.9-1.99 2L4 20c0 1.1.89 2 1.99 2H18c1.1 0 2-.9 2-2V8l-6-6zm2 16H8v-2h8v2zm0-4H8v-2h8v2zm-3-5V3.5L18.5 9H13z"/>
  </svg>
);

const TimeIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M11.99 2C6.47 2 2 6.48 2 12s4.47 10 9.99 10C17.52 22 22 17.52 22 12S17.52 2 11.99 2zM12 20c-4.42 0-8-3.58-8-8s3.58-8 8-8 8 3.58 8 8-3.58 8-8 8zm.5-13H11v6l5.25 3.15.75-1.23-4.5-2.67z"/>
  </svg>
);

const CheckIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/>
  </svg>
);

const ErrorIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
  </svg>
);

const SpinnerIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor" className="spinner">
    <path d="M12 4V1L8 5l4 4V6c3.31 0 6 2.69 6 6 0 1.01-.25 1.97-.7 2.8l1.46 1.46C19.54 15.03 20 13.57 20 12c0-4.42-3.58-8-8-8zm0 14c-3.31 0-6-2.69-6-6 0-1.01.25-1.97.7-2.8L5.24 7.74C4.46 8.97 4 10.43 4 12c0 4.42 3.58 8 8 8v3l4-4-4-4v3z"/>
  </svg>
);

const EncryptionIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M18 8h-1V6c0-2.76-2.24-5-5-5S7 3.24 7 6v2H6c-1.1 0-2 .9-2 2v10c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V10c0-1.1-.9-2-2-2zm-6 9c-1.1 0-2-.9-2-2s.9-2 2-2 2 .9 2 2-.9 2-2 2zM15.1 8H8.9V6c0-1.71 1.39-3.1 3.1-3.1 1.71 0 3.1 1.39 3.1 3.1v2z"/>
  </svg>
);

const BlockchainIcon = ({ size = 20 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="currentColor">
    <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-1 17.93c-3.94-.49-7-3.85-7-7.93 0-.62.08-1.21.21-1.79L9 15v1c0 1.1.9 2 2 2v1.93zm6.9-2.54c-.26-.81-1-1.39-1.9-1.39h-1v-3c0-.55-.45-1-1-1H8v-2h2c.55 0 1-.45 1-1V7h2c1.1 0 2-.9 2-2v-.41c2.93 1.18 5 4.05 5 7.41 0 2.08-.8 3.97-2.1 5.39z"/>
  </svg>
);

// Contract ABI
const contractABI = [
  {
    "inputs": [],
    "name": "getNote",
    "outputs": [
      {"internalType": "bytes32", "name": "", "type": "bytes32"},
      {"internalType": "uint256", "name": "", "type": "uint256"}
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [{"internalType": "bytes32", "name": "encryptedData", "type": "bytes32"}],
    "name": "setNote",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [],
    "name": "hasNote",
    "outputs": [{"internalType": "bool", "name": "", "type": "bool"}],
    "stateMutability": "view",
    "type": "function"
  }
];

function App() {
  const [account, setAccount] = useState('');
  const [contract, setContract] = useState(null);
  const [note, setNote] = useState('');
  const [userNote, setUserNote] = useState('');
  const [loading, setLoading] = useState(false);
  const [network, setNetwork] = useState('');
  const [status, setStatus] = useState('disconnected');
  const [transactionHash, setTransactionHash] = useState('');
  const [lastUpdated, setLastUpdated] = useState('');

  const contractAddress = process.env.REACT_APP_CONTRACT_ADDRESS || "0x0000000000000000000000000000000000000000";

  // Sepolia network bilgileri
  const sepoliaChainId = '0xaa36a7';

  const connectWallet = async () => {
    if (window.ethereum) {
      try {
        setStatus('connecting');
        
        // Sepolia network'üne geçiş yapmayı dene
        try {
          await window.ethereum.request({
            method: 'wallet_switchEthereumChain',
            params: [{ chainId: sepoliaChainId }],
          });
        } catch (switchError) {
          if (switchError.code === 4902) {
            await window.ethereum.request({
              method: 'wallet_addEthereumChain',
              params: [{
                chainId: sepoliaChainId,
                chainName: 'Sepolia Testnet',
                rpcUrls: ['https://sepolia.infura.io/v3/'],
                blockExplorerUrls: ['https://sepolia.etherscan.io'],
                nativeCurrency: {
                  name: 'Sepolia Ether',
                  symbol: 'ETH',
                  decimals: 18
                }
              }],
            });
          }
        }

        const accounts = await window.ethereum.request({
          method: 'eth_requestAccounts'
        });
        
        setAccount(accounts[0]);
        setStatus('connected');
        initializeContract();
        checkNetwork();
        
      } catch (error) {
        console.error('Wallet connection failed:', error);
        setStatus('error');
        showNotification('Wallet connection failed: ' + error.message, 'error');
      }
    } else {
      setStatus('no_wallet');
      showNotification('Please install MetaMask!', 'error');
    }
  };

  const checkNetwork = async () => {
    if (window.ethereum) {
      const chainId = await window.ethereum.request({ method: 'eth_chainId' });
      setNetwork(chainId === sepoliaChainId ? 'Sepolia' : 'Wrong Network');
    }
  };

  const initializeContract = useCallback(async () => {
    if (!window.ethereum || !account) return;
    
    try {
      const provider = new ethers.BrowserProvider(window.ethereum);
      const signer = await provider.getSigner();
      const notebookContract = new ethers.Contract(contractAddress, contractABI, signer);
      setContract(notebookContract);
      setStatus('ready');
    } catch (error) {
      console.error('Contract initialization failed:', error);
      setStatus('error');
    }
  }, [account, contractAddress]);

  // Basit bir şifreleme fonksiyonu (simülasyon)
  const simpleEncrypt = (text) => {
    return ethers.keccak256(ethers.toUtf8Bytes(text + account + Date.now()));
  };

  const saveNote = async () => {
    if (!contract || !note) return;
    
    setLoading(true);
    setStatus('saving');
    try {
      const encryptedData = simpleEncrypt(note);
      
      const tx = await contract.setNote(encryptedData, {
        gasLimit: 100000
      });
      
      setTransactionHash(tx.hash);
      showNotification('Transaction sent! Waiting for confirmation...', 'info');
      
      const receipt = await tx.wait();
      setLastUpdated(new Date().toLocaleTimeString());
      setStatus('saved');
      showNotification('Note encrypted and saved successfully!', 'success');
      setNote('');
      setTransactionHash(receipt.hash);
      
    } catch (error) {
      console.error('Error saving note:', error);
      setStatus('error');
      showNotification('Error saving note: ' + error.message, 'error');
    }
    setLoading(false);
  };

  const readNote = async () => {
    if (!contract) return;
    
    setLoading(true);
    setStatus('reading');
    try {
      const hasUserNote = await contract.hasNote();
      
      if (hasUserNote) {
        const [encryptedData, timestamp] = await contract.getNote();
        const date = new Date(Number(timestamp) * 1000);
        setUserNote(`Last updated: ${date.toLocaleString()}`);
        setStatus('read');
        showNotification('Note retrieved successfully!', 'success');
      } else {
        setUserNote('No encrypted note found for your address');
        setStatus('no_note');
        showNotification('No note found for this address', 'info');
      }
    } catch (error) {
      console.error('Error reading note:', error);
      setStatus('error');
      showNotification('Error reading note: ' + error.message, 'error');
    }
    setLoading(false);
  };

  const showNotification = (message, type = 'info') => {
    const notification = document.createElement('div');
    notification.className = `notification ${type}`;
    notification.innerHTML = `
      <div class="notification-content">
        <span class="notification-icon">
          ${type === 'success' ? '<CheckIcon />' : 
            type === 'error' ? '<ErrorIcon />' : 
            '<NetworkIcon />'}
        </span>
        <span>${message}</span>
      </div>
    `;
    document.body.appendChild(notification);
    
    setTimeout(() => {
      notification.remove();
    }, 5000);
  };

  const getStatusIcon = () => {
    switch (status) {
      case 'connected': 
      case 'ready': 
      case 'saved': 
      case 'read': 
        return <CheckIcon size={16} />;
      case 'error': 
        return <ErrorIcon size={16} />;
      case 'connecting':
      case 'saving':
      case 'reading':
        return <SpinnerIcon size={16} />;
      default: 
        return <NetworkIcon size={16} />;
    }
  };

  const getStatusColor = () => {
    switch (status) {
      case 'connected':
      case 'ready':
      case 'saved':
      case 'read': 
        return '#10B981'; // green
      case 'error': 
        return '#EF4444'; // red
      case 'no_wallet': 
        return '#F59E0B'; // amber
      default: 
        return '#3B82F6'; // blue
    }
  };

  useEffect(() => {
    if (window.ethereum) {
      window.ethereum.on('chainChanged', (chainId) => {
        window.location.reload();
      });

      window.ethereum.on('accountsChanged', (accounts) => {
        setAccount(accounts[0] || '');
        if (accounts[0]) {
          initializeContract();
        } else {
          setStatus('disconnected');
        }
      });

      // Sayfa yüklendiğinde wallet'ı kontrol et
      const checkConnectedWallet = async () => {
        try {
          const accounts = await window.ethereum.request({ method: 'eth_accounts' });
          if (accounts.length > 0) {
            setAccount(accounts[0]);
            setStatus('connected');
            initializeContract();
            checkNetwork();
          }
        } catch (error) {
          console.error('Error checking connected wallet:', error);
        }
      };

      checkConnectedWallet();
    }
  }, [initializeContract]);

  return (
    <div className="App">
      <div id="notifications"></div>

      <div className="container">
        {/* Header */}
        <header className="header">
          <div className="logo">
            <div className="logo-icon">
              <LockIcon size={32} />
            </div>
            <div>
              <h1>Zama Encrypted Vault</h1>
              <p>Secure Note Storage on Blockchain</p>
            </div>
          </div>
          <div className="network-status">
            <NetworkIcon size={18} />
            <span>{network || 'Not Connected'}</span>
          </div>
        </header>

        {/* Main Content */}
        <main className="main-content">
          {!account ? (
            <div className="connect-section">
              <div className="welcome-card">
                <div className="welcome-header">
                  <LockIcon size={48} />
                  <h2>Welcome to Zama Encrypted Vault</h2>
                </div>
                <p>Store your notes securely on the blockchain with advanced encryption simulation</p>
                
                <div className="feature-grid">
                  <div className="feature-card">
                    <EncryptionIcon size={24} />
                    <h4>Advanced Encryption</h4>
                    <p>SHA-3 simulated encryption for maximum security</p>
                  </div>
                  <div className="feature-card">
                    <BlockchainIcon size={24} />
                    <h4>Blockchain Storage</h4>
                    <p>Decentralized storage on Sepolia testnet</p>
                  </div>
                  <div className="feature-card">
                    <WalletIcon size={24} />
                    <h4>Web3 Ready</h4>
                    <p>Seamless MetaMask integration</p>
                  </div>
                </div>

                <button 
                  onClick={connectWallet} 
                  className="connect-btn primary"
                  disabled={status === 'connecting'}
                >
                  {status === 'connecting' ? (
                    <>
                      <SpinnerIcon size={18} />
                      Connecting...
                    </>
                  ) : (
                    <>
                      <WalletIcon size={18} />
                      Connect MetaMask
                    </>
                  )}
                </button>

                {status === 'no_wallet' && (
                  <div className="warning-message">
                    <ErrorIcon size={18} />
                    <span>MetaMask not detected. Please install MetaMask to continue.</span>
                  </div>
                )}
              </div>
            </div>
          ) : (
            <div className="app-content">
              {/* User Info Card */}
              <div className="user-card">
                <div className="user-info">
                  <div className="user-avatar">
                    <WalletIcon size={24} />
                  </div>
                  <div className="user-details">
                    <h3>Connected Wallet</h3>
                    <p className="wallet-address">{account}</p>
                  </div>
                </div>
                <div className="status-badge" style={{ borderColor: getStatusColor() }}>
                  <span className="status-dot" style={{ backgroundColor: getStatusColor() }}></span>
                  {getStatusIcon()}
                  <span className="status-text">{status.toUpperCase()}</span>
                </div>
              </div>

              {/* Note Input Section */}
              <div className="card">
                <div className="card-header">
                  <LockIcon size={20} />
                  <h3>Encrypt & Save Note</h3>
                </div>
                <div className="card-body">
                  <div className="input-group">
                    <label>Your Secret Note</label>
                    <textarea
                      value={note}
                      onChange={(e) => setNote(e.target.value)}
                      placeholder="Enter your confidential note here... This text will be encrypted and stored securely on the blockchain."
                      className="note-textarea"
                      rows={4}
                      disabled={loading}
                    />
                  </div>
                  <button 
                    onClick={saveNote} 
                    disabled={loading || !note.trim()}
                    className="action-btn primary"
                  >
                    {loading ? (
                      <>
                        <SpinnerIcon size={18} />
                        Encrypting & Saving...
                      </>
                    ) : (
                      <>
                        <LockIcon size={18} />
                        Encrypt & Save to Blockchain
                      </>
                    )}
                  </button>
                  <div className="hint">
                    <NetworkIcon size={14} />
                    <span>Note will be stored on Sepolia testnet with simulated encryption</span>
                  </div>
                </div>
              </div>

              {/* Read Note Section */}
              <div className="card">
                <div className="card-header">
                  <EyeIcon size={20} />
                  <h3>Retrieve Note</h3>
                </div>
                <div className="card-body">
                  <button 
                    onClick={readNote} 
                    disabled={loading}
                    className="action-btn secondary"
                  >
                    {loading ? (
                      <>
                        <SpinnerIcon size={18} />
                        Retrieving...
                      </>
                    ) : (
                      <>
                        <EyeIcon size={18} />
                        Retrieve Note Information
                      </>
                    )}
                  </button>
                  
                  {userNote && (
                    <div className="result-card">
                      <div className="result-icon">
                        <TimeIcon size={20} />
                      </div>
                      <div className="result-content">
                        <h4>Note Storage Information</h4>
                        <p>{userNote}</p>
                      </div>
                    </div>
                  )}
                </div>
              </div>

              {/* Transaction Info */}
              {transactionHash && (
                <div className="card">
                  <div className="card-header">
                    <ContractIcon size={20} />
                    <h3>Latest Transaction</h3>
                  </div>
                  <div className="card-body">
                    <a 
                      href={`https://sepolia.etherscan.io/tx/${transactionHash}`}
                      target="_blank"
                      rel="noopener noreferrer"
                      className="transaction-link"
                    >
                      <ContractIcon size={16} />
                      View on Etherscan: {transactionHash.slice(0, 10)}...{transactionHash.slice(-8)}
                    </a>
                    {lastUpdated && (
                      <div className="timestamp">
                        <TimeIcon size={14} />
                        <span>Last updated: {lastUpdated}</span>
                      </div>
                    )}
                  </div>
                </div>
              )}

              {/* Contract Info */}
              <div className="info-grid">
                <div className="info-card">
                  <div className="info-icon">
                    <ContractIcon size={24} />
                  </div>
                  <div className="info-content">
                    <h4>Contract Address</h4>
                    <p>{contractAddress.slice(0, 10)}...{contractAddress.slice(-8)}</p>
                  </div>
                </div>
                <div className="info-card">
                  <div className="info-icon">
                    <BlockchainIcon size={24} />
                  </div>
                  <div className="info-content">
                    <h4>Blockchain Network</h4>
                    <p>Sepolia Testnet</p>
                  </div>
                </div>
                <div className="info-card">
                  <div className="info-icon">
                    <EncryptionIcon size={24} />
                  </div>
                  <div className="info-content">
                    <h4>Encryption Method</h4>
                    <p>SHA-3 Simulation</p>
                  </div>
                </div>
              </div>
            </div>
          )}
        </main>

        {/* Footer */}
        <footer className="footer">
          <div className="footer-content">
            <p>🔐 Zama Encrypted Vault - Enterprise Grade Security</p>
            <p>Built with React, Ethers.js & Modern Web3 Technologies</p>
          </div>
        </footer>
      </div>
    </div>
  );
}

export default App;
```

**🎨 Prepare src/App.css file**

```bash
/* App.css */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --primary-color: #3B82F6;
  --success-color: #10B981;
  --error-color: #EF4444;
  --warning-color: #F59E0B;
  --text-primary: #1F2937;
  --text-secondary: #6B7280;
  --bg-primary: #FFFFFF;
  --bg-secondary: #F9FAFB;
  --border-color: #E5E7EB;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  color: var(--text-primary);
}

.App {
  min-height: 100vh;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Header */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 0;
  margin-bottom: 30px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 15px;
}

.logo-icon {
  background: rgba(255, 255, 255, 0.1);
  padding: 12px;
  border-radius: 12px;
  backdrop-filter: blur(10px);
}

.logo h1 {
  color: white;
  font-size: 2.2em;
  font-weight: 700;
  margin-bottom: 4px;
}

.logo p {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9em;
}

.network-status {
  background: rgba(255, 255, 255, 0.1);
  padding: 10px 20px;
  border-radius: 25px;
  color: white;
  display: flex;
  align-items: center;
  gap: 8px;
  backdrop-filter: blur(10px);
  font-size: 0.9em;
}

/* Main Content */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

/* Welcome Section */
.connect-section {
  display: flex;
  justify-content: center;
  align-items: center;
  flex: 1;
}

.welcome-card {
  background: var(--bg-primary);
  padding: 50px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 20px 40px rgba(0,0,0,0.1);
  backdrop-filter: blur(10px);
  max-width: 600px;
  width: 100%;
}

.welcome-header {
  margin-bottom: 20px;
}

.welcome-header h2 {
  color: var(--text-primary);
  margin-bottom: 10px;
  font-size: 2em;
  font-weight: 700;
}

.welcome-card > p {
  color: var(--text-secondary);
  margin-bottom: 40px;
  font-size: 1.1em;
  line-height: 1.6;
}

.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}

.feature-card {
  padding: 20px;
  background: var(--bg-secondary);
  border-radius: 12px;
  text-align: center;
}

.feature-card svg {
  color: var(--primary-color);
  margin-bottom: 10px;
}

.feature-card h4 {
  color: var(--text-primary);
  margin-bottom: 8px;
  font-size: 1em;
}

.feature-card p {
  color: var(--text-secondary);
  font-size: 0.85em;
  line-height: 1.4;
}

.warning-message {
  display: flex;
  align-items: center;
  gap: 8px;
  justify-content: center;
  color: var(--warning-color);
  margin-top: 20px;
  font-size: 0.9em;
}

/* Buttons */
.connect-btn, .action-btn {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 15px 30px;
  border: none;
  border-radius: 12px;
  font-size: 1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  text-decoration: none;
  justify-content: center;
  width: 100%;
}

.connect-btn.primary, .action-btn.primary {
  background: linear-gradient(135deg, var(--primary-color), #1D4ED8);
  color: white;
}

.connect-btn.secondary, .action-btn.secondary {
  background: transparent;
  color: var(--primary-color);
  border: 2px solid var(--primary-color);
}

.connect-btn:hover, .action-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.2);
}

.connect-btn:disabled, .action-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

/* App Content */
.app-content {
  display: grid;
  gap: 25px;
  grid-template-columns: 1fr;
}

/* Cards */
.card {
  background: var(--bg-primary);
  border-radius: 16px;
  padding: 25px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.08);
  border: 1px solid var(--border-color);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid var(--border-color);
}

.card-header h3 {
  color: var(--text-primary);
  font-size: 1.3em;
  font-weight: 600;
}

.card-body {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* User Card */
.user-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  padding: 25px;
  border-radius: 16px;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 15px;
}

.user-avatar {
  background: rgba(255,255,255,0.2);
  padding: 12px;
  border-radius: 12px;
}

.user-details h3 {
  margin-bottom: 5px;
  font-size: 1.1em;
}

.wallet-address {
  font-family: 'Monaco', 'Consolas', monospace;
  background: rgba(255,255,255,0.2);
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 0.85em;
}

.status-badge {
  display: flex;
  align-items: center;
  gap: 8px;
  background: white;
  color: var(--text-primary);
  padding: 8px 16px;
  border-radius: 20px;
  border: 2px solid;
  font-size: 0.85em;
  font-weight: 600;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

/* Input Group */
.input-group label {
  display: block;
  margin-bottom: 8px;
  color: var(--text-primary);
  font-weight: 600;
  font-size: 0.9em;
}

.note-textarea {
  width: 100%;
  padding: 15px;
  border: 2px solid var(--border-color);
  border-radius: 12px;
  font-size: 1em;
  font-family: inherit;
  resize: vertical;
  transition: border-color 0.3s;
  background: var(--bg-secondary);
}

.note-textarea:focus {
  outline: none;
  border-color: var(--primary-color);
  background: white;
}

.hint {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--text-secondary);
  font-size: 0.85em;
}

/* Result Card */
.result-card {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  background: var(--bg-secondary);
  padding: 20px;
  border-radius: 12px;
  border-left: 4px solid var(--success-color);
}

.result-icon {
  margin-top: 2px;
}

.result-content h4 {
  color: var(--text-primary);
  margin-bottom: 5px;
  font-size: 1em;
}

.result-content p {
  color: var(--text-secondary);
  font-size: 0.9em;
}

/* Transaction Link */
.transaction-link {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--primary-color);
  text-decoration: none;
  font-family: monospace;
  font-size: 0.9em;
  transition: color 0.3s;
}

.transaction-link:hover {
  color: #1D4ED8;
  text-decoration: underline;
}

.timestamp {
  display: flex;
  align-items: center;
  gap: 6px;
  color: var(--text-secondary);
  font-size: 0.85em;
}

/* Info Grid */
.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.info-card {
  display: flex;
  align-items: center;
  gap: 15px;
  background: var(--bg-secondary);
  padding: 20px;
  border-radius: 12px;
  transition: transform 0.3s;
}

.info-card:hover {
  transform: translateY(-2px);
}

.info-icon {
  background: var(--primary-color);
  color: white;
  padding: 12px;
  border-radius: 10px;
}

.info-content h4 {
  color: var(--text-secondary);
  font-size: 0.85em;
  margin-bottom: 5px;
}

.info-content p {
  color: var(--text-primary);
  font-weight: 600;
  font-size: 0.9em;
}

/* Spinner Animation */
.spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Notifications */
.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  background: white;
  padding: 15px 20px;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  z-index: 1000;
  animation: slideIn 0.3s ease;
  border-left: 4px solid;
}

.notification.success {
  border-left-color: var(--success-color);
}

.notification.error {
  border-left-color: var(--error-color);
}

.notification.info {
  border-left-color: var(--primary-color);
}

.notification-content {
  display: flex;
  align-items: center;
  gap: 10px;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

/* Footer */
.footer {
  text-align: center;
  padding: 40px 0 20px;
  color: rgba(255, 255, 255, 0.8);
  margin-top: 50px;
}

.footer-content p {
  margin: 5px 0;
  font-size: 0.9em;
}

/* Responsive Design */
@media (max-width: 768px) {
  .container {
    padding: 15px;
  }
  
  .header {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }
  
  .logo h1 {
    font-size: 1.8em;
  }
  
  .welcome-card {
    padding: 30px 20px;
  }
  
  .feature-grid {
    grid-template-columns: 1fr;
  }
  
  .user-card {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }
  
  .info-grid {
    grid-template-columns: 1fr;
  }
  
  .card {
    padding: 20px;
  }
  
  .action-btn {
    padding: 12px 20px;
    font-size: 0.9em;
  }
}
```
**Let's goooo Then Start.**
PORT=5300 npm start


✨ Done! You now have a working simulation of **Encrypted Notes on Blockchain with Zama FHEVM template**.  
