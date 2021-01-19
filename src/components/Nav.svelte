<script>
    export let menu = {}
</script>
<nav>
  <ul>
    {#each Object.entries(menu) as item}
        <li><a href="{item[1]}">{item[0]}</a></li>
    {/each}
  </ul>
</nav>

<style>
    ul li {
        display: inline;
        padding-right: 10px;
    }

    ul {
        display: block;
        float: left;
    }
</style>
