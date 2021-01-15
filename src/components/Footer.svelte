<script>
    import { personal } from "@config/vars.js"
    import { onMount } from 'svelte';

	let photos = [];
    export let salutation
    onMount(async () => {
		const res = await fetch(`https://jsonplaceholder.typicode.com/photos?_limit=20`);
		photos = await res.json();
	});
</script>

<footer>
    <p>Author: {salutation} {personal.name || 'Jin'}</p>
    <p><a href="mailto:{personal.email}">{personal.email}</a></p>
    <div class="photos">
	{#each photos as photo,i}
		<figure>
			<img src={photo.thumbnailUrl} alt={photo.title}>
            <figcaption>{i}: {photo.title}</figcaption>
		</figure>
	{:else}
		<!-- this block renders when photos.length === 0 -->
		<p>loading...</p>
	{/each}
</div>
</footer>

<style>
    p {
        font-size: 30px;
		color: #ff3e00;
    }
</style>
