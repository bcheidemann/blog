<script>
  import { format, parseISO } from 'date-fns'
  import ButtonLink from './ButtonLink.svelte'

  export let post
  export let small = false
</script>

<article class="flex flex-col" aria-labelledby="article-title">
  <div>
    {#if !small}
      <h3 class="!mt-0 !mb-2" id="article-title">
        <a href={`/posts/${post.slug}`}>{post.title}</a>
      </h3>
    {:else}
      <h3 class="!mt-0 !mb-2" id="article-title">
        <a href={`/posts/${post.slug}`}>{post.title}</a>
      </h3>
    {/if}
    <div class="flex gap-x-4 gap-y-4 flex-wrap justify-between opacity-70">
      <span class="text-xl truncate">{post.readingTime}</span>
      <div class="flex wrap">
        {#each post.categories as category}
          <span class="mr-2 px-4 py-1 shadow-md shadow-slate-400 dark:shadow-black rounded-full bg-slate-500 text-white text-sm truncate">
            {category}
          </span>
        {/each}
      </div>
    </div>
  </div>
  <div class="flex-1">{@html post.preview.html}</div>
  <slot name="actions">
    <div class="flex justify-end w-full">
      <ButtonLink href={`/posts/${post.slug}`}>Read More</ButtonLink>
    </div>
  </slot>
</article>
