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
    <div class="opacity-70">
      <time>{format(new Date(parseISO(post.date)), 'MMMM d, yyyy')}</time>
      •
      <span>{post.readingTime}</span>
    </div>
  </div>
  <div class="flex-1">{@html post.preview.html}</div>
  <slot name="actions">
    <div class="flex justify-end w-full">
      <ButtonLink href={`/posts/${post.slug}`}>Read More</ButtonLink>
    </div>
  </slot>
</article>
