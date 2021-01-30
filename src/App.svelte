<script>
  import { Router, Route } from 'svelte-routing';
  import Home from '@components/Home.svelte';
  import NotFound from '@components/NotFound.svelte';
  import Monitoring from '@components/Monitoring.svelte';

  export let url = '';
</script>


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

<main>
    <Router url={url}>
        <Route path="monitoring" component={Monitoring} />
        <Route path="/" component={Home} />
        <Route component={NotFound} />
    </Router>
</main>
