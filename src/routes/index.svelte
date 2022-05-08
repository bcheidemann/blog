<script context="module">
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
  <!-- replace with a bio about you, or something -->
  <div class="flex flex-col items-center justify-center text-xl mb-8 text-center">
    <h1>About Me</h1>
    <p>
      Hi! I'm Ben. I'm a full stack web developer with many passions and interests. I'm not sure exactly what I will post here yet, nor how frequently I will upload new posts. However, I figured that a good place to start would be to post about the projects I am working on in my free time, in the hopes that whoever stumbles across this page will find reading about these projects as interesting as I found them to work on. Thanks for stopping by! 😊
    </p>
  </div>

  <!-- recent posts -->
  <h1 class="flex items-baseline gap-4 mb-4">
    Recent Posts
    <ButtonLink href="/posts" size="medium" raised={false} class="opacity-60">View All</ButtonLink>
  </h1>
  <ul class="grid gap-4 grid-cols-1 sm:grid-cols-2 pl-0">
    {#each recentPosts as post}
      <li class="flex p-4 border border-slate-300 dark:border-slate-700 rounded-lg">
        <PostPreview {post} small />
      </li>
    {/each}
  </ul>
</div>
