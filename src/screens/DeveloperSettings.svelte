<!-- You probably know what this is. -->

<script>
	import Container from "../lib/Container.svelte";

	import {user, modalShown, modalPage, screen, setupPage, disconnected ,mainPage as page, groupcats} from "../lib/stores.js";
	import * as clm from "../lib/clmanager.js";
    import { tick } from "svelte";
    import { Show } from "../lib/Modal";
    import { shiftHeld } from "../lib/keyDetect";
	import Dialog from "../lib/Dialog.js";
</script>

<!--
	<p>Quote: {$user.quote}</p>
	<form 
		class="createpost"
		on:submit|preventDefault={e => {
			//spinner.set(true);
			const _user = $user;
			_user.quote = e.target[0].value;
			user.set(_user);

			clm.updateProfile();
			//spinner.set(false);
		}}
	>
		<input
			type="text"
			class="white"
			placeholder="Write something..."
				id="qinput"
				name="qinput"
			autocomplete="false"
		>
		<button>Save Quote</button>
	</form>
-->
<Container>
	<h1>Developer Settings</h1>
	These should only be used for development purposes.
</Container>
<Container>
	<div class="settings-controls">
		<input
			type="checkbox"
			checked={(localStorage.getItem("SkipDisconnect") == "true") && true || false}
			on:change={() => {
				let bool = !((localStorage.getItem("SkipDisconnect") == "true") && true || false)
				localStorage.setItem("SkipDisconnect", bool.toString());
				console.log("Disconnection modal closing setting changed")
			}}
		>
	</div>

	<h2>Allow closing disconnection modals</h2>
	This allows you to close disconnection modal's if your having issues with fakeout server disconnects.
</Container>
<Container>
	<div class="settings-controls">
		<button
			class="circle settings"
			on:click={() => {
				$modalPage = "clear-data";
				$modalShown = true;
				console.warn("Clear local data modal opened, proceed with caution!")
			}}
		></button>
	</div>

	<h2>Clear local data</h2>
	This should only be used if you're having issues with BetterMeower. This clears saved logins, local settings and preferences, and your Webhooks terms of service agreement.
</Container>
<Container>
	<h2>Open modal</h2>
	Opens a modal.
	<hr>

	<form 
		class="createpost"
		autocomplete="off"
		on:submit|preventDefault={e => {	
			Show(e.target[0].value);
			console.log("Modal opened via developer tools")
		}}
	>
		<input
			type="text"
			class="white"
			placeholder="Modal name"
			name="gtuser"
			autocomplete="false"
			on:keydown={(event) => {
				if (event.key == "Enter" && !shiftHeld) {
					event.preventDefault();
					document.getElementById("submitpost").click();
				}
			}}
		>
		<button id="submitpost">Open Modal</button>
	</form>
</Container>
<Container>
	<h2>Change screen</h2>
	Changes to a certain screen
	<hr>
	<form 
		class="createpost"
		autocomplete="off"
		on:submit|preventDefault={e => {	
			page.set(e.target[0].value)
			console.log("Screen changed via developer tools")
		}}
	>
		<input
			type="text"
			class="white"
			placeholder="Screen name"
			name="gtuser2"
			autocomplete="false"
			on:keydown={(event) => {
				if (event.key == "Enter" && !shiftHeld) {
					event.preventDefault();
					document.getElementById("submitpost2").click();
				}
			}}
		>
		<button id="submitpost2">Open Screen</button>
	</form>
</Container>
<Container>
	<h2>Create Dialog</h2>
	Create a dialog modal
	<hr>
	<form 
		class="createpost"
		autocomplete="off"
		on:submit|preventDefault={e => {	
			Dialog(e.target[0].value,e.target[1].value);
			console.log("Dialog created via developer tools")
		}}
	>
		<input
			type="text"
			class="white"
			placeholder="Dialog name"
			name="dgt"
			autocomplete="Title"
			on:keydown={(event) => {
				if (event.key == "Enter" && !shiftHeld) {
					event.preventDefault();
					document.getElementById("submitpost3").click();
				}
			}}
		>
		<hr>
		<input
			type="text"
			class="white"
			placeholder="Text"
			name="dgd"
			autocomplete="false"
			on:keydown={(event) => {
				if (event.key == "Enter" && !shiftHeld) {
					event.preventDefault();
					document.getElementById("submitpost3").click();
				}
			}}
		>
		<button id="submitpost3">Create dialog</button>
	</form>
