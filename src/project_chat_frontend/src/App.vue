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
      principalText: "",
      targetPrincipal: "",
    }
  },
  methods: {
    async dodajChatMSG() {
      if (!this.identity || this.identity.getPrincipal() === Principal.anonymous()) {
        throw new Error('Please log in')
      }
      const targetPrincipal = Principal.fromText(this.targetPrincipal)
      if (!targetPrincipal || targetPrincipal === Principal.anonymous()){
        throw new Error('Wrong target')
      }

      const backend = createActor(canisterId, {
        agentOptions: {
          identity: this.identity
        }
      });
      await backend.add_chat_msg(this.newChat, targetPrincipal)
      await this.pobierzChaty()
    },
    async pobierzChaty() {
      if (!this.identity || this.identity.getPrincipal() === Principal.anonymous()) {
        throw new Error('Please log in')
      }
      const targetPrincipal = Principal.fromText(this.targetPrincipal)
      if (!targetPrincipal || targetPrincipal === Principal.anonymous()) {
        throw new Error('Wrong target')
      }
      this.chats = await project_chat_backend.get_chat(this.identity.getPrincipal(), targetPrincipal)
    },
    async login() {
      const authClient = await AuthClient.create();
      await authClient.login({
        identityProvider: "http://b77ix-eeaaa-aaaaa-qaada-cai.localhost:4943/"
      })

      const identity = authClient.getIdentity();
      this.principalText = identity.getPrincipal().toText()
      console.log("Zalogowano", this.principalText)
      this.identity = identity;
      await this.pobierzChaty()
    }
  },
  mounted() {
    this.pobierzChaty()
  }
}
</script>

<template>
  <main>
    <img src="/logo2.svg" alt="DFINITY logo" />
    <br />
    <br />
    {{ principalText }}
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