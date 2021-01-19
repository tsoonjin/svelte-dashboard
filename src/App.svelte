<script>
    import { DataTable } from "carbon-components-svelte";
    import Footer from '@components/Footer.svelte'
    import Nav from '@components/Nav.svelte'
    import * as metrics from '@data/metrics.json'

    let name = process.env.name;
    let like = 0
    const head = [
        {
            key: "minRequests",
            value: "Min Requests"
        },
        {
            key: "meanRequests",
            value: "Avg Requests"
        },
        {
            key: "maxRequests",
            value: "Max Requests"
        },
        {
            key: "totalRequests",
            value: "Total Requests"
        }
    ]
    const data = Object.entries(metrics.frontend.health.result).map(([k, v]) => ({...v, id: k}))
     const settings = {
        sortable: true,
        pagination: true,
        rowPerPage: 50,
        columnFilter: true,
    }
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
    {console.log(data)}
    <DataTable zebra sortable headers={head} rows={data} />
    <Nav menu={{Health: "/health", Security: "/security"}} />
    <button on:click={handleLike}>
        {like} {like === 1 ? "like" : "likes"}
    </button>
    <input bind:value={name}/>
    <Footer salutation={"Mr. "}/>
</main>

<style lang="scss" global>
    /** Gray 10 theme **/
    @import "carbon-components-svelte/css/g90";
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
