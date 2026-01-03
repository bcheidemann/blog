<script context="module">
  import policy from "../../../content/myweightlogbook-privacy-policy.md"

  export const prerender = true
</script>

<script>
  import { name } from '$lib/info.js'
</script>

<svelte:head>
  <title>My Weight Logbook - Privacy Policy</title>
</svelte:head>

<div class="flex flex-col flex-grow">
  <section class="flex flex-col text-justify text-xl [&>h1]:mb-4 [&>h2]:mt-4 [&>h2]:mb-4 [&>h3]:mt-4 [&>h3]:mb-4 [&>p:first-of-type]:mt-0 [&>p:first-of-type]:font-semibold [&>p]:mb-4 [&>p]:mt-0 [&>ul]:pb-4">
    <svelte:component this={policy} />
  </section>
</div>

