<script>
    import { DataTable } from "carbon-components-svelte";
    import Footer from '@components/Footer.svelte'
    import Nav from '@components/Nav.svelte'
    import * as metrics from '@data/metrics.json'

    let name = process.env.name;
    let like = 0
    const head2 = [
        {
            key: "id",
            value: "Id"
        },
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
        },
        {
            key: "avgLatency",
            value: "Avg Latency"
        },
        {
            key: "error5XXRate",
            value: "Error 5XX Rate"
        },
        {
            key: "totalErrorRate",
            value: "Total Error Rate"
        }
    ]
    const head = [
        {
            key: "id",
            value: "Url"
        },
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
    const data2 = Object.entries(metrics.backend.apigw.result).map(([k, v]) => ({...v, id: k}))
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
	<h1>Frontend</h1>
    {console.log(data)}
    <DataTable zebra sortable headers={head} rows={data} />
	<h1>Backend</h1>
    <DataTable zebra sortable headers={head2} rows={data2} />
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
