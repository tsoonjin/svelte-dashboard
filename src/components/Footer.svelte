<script>
    import { personal } from "@config/vars.js"
    export let salutation
</script>

<footer>
    <p>Author: {salutation} {personal.name || 'Jin'}</p>
    <p><a href="mailto:{personal.email}">{personal.email}</a></p>
</footer>
