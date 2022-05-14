<script context="module">
  import bio from "../../content/bio.md"
  import content from "../../content/content.md"

  export const prerender = true

  export const load = async ({ fetch }) => {
    return {
      props: {
        recentPosts: await fetch('/posts.json?limit=2').then((res) => res.json())
      }
    }
  }
</script>

<script>
  import ButtonLink from '$lib/components/ButtonLink.svelte'
  import PostPreview from '$lib/components/PostPreview.svelte'
  import { name } from '$lib/info.js'

  export let recentPosts
</script>

<svelte:head>
  <title>{name}</title>
</svelte:head>

<div class="flex flex-col flex-grow">
  <!-- Biography -->
  <div class="flex flex-col items-center justify-center text-xl text-center">
    <svelte:component this={bio} />
  </div>

  <hr />

  <!-- recent posts -->
  <h1 class="flex items-baseline gap-4 mb-4">
    <a href="#recent-posts" id="recent-posts">Recent Posts</a>
    <ButtonLink href="/posts" size="medium" raised={false} class="opacity-60">View All</ButtonLink>
  </h1>
  <ul class="grid gap-4 grid-cols-1 sm:grid-cols-2 pl-0">
    {#each recentPosts as post}
      <li class="flex p-4 border border-slate-300 dark:border-slate-700 rounded-lg">
        <PostPreview {post} small />
      </li>
    {/each}
  </ul>

  <hr />

  <!-- Page Content -->
  <div class="flex flex-col items-center justify-center text-xl mb-8 mt-16 text-center">
    <svelte:component this={content} />
  </div>
</div>
