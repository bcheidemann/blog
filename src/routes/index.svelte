<script context="module">
  import bio from "../../content/bio.md"

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
  <section class="flex flex-col justify-center text-justify text-xl [&>h2]:text-center [&>ul]:text-left">
    <svelte:component this={bio} />
  </section>

  <hr />

  <!-- recent posts -->
  <section>
    <span class="flex items-baseline gap-4">
      <h1>
        <a href="#recent-posts" id="recent-posts">Recent Articles</a>
      </h1>
      <ButtonLink href="/posts" size="medium" raised={false} class="opacity-60">View All</ButtonLink>
    </span>
    <ul class="grid gap-4 grid-cols-1 sm:grid-cols-2 pl-0">
      {#each recentPosts as post}
        <li class="flex p-4 border border-slate-300 dark:border-slate-700 rounded-lg">
          <PostPreview {post} small />
        </li>
      {/each}
    </ul>
  </section>
</div>
