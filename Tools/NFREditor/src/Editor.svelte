<script lang="ts">
import { init, onMount } from "svelte/internal";
/**
 * Next:
 * 1. Save editing path in query string
 * 2. Component to explore repo starting from /Best practices
 * 3. Beautify HTML
 * 4. Nicer formatting - wider Text boxes  
 * 
 */
	export let name: string;
	export let path: string;

	import APIHelper from "./api";
	import TextEdit from "./TextEdit.svelte";

	function listRepo() {
		APIHelper.doOperation();
	}

	function downloadData() {
		listRepo();
		//download("nfr.html", "<html><body><p>Test</p></body></html>");
	}

	function download(filename, text) {
		var element = document.createElement("a");
		element.setAttribute(
			"href",
			"data:text/plain;charset=utf-8," + encodeURIComponent(text)
		);
		element.setAttribute("download", filename);

		element.style.display = "none";
		document.body.appendChild(element);

		element.click();

		document.body.removeChild(element);
	}

	function initRepo(){
		APIHelper.token =  "f5373c1fe2b8daf102ec8280d63eccf019a30a77";
        APIHelper.user = "costin-hcl";
        APIHelper.repo = "hello-world";
	}
	
	onMount(() => {
       initRepo();
    	path = "Best practices/NFR1 - Debuggability and observability/Dynamics 365 Finance/ApplicationInsights from D365FSCM.html";
    });
</script>

<main>
	<h1>Best practices. Together</h1>
 
	 
	{#if APIHelper.token}
	<div><input bind:value="{path}" />
	</div>
	<TextEdit path={path} />
	{/if}
</main>
<svelte:head>
	<link
		rel="stylesheet"
		href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css"
	/>
	<script src="https://cdn.rawgit.com/beautify-web/js-beautify/v1.13.5/js/lib/beautify-html.js"></script>
</svelte:head>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	input{
		min-width: 700px;
		font-size: smaller;
	}
	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
