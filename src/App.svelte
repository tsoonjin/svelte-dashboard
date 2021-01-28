<script>
    import { DataTable, Button, DatePicker, DatePickerInput } from "carbon-components-svelte";
    import Footer from '@components/Footer.svelte'
    import { csvGenerator } from "@utils/csvGenerator.js"
    import Nav from '@components/Nav.svelte'
    import * as metrics from '@data/metrics.json'

    let name = process.env.name;
    let like = 0
    const head3 = [
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
            key: "error5XXRate",
            value: "Error 5XX Rate"
        },
        {
            key: "error4XXRate",
            value: "Error 4XX Rate"
        }
    ]
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
    const data = Object.entries(metrics.frontend.health.result).map(([k, v]) => ({id: k, ...v}))
    const data2 = Object.entries(metrics.backend.apigw.result).map(([k, v]) => {
        const { request } = v
        delete v['request'];
        return {id: k, ...v, ...request}
    })
    const data3 = Object.entries(metrics.backend.cf.result).map(([k, v]) => {
        const { request } = v
        delete v['request'];
        return {id: k, ...v, ...request}
    })
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

    function downloadHandler(data, head, fileName) {
        const tableKeys = Object.keys(data[0])
        const tableHeader = head.map(h => h.value)
        return () => { csvGenerator(data, tableKeys, tableHeader, `${fileName}.csv`)
        }}

</script>
<main>
	<h1>Frontend</h1>
    <h2>Cloudfront</h2>
    <DatePicker datePickerType="single">
        <DatePickerInput labelText="Select month" placeholder="mm/yyyy" />
    </DatePicker>
    <DataTable zebra sortable headers={head} rows={data} />
    <Button on:click={downloadHandler(data, head, "portalCF")}>Download</Button>
	<h1>Backend</h1>
    <h2>API Gateway</h2>
    <DataTable zebra sortable headers={head2} rows={data2} />
    <Button on:click={downloadHandler(data2, head2, "serviceAPIGateway")}>Download</Button>
    <h2>Cloudfront</h2>
    <DataTable zebra sortable headers={head3} rows={data3} />
    <Button on:click={downloadHandler(data3, head3, "serviceCF")}>Download</Button>
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
		color: #fffffe;
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
