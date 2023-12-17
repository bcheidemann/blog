<script>
  import '../app.css'
  import '../prism.css'
  import 'focus-visible'
  import MoonIcon from 'heroicons-svelte/solid/MoonIcon.svelte'
  import SunIcon from 'heroicons-svelte/solid/SunIcon.svelte'
  import { browser, dev } from '$app/env'
  import { name } from '$lib/info'

  let prefersLight = browser ? Boolean(JSON.parse(localStorage.getItem('prefersLight'))) : false
</script>

<div class="flex flex-col min-h-screen">
  <div class="mx-auto flex flex-col flex-grow w-full max-w-4xl">
    <header class="flex h-16 px-4 py-2 justify-between items-center">
      <h2
        class="!text-transparent bg-clip-text bg-gradient-to-r from-blue-500 to-teal-500 dark:from-violet-500 dark:to-pink-500"
      >
        <a class="text-lg sm:text-2xl font-bold" href="/">
          {name}
        </a>
      </h2>
      {#if browser}
        <button
          type="button"
          role="switch"
          aria-label="Toggle Dark Mode"
          aria-checked={!prefersLight}
          class="h-4 w-4 sm:h-8 sm:w-8 sm:p-1"
          on:click={() => {
            prefersLight = !prefersLight
            localStorage.setItem('prefersLight', prefersLight.toString())

            if (prefersLight) {
              document.querySelector('html').classList.remove('dark')
            } else {
              document.querySelector('html').classList.add('dark')
            }
          }}
        >
          {#if prefersLight}
            <MoonIcon class="text-slate-500" />
          {:else}
            <SunIcon class="text-slate-400" />
          {/if}
        </button>
      {/if}
    </header>
    <main
      class="prose prose-slate prose-sm sm:prose sm:prose-slate sm:prose-lg sm:max-w-none dark:prose-invert flex flex-col w-full flex-grow py-4 px-4 mx-auto"
    >
      <slot />
    </main>
    <hr />
    <footer class="prose prose-slate prose-sm sm:prose sm:prose-slate sm:prose-ms sm:max-w-none dark:prose-invert flex flex-col w-full flex-grow px-4 pb-4 mx-auto opacity-70">
      <div class="flex justify-between gap-x-8 flex-col-reverse md:flex-row">
        <div>
          <h2 class="mb-2">Privacy Policy</h2>
          <p class="text-sm text-justify md:text-xs">
            Ben Heidemann Limited does not track, store or utilise any information relating to
            your visit to this website. The website may contain links to other websites run by other
            organisations. This Privacy Notice applies only to our website, so we encourage you to
            read the privacy statements on the other websites you visit. We cannot be responsible
            for the privacy policies and practices of other sites even if you access them using
            links from our website. In addition, if you linked to our website from a third-party
            site, we cannot be responsible for the privacy policies and practices of the owners and
            operators of that third-party site and recommend that you check the Privacy Notice of
            that third-party site.
          </p>
        </div>
        <div class="min-w-max">
          <h2>Contact Details</h2>
          <address>
            0/2 23 Bolton Drive
            <br />
            Glasgow
            <br />
            G42 9DX
            <br />
          </address>
          <a href="tel:+447472564288">tel: +44 7472 564288</a>
          <br />
          <a href="mailto:ben@heidemann.dev">email: ben@heidemann.dev</a>
        </div>
      </div>
      <div class="text-right opacity-70 text-xs sm:text-sm mt-6 sm:mt-10">
        © Copyright Ben Heidemann Limited All Rights Reserved
      </div>
    </footer>
  </div>
</div>
