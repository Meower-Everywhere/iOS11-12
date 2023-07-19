<!--
	The home page!
	It features live post updates and a load more button which is pretty nice.
-->

<script>
	import {auth_header, ulist, spinner, lastTyped, mainPage as page, modalPage, modalShown, disconnected, user} from "../lib/stores.js";
	import {shiftHeld} from "../lib/keyDetect.js";
	import Post from "../lib/Post.svelte";
	import Container from "../lib/Container.svelte";
	import Loading from "../lib/Loading.svelte";
	import TypingIndicator from "../lib/TypingIndicator.svelte";
	import {link} from "../lib/clmanager.js";
	import {apiUrl, encodeApiURLParams} from "../lib/urls.js";

	import * as emoji from "node-emoji";

	// @ts-ignore
	import {autoresize} from "svelte-textarea-autoresize";

	import {fly} from "svelte/transition";
	import {flip} from 'svelte/animate';
    import Dialog from "../lib/Dialog.js";
    import { playNotification } from "../lib/sounds.js";
    import Sidebar from "./Sidebar.svelte";
    //import AddMember from "src/lib/modals/AddMember.svelte";
	//Zed just told me the cl4 port will move the mod panel to a seperate site

	let id = 0;
	export let posts = [];

	let pagesLoaded = 0;
	let pageLoading = false;
	let numPages = null;
	let postErrors = "";

	// As we use a Load More button and the home is sorted newest-first,
	// we need an offset for posts to be continuous.
	let postOffset = 0;

	let postInput;

	/**
	 * Loads a page, with offset and overflow calculations.
	 * 
	 * @param {number} [page] The page to load. If not present, simply clears the posts.
	 * @returns {Promise<array>} The posts array.
	 */
	async function loadPage(page) {
		pageLoading = true;
		if (page === undefined) {
			posts = [];
		} else {
			// 25 posts per page...
			let realPage = page + Math.floor(postOffset / 25);
			let realOffset = postOffset % 25;

			try {
				let path = `home?autoget&page=`;
				if (encodeApiURLParams) path = encodeURIComponent(path);
				const resp = await fetch(
					`${apiUrl}${path}${realPage}`,
					{headers: $auth_header}
				);
				if (!resp.ok) {
					throw new Error("Response code is not OK; code is " + resp.status);
				}
				const json = await resp.json();

				let realPosts = json.autoget;
				realPosts.splice(0, realOffset);

				numPages = json.pages;

				let overflowResp, overflowJson;
				if (realOffset > 0 && pagesLoaded < numPages) {
					overflowResp = await fetch(
						`${apiUrl}${path}${realPage+1}`
					);
					if (!resp.ok) {
						throw new Error("Overflow response code is not OK; code is " + resp.status);
					}
					overflowJson = await overflowResp.json();

					realPosts = realPosts.concat(
						overflowJson.autoget.slice(
							0, realOffset
						)
					);
				}

				for (const post of realPosts) {
					posts.push({
						id: id++,
						post_id: post.post_id,
						user: post.u,
						content: post.p,
						date: post.t.e,
					});
				}
				pagesLoaded = page;
			} catch(e) {
				pageLoading = false;
				throw e;
			}
		}
		posts = posts;
		pageLoading = false;
		return posts;
	}

	/**
	 * Adds a post to the list.
	 * 
	 * @param {object} post
	 */
	function addPost(post) {
		posts.unshift({
			id: id++,
			...post
		});
		posts = posts;
	}

	function post(url = '', data = {}) {
		// Default options are marked with *
		fetch(url, {
			method: 'POST',
			//mode: 'no-cors',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify(data)
		});
	}

	/**
	 * Adds events to listen for live post updates.
	 */
	function listenOnLink() {
		link.on("direct", cmd => {
			if ($page === "home" && cmd.val.mode === 1) {
				if (!(cmd.val.post_origin === "home")) return;
				if (cmd.val.p.includes("@"+$user.name)) {
					playNotification()
				}
				addPost({
					post_id: cmd.val._id,
					user: cmd.val.u,
					content: cmd.val.p,
					date: cmd.val.t.e,
				});
				postOffset++;
				posts = posts;
			}
			if (cmd.val.mode === "delete") {
				posts = posts.filter(post => post.post_id !== cmd.val.id);
			}
		});
	}
	if (link.ws.readyState === 1) {
		listenOnLink();
	} else {
		link.ws.addEventListener("open", listenOnLink);
	}

	// Replace "dev" in variable name with release when releasing as main
	// Change variable name to name relevant with the news page's subject.
	if (localStorage.getItem("classic-release") === null) {
							$modalPage = "NewsModal";
							$modalShown = true;
							spinner.set(false);
						}

	let _ulist = $ulist;
	ulist.subscribe(val => {
		_ulist = val;
	})


