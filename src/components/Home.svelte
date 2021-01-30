<script>
    import { onMount } from "svelte";
    import { DataTable, Button, DatePicker, DatePickerInput, Loading } from "carbon-components-svelte";
    import Footer from '@components/Footer.svelte'
    import { csvGenerator } from "@utils/csvGenerator.js"
    import Nav from '@components/Nav.svelte'

    const metricsEndpoint = "https://d5eng-api.eco.astro.com.my/metrics"
    let name = process.env.name;
    let like = 0
    let selectedDate = new Date((new Date()).getTime() - (30 * 24 * 3600000))
    /* let queryDate = `${today.getYear()}/${today.getMonth()}` */
    let queryDate = `2021/01`
    let data = fetch(`${metricsEndpoint}/aws/${queryDate}.json`).then(res => res.json())

    onMount(() => {
        console.log("Fetching data")
        data = fetch(`${metricsEndpoint}/aws/${queryDate}.json`).then(res => res.json())})

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
<div>
    <h1>D5 Engineering Dashboard</h1>
    <div class="config">
        <DatePicker class="date" datePickerType="single">
        <DatePickerInput labelText="Start Date" placeholder="mm/dd/yyyy" />
        </DatePicker>
        <DatePicker class="date" datePickerType="single">
        <DatePickerInput labelText="End Date" placeholder="mm/dd/yyyy" />
        </DatePicker>
    </div>
    <h2>Portal Cloudfront</h2>
    {#await data}
        <Loading withOverlay={false} small />
    {:then res}
        <div class="metrics">
        <DataTable zebra sortable headers={head} rows={Object.entries(res.frontend.health.result).map(([k, v]) => ({id: k, ...v}))}/>
        <Button class="btn" on:click={downloadHandler(Object.entries(res.frontend.health.result).map(([k, v]) => ({id: k, ...v})), head, "portalCF")}>Download</Button>
        </div>
    {:catch error}
        <p class="empty-data">No data</p>
    {/await}
    <h2>Service API Gateway</h2>
    {#await data}
        <Loading withOverlay={false} small />
    {:then res}
        <div class="metrics">"
        <DataTable zebra sortable headers={head2} rows={Object.entries(res.backend.apigw.result).map(([k, v]) => {
            const { request } = v
            delete v['request'];
            return {id: k, ...v, ...request}
                   })} />
        <Button class="btn" on:click={downloadHandler(Object.entries(res.backend.apigw.result).map(([k, v]) => {
            const { request } = v
            delete v['request'];
            return {id: k, ...v, ...request}
                }), head2, "serviceAPIGateway")}>Download</Button>
        </div>
    {:catch error}
        <p class="empty-data">No data</p>
    {/await}
    <h2>Service Cloudfront</h2>
    {#await data}
        <Loading withOverlay={false} small />
    {:then res}
        <div class="metrics">
        <DataTable zebra sortable headers={head3} rows={Object.entries(res.backend.cf.result).map(([k, v]) => {
            const { request } = v
            delete v['request'];
            return {id: k, ...v, ...request}
        })
        } />
        <Button class="btn" on:click={downloadHandler(Object.entries(res.backend.cf.result).map(([k, v]) => {
            const { request } = v
            delete v['request'];
            return {id: k, ...v, ...request}
            })
, head3, "serviceCF")}>Download</Button>
        </div>
    {:catch error}
        <p class="empty-data">No data</p>
    {/await}
</div>

<style lang="scss" global>
    /** Gray 10 theme **/
    @import "carbon-components-svelte/css/g10";
    .metrics {
        display: flow-root;
        margin-bottom: 16px;
    }

    .btn {
        margin-top: 16px;
        float: left
    }

	h2 {
        text-align: left;
		text-transform: uppercase;
		font-size: 2em;
		font-weight: 100;
	}

    .config {
        margin: 16px 0;
        display: flow-root
    }

    .date {
        float: left;
        margin-right: 24px
    }

    .empty-data {
        min-height: 200px;
    }
</style>
