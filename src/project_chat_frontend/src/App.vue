<script lang="ts">
import { ref } from 'vue';
import { canisterId, createActor, project_chat_backend } from '../../declarations/project_chat_backend';
import { AuthClient } from '@dfinity/auth-client';
import { HttpAgent } from '@dfinity/agent';
import type { Identity } from '@dfinity/agent';
import { Principal } from '@dfinity/principal';

export default {
  data() {
    return {
      newChat: "",
      chats: [] as string[][],
      identity: undefined as undefined | Identity,
      principal: undefined as undefined | Principal,
      targetPrincipal: "",
    }
  },
  methods: {
    isUserLoggedIn() {
      if (!this.identity || !this.principal || this.principal === Principal.anonymous()) {
        throw new Error('Please log in');
      }
      return {
        identity: this.identity,
        principal: this.principal,
      }
    },
    validateTargetPrincipal() {
      const cleanTargetPrincipal = this.targetPrincipal.trim();
      if (cleanTargetPrincipal === '') {
        throw new Error('No principal');
      }
      const targetPrincipal = Principal.fromText(cleanTargetPrincipal);
      if (!targetPrincipal || targetPrincipal === Principal.anonymous()) {
        throw new Error('Wrong target');
      }
      return targetPrincipal;
    },
    getAuthClient() {
      this.isUserLoggedIn();
      return createActor(canisterId, {
        agentOptions: {
          identity: this.identity,
        }
      });
    },
    async dodajChatMSG() {
      const targetPrincipal = this.validateTargetPrincipal();
      const backend = this.getAuthClient();
      await backend.add_chat_msg(this.newChat, targetPrincipal);
      await this.pobierzChaty();
    },
    async pobierzChaty() {
      const {identity, principal} = this.isUserLoggedIn();
      const targetPrincipal = this.validateTargetPrincipal();

      const chatPath = [targetPrincipal, identity.getPrincipal()].sort();
      this.chats = await project_chat_backend.get_chat(chatPath);
    },
    async login() {
      const authClient = await AuthClient.create();
      await authClient.login({
        identityProvider: "http://b77ix-eeaaa-aaaaa-qaada-cai.localhost:4943/",
        onSuccess: async () => {
          const identity = authClient.getIdentity();
          this.principal = identity.getPrincipal();
          this.identity = identity;
          console.log("Zalogowano", this.principal);
          await this.pobierzChaty();
        }
      })
    }
  },
}
</script>

<template>
  <main>
    <img src="/logo2.svg" alt="DFINITY logo" />
    <br />
    <br />
    {{ principal }}
    <button @click='login'>Login</button>
    <div>
      <input v-model='targetPrincipal'>
      <button @click='pobierzChaty'>Pobierz chat</button>
    </div>
    <div>
      <div v-for="chat in chats[0]">
        {{ chat }}
      </div>
    </div>
    <div>
      <textarea v-model="newChat"></textarea>
      <button @click="dodajChatMSG">
        Dodaj notatkę
      </button>
    </div>
  </main>
</template>

<!-- const agent = await HttpAgent.create({ identity: this.identity })
      this.backend = createActor(canisterId, { agent }) -->