</script>

<div class="home">
	<!--
		 How do i use webhooks

		send a post request to webhooks.meower.org
		with the json 
		json
		"post":"some_post"
		

		add a username peram to get use non guest mode
	-->
	{#await loadPage(1)}
		<div class="fullcenter">
			<Loading />
		</div>
	{:then}
		<Container>
			<div class="settings-controls">
				<button on:click={()=>page.set("search")} class="search circle">
					<!-- <img
						src={search}
						alt="search"
						width="65%"
						height="65%"
						draggable={false}
					/> -->
				</button>
			</div>
			<!--<div class="settings-controls">
				<button
					class="circle settings"
					on:click={()=>{
						alert("10% finished Mod Panel (That is also unrestricted)")
						page.set("Mod_Panel")
					}}
				>
			</div>-->
			<!--Zed just told me the cl4 port will move the mod panel to a seperate site-->
			<h1>Home</h1>
			{#if $user.name}
				Welcome back <b>@{$user.name}!</b>
			{:else}
				<b>Hello guest</b>! To sign in or create an account, click the profile button on the sidebar.
			{/if}
		</Container>
		<Container>
			{#if _ulist.length === 0}
				{#if $disconnected}
					Hmmm, looks like you were disconnected from the server.
					<br>
					There are currently 0 users online
				{:else}
					Stop playing in dev tools,
					<br>
					There are currently 0 users online
				{/if}
			{:else}
				<b>Users Online:</b>{#if _ulist.length}{" "}{_ulist.join(", ")}{/if}.
				<br>
				There are currently <b>{_ulist.length}</b> users online
			{/if}
		</Container>
		<!--<Container>
		<h2>Search</h2>
		<h4>Search Meower for posts, users, and stats.</h4>
			<div class="settings-controls">
				<button on:click={()=>page.set("search")} class="search circle">
					<img
						src={search}
						alt="search"
						width="65%"
						height="65%"
						draggable={false}
					/>
				</button>
			</div>
		</Container>-->
		<!-- svelte-ignore missing-declaration -->
		<form 
			class="createpost"
			autocomplete="off"
			on:submit|preventDefault={e => {			
				postErrors = "";
				if (!e.target[0].value.trim()) {
					postErrors = "You cannot send an empty post!";
					return false;
				};

				spinner.set(true);

				e.target[1].disabled = true;
				if ($user.name) {
					link.send({
						cmd: "direct",
						val: {
							cmd: "post_home",
							val: emoji.emojify(e.target[0].value),
						},
						listener: "post_home",
					});
					const postListener = link.on("statuscode", cmd => {
						if (cmd.listener !== "post_home") return;
						link.off(postListener);
						spinner.set(false);

						e.target[1].disabled = false;

						if (cmd.val === "I:100 | OK") {
							e.target[0].value = "";
							e.target[0].rows = "1";
							e.target[0].style.height = "45px";
						} else if (cmd.val === "E:106 | Too many requests") {
							postErrors = "You're posting too fast!";
						} else {
							postErrors = "Unexpected " + cmd.val + " error!";
						}
					});
					return false;
				} else {
					if (_ulist.includes("Webhooks")) {
						if (localStorage.getItem("webhooksAgree") === null) {
							$modalPage = "WH_agree"
							$modalShown = true
							e.target[1].disabled = false;
							e.target[0].value = "";
							e.target[0].rows = "1";
							e.target[0].style.height = "45px";
							spinner.set(false);
						} else {
							post("https://webhooks.meower.org/post/home",{post: e.target[0].value})
							e.target[1].disabled = false;
							e.target[0].value = "";
							e.target[0].rows = "1";
							e.target[0].style.height = "45px";
							spinner.set(false);
						}
					} else {
						Dialog("Webhooks is down","Sorry, the Webhooks servers are currently down, so you can't post as a guest right now.")
						e.target[1].disabled = false;
						e.target[0].value = "";
						e.target[0].rows = "1";
						e.target[0].style.height = "45px";
						spinner.set(false);
					}
				}
			}}
		>
			<textarea
				type="text"
				class="white"
				placeholder="Write something..."
				id="postinput"
				name="postinput"
				autocomplete="false"
				maxlength=""
				rows="1"
                autofocus
				use:autoresize
				on:input={() => {
					if ($lastTyped + 1500 < new Date() * 1) {
						lastTyped.set(new Date() * 1);
						link.send({
							cmd: "direct",
							val: {
								cmd: "set_chat_state",
								val: {
									chatid: "livechat",
									state: 101
								},
							},
							listener: "typing_indicator",
						});
					}
				}}
				on:keydown={(event) => {
					if (event.key == "Enter" && !shiftHeld) {
						event.preventDefault();
						document.getElementById("submitpost").click();
					}
				}}
				bind:this={postInput}
			></textarea>
			<button id="submitpost">Post</button>
		</form>
		<div class="post-errors">{postErrors}</div>
		<TypingIndicator />
		{#if posts.length < 1}
			{#if $user.name}
				No posts here. Check back later or be the first to post!
			{:else}
				No posts here. Check back later!
			{/if}
		{:else}
			{#each posts as post (post.id)}
				<div
					transition:fly|local="{{y: -50, duration: 250}}"
					animate:flip="{{duration: 250}}"
                    id={post.post_id}
				>
					<Post post={post} input={postInput} />
				</div>
			{/each}
		{/if}
		<div class="center">
			{#if pageLoading}
				<Loading />
			{:else}
				{#if numPages && numPages > pagesLoaded}
					<button 
						class="load-more"
						on:click={() => loadPage(pagesLoaded + 1)}
					>
						Load More
					</button>
				{/if}
			{/if}
		</div>
	{:catch error}
		<Container>
			<h1>Home</h1>
			Error loading posts. Please try again.
			<script>console.error("Posts were unable to load");</script>
			<pre><code>{error}</code></pre>
		</Container>
	{/await}
</div>

<style>
	.createpost {
		display: flex;
		margin-bottom: 0.5em;
		border-radius: 5px;
	}
	.createpost textarea {
		flex-grow: 1;
		margin-right: 0.25em;
		resize: none;
		max-height: 300px;
		border-radius: 5px;
	}
	.home {
		height: 100%;
	}
	.center {
		text-align: center;
	}
	.load-more {
		width: 100%;
		margin-bottom: 1.88em;
	}
	input[type="checkbox"], button.circle {
		border: none;
		margin: 0;
		margin-left: 0.125em;
	}
	.fullcenter {
		text-align: center;
		display: flex;
		justify-content: center;
		align-items: center;

		width: 100vw;
		height: 100vh;

		position: fixed;
		top: 0;
		left: 0;
	}
	.post-errors {
		color: red;
		font-size: 75%;
		font-weight: bold;
		margin: 0.25em 0;
	}
	.settings-controls {
		position: absolute;
		top: 0.25em;
		right: 0.25em;
	}
</style>
