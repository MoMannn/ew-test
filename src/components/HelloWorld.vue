<script setup lang="ts">
import { WalletWidget, useAccount } from "@apillon/wallet-vue";
import { parseEther, formatEther } from 'ethers';
import { EmbeddedEthersSigner } from "@apillon/wallet-sdk";
import { ref, watch, onMounted } from 'vue'
const copyStatus = ref('Copy')
const balance = ref(null)
const { info } = useAccount();

defineProps<{
  msg: string
}>()

// Watch for changes in the wallet connection
watch(() => info?.address, (newAddress) => {
  if (newAddress) {
    console.log('Wallet connected:', newAddress)
    getWalletBalance()
  } else {
    balance.value = null
  }
}, { immediate: true })

// Check connection status when component mounts
onMounted(async () => {
  if (info?.address) {
    console.log('Wallet already connected:', info.address)
    await getWalletBalance()
  }
})


const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(info.address)
    copyStatus.value = 'Copied!'
    setTimeout(() => {
      copyStatus.value = 'Copy'
    }, 2000)
  } catch (err) {
    console.error('Failed to copy:', err)
    copyStatus.value = 'Failed!'
    setTimeout(() => {
      copyStatus.value = 'Copy'
    }, 2000)
  }
}

// Create a reusable function to get signer
const getSigner = () => new EmbeddedEthersSigner()

async function getWalletBalance() {
  try {
    if (!info?.address) {
      console.log('No wallet connected')
      return
    }

    const signer = getSigner()
    const provider = await signer.provider
    const rawBalance = await provider.getBalance(info.address)
    console.log(balance)
    balance.value = formatEther(rawBalance)
  } catch (error) {
    console.error('Error fetching balance:', error)
    throw error
  }
}

const signedMessage = ref('')
const signError = ref('')
const signMessage = async () => {
  try {
    const signer = getSigner()
    const message = "hello from apillon"
    const signature = await signer.signMessage(message)
    signedMessage.value = signature
    console.log('Signed message:', signature)
  } catch (error) {
    console.error('Error signing message:', error)
    // Check if it's a user rejection
    if (error.message?.includes('User Rejected Request')) {
      signError.value = 'Message signing was rejected'
    } else {
      signError.value = 'Failed to sign message. Please try again.'
    }
  }
}

// Add these refs for the transfer functionality
const recipientAddress = ref('')
const transferAmount = ref('')
const transferStatus = ref('')
const transferError = ref('')

const transferTokens = async () => {
  try {
    // Clear previous status/error
    transferStatus.value = ''
    transferError.value = ''

    // Input validation
    if (!recipientAddress.value || !transferAmount.value) {
      transferError.value = 'Please provide both address and amount'
      return
    }

    const signer = getSigner()
    
    // Create transaction object
    const tx = {
      to: recipientAddress.value,
      value: parseEther(transferAmount.value.toString()) 
    }

    // Send transaction
    transferStatus.value = 'Sending transaction...'
    const transaction = await signer.sendTransaction(tx)
    
    transferStatus.value = 'Waiting for confirmation...'
    const provider = await signer.provider
    const receipt = await provider.waitForTransaction(transaction.hash)
    
    transferStatus.value = `Transfer successful! Transaction hash: ${receipt.hash}`
    
    // Clear inputs after successful transfer
    recipientAddress.value = ''
    transferAmount.value = ''
    
    // Refresh balance
    await getWalletBalance()
  } catch (error) {
    console.error('Transfer error:', error)
    if (error.message?.includes('User Rejected Request')) {
      transferError.value = 'Transaction was rejected'
    } else if (error.message?.includes('insufficient funds')) {
      transferError.value = 'Insufficient funds for transfer'
    } else {
      transferError.value = 'Failed to complete transfer. Please try again.'
    }
  }
}

</script>

<template>
  <div class="greetings">
    <h3>
      <div class="wallet-container">
        <div class="info-container">
          <div v-if="info.address" style="display: flex; align-items: center; gap: 0.5rem">
            <p style="font-size: 0.8rem">
              Connected Address: {{ info.address.slice(0,4) + '...' + info.address.slice(-4) }}
            </p>
            <svg 
              @click="copyToClipboard"
              title="Copy address" 
              xmlns="http://www.w3.org/2000/svg" 
              width="12" 
              height="12" 
              viewBox="0 0 24 24" 
              fill="none" 
              stroke="black" 
              stroke-width="2" 
              stroke-linecap="round" 
              stroke-linejoin="round"
              style="cursor: pointer;"
            >
              <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
              <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
            </svg>
          </div>
          <p v-else style="font-size: 0.8rem">Not connected</p>
        </div>
        <WalletWidget
          clientId="c0bde6f1-0cf0-46f5-848c-d4c78044bc06"
          :defaultNetworkId="1287"
          :networks="[
            {
              name: 'Moonbase Testnet',
              id: 1287,
              rpcUrl: 'https://rpc.testnet.moonbeam.network',
              explorerUrl: 'https://moonbase.moonscan.io',
            }
          ]"
        />
      </div>
      <div>
        <button @click="getWalletBalance" style="margin-top: 1rem;">Get Wallet Balance</button>
        <p v-if="balance" style="font-size: 0.8rem; margin-top: 0.5rem;">
          Balance: {{ balance }} ETH
        </p>
      </div>
      <div>
        <button @click="signMessage" style="margin-top: 1rem;">Sign Message</button>
        <p v-if="signedMessage" style="font-size: 0.8rem; margin-top: 0.5rem; word-break: break-all;">
          Signed Message: {{ signedMessage }}
        </p>
          <!-- Add error message display -->
       <p v-if="signError" style="font-size: 0.8rem; margin-top: 0.5rem; color: red;">
        {{ signError }}
      </p>
      </div>
    </h3>

     <!-- Add this new section for transfer functionality -->
  <div style="margin-top: 1rem;">
    <h4>Send Tokens</h4>
    <div style="display: flex; flex-direction: column; gap: 0.5rem; max-width: 300px;">
      <input 
        v-model="recipientAddress"
        placeholder="Recipient Address"
        style="padding: 0.5rem;"
      />
      <input 
        v-model="transferAmount"
        type="number"
        step="0.000000000000000001"
        placeholder="Amount"
        style="padding: 0.5rem;"
      />
      <button 
        @click="transferTokens"
        :disabled="!recipientAddress || !transferAmount"
      >
        Send
      </button>
      
      <!-- Status and error messages -->
      <p v-if="transferStatus" style="font-size: 0.8rem; margin-top: 0.5rem; color: green;">
        {{ transferStatus }}
      </p>
      <p v-if="transferError" style="font-size: 0.8rem; margin-top: 0.5rem; color: red;">
        {{ transferError }}
      </p>
    </div>
  </div>
  </div>
</template>

<style >
 .wallet-container {
        position: fixed;
        top: 1rem;
        right: 1rem;
        z-index: 100;

      }
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
