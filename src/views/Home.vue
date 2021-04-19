<template>
  <img alt="Vue logo" src="../assets/logo.png" />
  <div class="home">
    <button @click="fetchGreeting">Fetch Greeting</button>
    <button @click="setGreeting">Set Greeting</button>
    <input v-model="greeting" placeholder="Set greeting" />
  </div>
</template>

<script lang="ts">
// @ts-nocheck
import { defineComponent, ref } from "vue";
import { ethers } from "ethers";
import Greeter from "../artifacts/contracts/Greeter.sol/Greeter.json";

export default defineComponent({
  name: "Home",
  setup() {
    // NOTE Key commands:
    // npx hardhat node
    // npx hardhat run scripts/deploy.js --network localhost
    // Let's store Greeter Address we got from npx hardhat ??
    const greeterAddress = "0x5fbdb2315678afecb367f032d93f642f64180aa3";
    const greeting = ref<string>("");

    async function setGreeting() {
      // Check that user typed/entered a greeting
      if (!greeting.value) return;
      if (typeof window.ethereum !== "undefined") {
        await requestAccount();
        // Create another provider so we can make a transaction to the actual blockchain
        // For this we need a legit signer
        const provider = new ethers.providers.Web3Provider(window.ethereum);
        const signer = provider.getSigner();
        // Create a new instance of the contract but with the signer this time
        const contract = new ethers.Contract(
          greeterAddress,
          Greeter.abi,
          signer
        );
        const transaction = await contract.setGreeting(greeting.value);
        // Commit the actual transaction to the blockchain
        await transaction.wait();
        fetchGreeting();
      }
    }

    async function requestAccount() {
      // Request account info from Metamask wallet. Prompts user to connect
      // Can return an Array of accounts as well if you'd like
      await window.ethereum.request({ method: "eth_requestAccounts" });
    }

    async function fetchGreeting() {
      // Check whether Metamask Extension is connected
      // window.ethereum will be injected into window if so
      if (typeof window.ethereum !== "undefined") {
        // Create a new provider
        const provider = new ethers.providers.Web3Provider(window.ethereum);
        // Create a new instance of the contract
        const contract = new ethers.Contract(
          greeterAddress,
          Greeter.abi,
          provider
        );
        try {
          const data = await contract.greet();
          console.log("data: ", data);
        } catch (err) {
          console.log("Error: ", err);
        }
      }
    }

    return { greeterAddress, greeting, fetchGreeting, setGreeting };
  },
});
</script>

<style scoped>
.home {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-content: center;
  align-items: center;
}
</style>
