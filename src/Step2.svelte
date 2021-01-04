<script lang="ts">
  import { onDestroy } from 'svelte'
  import { tweened } from 'svelte/motion'
  import { cubicOut } from 'svelte/easing'

  const animation = tweened(0, {
    duration: 4000,
    easing: cubicOut
  })
  const interval = setInterval(() => {
    animation.set($animation===1 ? 0 : 1)
  }, 5000)
  onDestroy(() => clearInterval(interval))
</script>

<main>
  <div>$animation: {$animation}</div>
</main>

<style>
</style>
