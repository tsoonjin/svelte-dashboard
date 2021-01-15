<script>
    import Footer from '@components/Footer.svelte'
    let name = process.env.name;
    let like = 0
    $: likeNeeded = 100 - like;


    function handleLike() {
        like += 1
    }
</script>
<main>
	<h1>Hello {name}!</h1>
	<p>Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.</p>
    <h2>You need {likeNeeded}</h2>
    {#if likeNeeded % 2 === 0}
        <h3>Even likes</h3>
    {/if}
    <button on:click={handleLike}>
        {like} {like === 1 ? "like" : "likes"}
    </button>
    <div>
        <input bind:value={name}/>
<Footer />
</main>

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

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
