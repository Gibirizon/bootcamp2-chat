<script lang="ts">
import { ref } from "vue";
import { bootcamp_chat_backend, canisterId, createActor } from "../../declarations/bootcamp_chat_backend/index";
import { AuthClient } from "@dfinity/auth-client";
import { HttpAgent } from "@dfinity/agent";
import type { Identity } from "@dfinity/agent";
import { Principal } from "@dfinity/principal";
export default {
	data() {
		return {
			newChat: "",
			chats: [] as string[][],
			identity: undefined as undefined | Identity,
			principal: undefined as undefined | Principal,
			targetPricnipal: "",
		};
	},
	methods: {
		isUserLoggedIn() {
			if (!this.identity || !this.principal || this.principal === Principal.anonymous()) {
				throw new Error("Plz log in");
			}
			return {
				identity: this.identity,
				principal: this.principal,
			};
		},
		validateTargetPricnipal() {
			const cleanTargetPrincipal = this.targetPricnipal.trim();
			if (cleanTargetPrincipal === "") {
				throw new Error("Empty target principal");
			}
			const targetPricnipal = Principal.fromText(cleanTargetPrincipal);
			if (!targetPricnipal || targetPricnipal === Principal.anonymous()) {
				throw new Error("Invalid target principal");
			}
			return targetPricnipal;
		},
		getAuthClient() {
			this.isUserLoggedIn();
			return createActor(canisterId, {
				agentOptions: {
					identity: this.identity,
				},
			});
		},
		async dodajChatMSG() {
			const targetPricnipal = this.validateTargetPricnipal();

			const backend = this.getAuthClient();
			await backend.add_chat_msg(this.newChat, targetPricnipal);
			await this.pobierzChaty();
		},
		async pobierzChaty() {
			console.log("pobieram chaty");
			const { identity, principal } = this.isUserLoggedIn();
			const targetPricnipal = this.validateTargetPricnipal();

			const chatPath = [targetPricnipal, identity.getPrincipal()].sort();
			this.chats = await bootcamp_chat_backend.get_chat(chatPath);
			console.log("chats: ", this.chats);
		},
		async login() {
			const authClient = await AuthClient.create();
			await authClient.login({
				identityProvider: "http://be2us-64aaa-aaaaa-qaabq-cai.localhost:4943/",
				onSuccess: async () => {
					const identity = authClient.getIdentity();
					this.principal = identity.getPrincipal();
					this.identity = identity;
					console.log("Zalogowano: ", this.principal);
					await this.pobierzChaty();
				},
			});
		},
	},
};
</script>

<template>
	<main>
		<img src="/logo2.svg" alt="DFINITY logo" />
		<br />
		<br />
		{{ principal }} <button @click="login">Login</button>
		<div>
			<input type="text" v-model="targetPricnipal" />
			<button @click="pobierzChaty">pobierz chaty</button>
		</div>
		<div v-for="chat in chats[0]">
			{{ chat }}
		</div>
		<div>
			<textarea v-model="newChat"></textarea>
			<button @click="dodajChatMSG">Dodaj wiadomość</button>
		</div>
	</main>
</template>
4pkhg-of7q7-ha36i-i66os-n3chk-ggllo-mobmj-xordn-dwbs7-sqnbs-uae
bvcbd-nqpjj-cs5ol-b4ubk-36joe-hlitl-4suhj-w7k6x-uwmkc-2mj5m-sqe
