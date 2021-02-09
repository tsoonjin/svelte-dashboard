<script>
    import { onMount } from "svelte";
    import {
        DataTable,
        Button,
        ComboBox,
        DatePicker,
        DatePickerInput,
        FormGroup,
        RadioButtonGroup,
        RadioButton,
        Loading
    } from "carbon-components-svelte";
    import Footer from '@components/Footer.svelte'
    import { csvGenerator } from "@utils/csvGenerator.js"
    import Nav from '@components/Nav.svelte'

    const metricsEndpoint = "https://d5eng-api.eco.astro.com.my/metrics"
    const getDailyReportPrefix = (selectedDate) => {
        const [month, day, year] = selectedDate.split('/')
        return `${year}/${month}/daily/${day}/${year}-${month}-${day}`
    }
    const today = new Date()
    let queryPeriod = 'monthly'
    let year = today.getFullYear()
    let month = today.getMonth()
    const monthMapping = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
    let name = process.env.name;
    let selectedDate = ""
    /* let queryDate = `${today.getYear()}/${today.getMonth()}` */
    $: queryDate = queryPeriod === 'monthly' ? `${year}/${('0' + (month + 1)).slice(-2)}` : getDailyReportPrefix(selectedDate)
    $: data = fetch(`${metricsEndpoint}/aws/${queryDate}.json`).then(res => res.json())
    $: lighthouseData = fetch(`${metricsEndpoint}/lighthouse/prd/${queryDate}/${('0' + (month + 1)).slice(-2)}.json`).then(res => res.json())
    $: bugsnagData = fetch(`${metricsEndpoint}/bugsnag/${queryDate}/${('0' + (month + 1)).slice(-2)}.json`).then(res => res.json())

    const extractBugsnag = (res) => {
        return Object.entries(res).map(([key, value]) => {
            return {
                id: key,
                name: value.name,
                url: value.url,
                openErrors: value.openErrors,
                topErrors: value.openErrors > 0 ? value.topErrors.map(error => `${error.context}`).join(" , ") : ""
            }
        })
    }

    const extractLighthouse = (res) => {
        return res.map(site => {
            const { url, score, detail } = site
            return {
                id: url,
                score,
                performance: detail.performance,
                accessibility: detail.accessibility,
                seo: detail.seo
            }
        })
    }

    const bugsnagHeaders = [
        {
            key: "id",
            value: "Id"
        },
        {
            key: "name",
            value: "Name"
        },
        {
            key: "url",
            value: "Url"
        },
        {
            key: "openErrors",
            value: "Open Errors"
        }
    ]

    const lighthouseHeaders = [
        {
            key: "id",
            value: "Id"
        },
        {
            key: "score",
            value: "Overall"
        },
        {
            key: "performance",
            value: "Performance"
        },
        {
            key: "accessibility",
            value: "Accessibility"
        },
        {
            key: "seo",
            value: "SEO"
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


    function downloadHandler(data, head, fileName) {
        const tableKeys = Object.keys(data[0])
        const tableHeader = head.map(h => h.value)
        return () => { csvGenerator(data, tableKeys, tableHeader, `${fileName}.csv`)
        }}

</script>
<div>
    <h1>D5 Engineering Dashboard</h1>
    <FormGroup>
        <RadioButtonGroup name="radio-button-group" bind:selected={queryPeriod} on:change={() => console.log(queryPeriod)}>
            <RadioButton
                id="queryPeriod-1"
                value="monthly"
                labelText="Monthly"
            />
            <RadioButton
                id="queryPeriod-2"
                value="daily"
                labelText="Daily"
            />
        </RadioButtonGroup>
    </FormGroup>
    <div class="config">
        {#if queryPeriod === 'daily'}
        <DatePicker class="date" datePickerType="single" bind:value={selectedDate}>
        <DatePickerInput placeholder="mm/dd/yyyy"/>
        </DatePicker>
        {/if}
        {#if queryPeriod === 'monthly'}
            <ComboBox
                width="300px"
                placeholder="Select month"
                items={monthMapping.map((value, index) => ({ id: index, text: value}))}
                bind:selectedIndex={month}
                size="sm"
                />
        {/if}
    </div>
    <h2>Cloudfront</h2>
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
    <h2>Lighthouse</h2>
    {#await lighthouseData}
        <Loading withOverlay={false} small />
    {:then res}
        <div class="metrics">
        <DataTable zebra sortable headers={lighthouseHeaders} rows={extractLighthouse(res)}/>
        <Button class="btn" on:click={downloadHandler(extractLighthouse(res), lighthouseHeaders, `lighthouse-prd-${queryDate}.csv`)}>Download</Button>
        </div>
    {:catch error}
        <p class="empty-data">No data</p>
    {/await}
    <h2>Bugsnag</h2>
    {#await bugsnagData}
        <Loading withOverlay={false} small />
    {:then res}
        <div class="metrics">
        <DataTable zebra sortable headers={bugsnagHeaders} rows={extractBugsnag(res)}/>
        <Button class="btn" on:click={downloadHandler(extractBugsnag(res), bugsnagHeaders, `bugsnag-${queryDate}.csv`)}>Download</Button>
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