</Container>
<Container>
	<div class="settings-controls">
		<button
			class="circle settings"
			on:click={() => {
				cljs.disconnect();
				console.warn("Disconnection from Meower server triggered via developer tools")
			}}
		></button>
	</div>

	<h2>Disconnect</h2>
	Disconnects from the server by calling CLJS.disconnect()
	<br />
	Enable close disconnect modals to continue using Meower (offline) while disconnected.
</Container>
<Container>
	<div class="settings-controls">
		<button
			class="circle settings"
			on:click={async () => {
				cljs.disconnect()
				screen.set("setup");
				disconnected.set(false);
				await tick();
				setupPage.set("reconnect");
				console.log("Reconnection to Meower server triggered via developer tools.")
			}}
		></button>
	</div>

	<h2>Reconnect</h2>
	Reconnects to Meower servers.
</Container>
<Container>
	<h2>Set Meower Server</h2>
	Set to another meower server.
	<br>
	<hr>
	<form 
		class="createpost"
		autocomplete="off"
		on:submit|preventDefault={async e => {	
			localStorage.setItem("meower_linkurl",e.target[0].value+"/")
			localStorage.setItem("meower_apiurl",e.target[1].value+"/")
			cljs.disconnect()
			await tick();
			location.reload();
			console.log("Meower server was changed in developer tools, refreshing...")
		}}
	>
		<input
			type="text"
			class="white"
			placeholder="Server URL"
			name="switchserver"
			autocomplete="false"
			on:keydown={(event) => {
				if (event.key == "Enter" && !shiftHeld) {
					event.preventDefault();
					document.getElementById("submitpost4").click();
				}
			}}
		>
		<input
			type="text"
			class="white"
			placeholder="API URL"
			name="switchapi"
			autocomplete="false"
			on:keydown={(event) => {
				if (event.key == "Enter" && !shiftHeld) {
					event.preventDefault();
					document.getElementById("submitpost4").click();
				}
			}}
		>
		<button id="submitpost4">Connect</button>
	</form>
</Container>
<Container>
	<div class="settings-controls">
		<button
			class="circle settings"
			on:click={async () => {
				localStorage.setItem("meower_linkurl","wss://server-testing.bettermeower.app/")
				localStorage.setItem("meower_apiurl","https://api-testing.bettermeower.app/")
				cljs.disconnect()
				await tick();
				location.reload();
				console.log("Switching to BetterMeower testing server triggered via developer tools")
			}}
		></button>
	</div>

	<h2>Change to testing server</h2>
	Change to the server used in https://testing.bettermeower.app/
</Container>
<Container>
	<div class="settings-controls">
		<button
			class="circle settings"
			on:click={async () => {
				localStorage.removeItem("meower_linkurl")
				localStorage.removeItem("meower_apiurl")
				location.reload();
				console.warn("Server and API urls reset via developer tools")
			}}
		></button>
	</div>

	<h2>Reset server URLS</h2>
	Reset API and server URL's
</Container>
<Container>
	<div class="settings-controls">
		<button
			class="circle settings"
			on:click={async () => {
				Show("Change_store")
			}}
		></button>
	</div>

	<h2>Change Store vars</h2>
	Changes (Some) Vars in store.js
</Container>
<Container>
	<h2>Group cats</h2>
	Changes the amout of group cats
	<hr>
	<form 
		class="createpost"
		autocomplete="off"
		on:submit|preventDefault={e => {	
			groupcats.set(Number(e.target[0].value));
			console.warn("Amount of group cats changed via developer tools, please refresh if anything breaks")
		}}
	>
		<input
			type="text"
			class="white"
			placeholder="enter amount"
			value=200
			name="gcat"
			autocomplete="false"
			on:keydown={(event) => {
				if (event.key == "Enter" && !shiftHeld) {
					event.preventDefault();
					document.getElementById("submitpost5").click();
				}
			}}
		>
		<button id="submitpost5">Set</button>
	</form>
</Container>

<style>
	.settings-controls {
		position: absolute;
		top: 0.25em;
		right: 0.25em;
	}

	input[type="checkbox"], button.circle {
		border: none;
		margin: 0;
		margin-left: 0.125em;
	}
</style>
