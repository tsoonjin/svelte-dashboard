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
    let pagerduty = fetch(`${metricsEndpoint}/pagerduty/${queryDate}.json`).then(res => res.json())
    let runscope = fetch(`${metricsEndpoint}/runscope/${queryDate}.json`).then(res => res.json())

    const extractPagerduty = (res) => {
        return res.map(service => {
            const { service_id, total_incident_count, mean_seconds_to_resolve, team_name, service_name } = service
            return {
                id: service_id,
                service_name,
                team_name,
                total_incident_count,
                mean_seconds_to_resolve
            }
        })
    }

    const extractRunscope = (res) => {
        return Object.entries(res).map(([key, value]) => {
            const { avg_success_ratio, result } = value
            return {
                id: key,
                avg_success_ratio,
                response_time_50th_percentile: result.response_time_50th_percentile
            }
        })
    }

    const head2 = [
        {
            key: "id",
            value: "Id"
        },
        {
            key: "avg_success_ratio",
            value: "Success Ratio"
        },
        {
            key: "response_time_50th_percentile",
            value: "Avg Response Time (ms)"
        }
    ]
    const head = [
        {
            key: "id",
            value: "Service Id"
        },
        {
            key: "service_name",
            value: "Service"
        },
        {
            key: "team_name",
            value: "Team"
        },
        {
            key: "total_incident_count",
            value: "Total Incidents"
        },
        {
            key: "mean_seconds_to_resolve",
            value: "MTTR (Mean Time To Recover)"
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
    <h2>Pagerduty</h2>
    {#await pagerduty}
        <Loading withOverlay={false} small />
    {:then res}
        <div class="metrics">
            <DataTable zebra sortable headers={head} rows={extractPagerduty(res)}/>
        <Button class="btn" on:click={downloadHandler(extractPagerduty(res), head, "portalCF")}>Download</Button>
        </div>
    {:catch error}
        <p class="empty-data">No data</p>
    {/await}
    <h2>Runscope</h2>
    {#await runscope}
        <Loading withOverlay={false} small />
    {:then res}
        <div class="metrics">"
        <DataTable zebra sortable headers={head2} rows={extractRunscope(res)} />
        <Button class="btn" on:click={downloadHandler(extractRunscope(res), head2, "serviceAPIGateway")}>Download</Button>
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
