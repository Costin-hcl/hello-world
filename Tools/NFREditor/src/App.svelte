<script lang="ts">
import Editor from "./Editor.svelte";

	export let name: string;
</script>

<main>
<Editor {name}/>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

 
	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>