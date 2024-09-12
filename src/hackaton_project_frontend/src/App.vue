<script setup>
import { ref, onMounted } from 'vue';
import { hackaton_project_backend, canisterId, idlFactory } from 'declarations/hackaton_project_backend/index';
import { Principal } from '@dfinity/principal';
import { AuthClient } from "@dfinity/auth-client";
import { Actor, HttpAgent } from "@dfinity/agent";

// State management
let amounts = ref(['']);
let accounts = ref(['']);
let transferResults = ref([]);
let newPrincipal = ref('');
let allowedPrincipal = ref('');
let loggedPrincipal = ref('');
let error = ref('');
let errorRunes = ref('');
let successMessageRunes = ref('');
let successMessagePrivilege = ref('');
let loading = ref(false);
let webapp = null;

onMounted(async () => {
  try {
    const principal = await hackaton_project_backend.getAllowedPrincipal();
    allowedPrincipal.value = principal.length === 0 ? 'None' : principal;
  } catch (err) {
    error.value = "Failed to load allowed principal: " + err.message;
  }
});

// Function to add input fields
function addField() {
  amounts.value.push('');
  accounts.value.push('');
}

// Function to remove a field
function removeField(index) {
  amounts.value.splice(index, 1);
  accounts.value.splice(index, 1);
}

// Handle the bulk transfer
async function handleBulkTransfer(e) {
  loading.value = true;
  successMessageRunes.value = '';
  e.preventDefault();
  try {
    const transferArgs = {
      amounts: amounts.value.map((amount) => BigInt(amount)),
      toAccounts: accounts.value.map((account) => ({
        owner: Principal.fromText(account),
        subaccount: []
      })),
    };

    const results = await webapp.bulkTransfer(transferArgs);

    console.log(results);

    transferResults.value = results.map(obj => {
      const keys = Object.keys(obj);
      return keys.length > 0 ? keys[0] : null;
    });

    successMessageRunes.value = 'Transaction successful!';
    errorRunes.value = '';

  } catch (err) {
    errorRunes.value = "Transfer failed: " + err.message;
  } finally {
    loading.value = false;
  }
}

// Handle transferring privileges
async function handleTransferPrivilege(e) {
  successMessagePrivilege.value = '';
  loading.value = true;
  e.preventDefault();
  try {
    const success = await webapp.transferPrivilege(Principal.fromText(newPrincipal.value));
    allowedPrincipal.value = await webapp.getAllowedPrincipal();
    if (success) {
      allowedPrincipal.value = await webapp.getAllowedPrincipal();
      successMessagePrivilege.value = 'Privileges were transferred successfully!';
    } else {
      successMessagePrivilege.value = 'Privileges transfer failed.';
    }
    error.value = '';
  } catch (err) {
    error.value = "Privilege transfer failed: " + err.message;
  } finally {
    loading.value = false;
  }
}

async function handleLogin() {
  try {
    const authClient = await AuthClient.create();

    await authClient.login({
      identityProvider: "http://br5f7-7uaaa-aaaaa-qaaca-cai.localhost:4943/",
      onSuccess: async () => {

        const identity = authClient.getIdentity();
        const principal = identity.getPrincipal().toString();
        const agent = new HttpAgent({ identity });
        agent.fetchRootKey(); // This is needed for local development only

        webapp = Actor.createActor(idlFactory, {
          agent,
          canisterId: canisterId,
        });

        console.log("identity:", identity);
        console.log("principal:", principal);
        loggedPrincipal.value = principal;

        error.value = ""; // Clear any errors
      },
      onError: (err) => {
        error.value = "Login failed: " + err.message;
      }
    });
  } catch (err) {
    error.value = "Login failed: " + err.message;
  }
}

</script>

<template>
  <nav class="top-bar">
    <span v-if="loggedPrincipal">
      Logged in:
      <span class="logged-principal">
        {{ loggedPrincipal }}
      </span>
    </span>
    <button class="login-button" @click="handleLogin" :disabled="loggedPrincipal !== ''">
      {{ loggedPrincipal ? 'Logged' : 'Login' }}
    </button>
  </nav>

  <main>
    <h1>ICP Runes Control Panel</h1>
    <form @submit="handleBulkTransfer">
      <h2>Distribute Runes</h2>
      <div v-for="(amount, index) in amounts" :key="index">
        <div class="input-group">
          <div class="amount-input-group">
            <label>Amount: </label>
            <input v-model="amounts[index]" type="text" required />
          </div>
          <div class="account-input-group">
            <label>To Account: </label>
            <input v-model="accounts[index]" type="text" required />
          </div>
        </div>
        <button type="button" @click="removeField(index)">Remove</button>
      </div>
      <button type="button" @click="addField">Add Another Transfer</button>
      <br /><br />
      <button type="submit" :disabled="loading">
        <span v-if="loading">Processing...</span>
        <span v-else>Submit Transfers</span>
      </button>
    </form>
    <p v-if="errorRunes">{{ errorRunes }}</p>
    <p v-if="successMessageRunes" class="success-message">{{ successMessageRunes }}</p>

    <hr />

    <section>
      <span>Distribution Results</span>
      <ul>
        <li v-for="(result, index) in transferResults" :key="index">{{ result }}</li>
      </ul>
    </section>

  </main>
  <main>

    <h2>Transfer Privilege</h2>
    <form @submit="handleTransferPrivilege">
      <label>New Principal: </label>
      <input v-model="newPrincipal" type="text" required />
      <button type="submit" :disabled="loading">
        <span v-if="loading">Processing...</span>
        <span v-else>Transfer Privilege</span>
      </button>
    </form>

    <p v-if="successMessagePrivilege" class="success-message">{{ successMessagePrivilege }}</p>

    <hr />

    <p>Allowed Principal: {{ allowedPrincipal }}</p>

    <p v-if="error">{{ error }}</p>
  </main>
</template>
