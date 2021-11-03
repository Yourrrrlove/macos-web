<script lang="ts">
  import { wallpapersConfig } from '__/configs/wallpapers/wallpaper.config';
  import { smallerClosestValue } from '__/helpers/smaller-closest-value';
  import { createIntervalStore } from '__/stores/interval.store';
  import { theme } from '__/stores/theme.store';
  import { wallpaper } from '__/stores/wallpaper.store';

  let visibleBackgroundImage = '/assets/wallpapers/37-2.jpg';

  const interval = createIntervalStore(5 * 1000);

  $: {
    $interval;

    if (wallpapersConfig[$wallpaper.id].type === 'standalone') {
      $wallpaper.image = wallpapersConfig[$wallpaper.id].thumbnail;
      break $;
    }

    /** Only dynamic and light/dark wallpaper logic to tackle */
    // Now check if user really wants the change to happen.

    handleTheme();
    handleWallpaper();
  }

  function handleWallpaper() {
    const date = new Date();
    const hour = date.getHours();

    const wallpaperTimestampsMap = wallpapersConfig[$wallpaper.id].timestamps.wallpaper;
    const timestamps = Object.keys(wallpaperTimestampsMap);

    const minTimestamp = Math.min(...timestamps);
    const maxTimestamp = Math.max(...timestamps);

    if (hour > maxTimestamp || hour < minTimestamp) {
      // Go for the min timestamp value
      if (wallpaperTimestampsMap[maxTimestamp]) {
        $wallpaper.image = wallpaperTimestampsMap[maxTimestamp];
      }

      return;
    }

    // Now set the right timestamp
    const chosenTimeStamp = smallerClosestValue(timestamps, hour);

    if (wallpaperTimestampsMap[chosenTimeStamp]) {
      $wallpaper.image = wallpaperTimestampsMap[chosenTimeStamp];
    }
  }

  function handleTheme() {
    if (!$wallpaper.canControlTheme) return;

    const date = new Date();
    const hour = date.getHours();

    const themeTimestampsMap = wallpapersConfig[$wallpaper.id].timestamps.theme;
    const timestamps = Object.keys(themeTimestampsMap);

    const minTimestamp = Math.min(...timestamps);
    const maxTimestamp = Math.max(...timestamps);

    if (hour > maxTimestamp || hour < minTimestamp) {
      // Go for the min timestamp value
      $theme = 'dark';
      return;
    }

    // Now set the right timestamp
    const chosenTimeStamp = smallerClosestValue(timestamps, hour);
    $theme = themeTimestampsMap?.[chosenTimeStamp] || 'light';
  }

  function previewImageOnLoad() {
    visibleBackgroundImage = `/assets/wallpapers/${$wallpaper.image}.jpg`;
  }
</script>

<!-- Prefetch all wallpapers -->
<svelte:head>
  {#each Object.values(wallpapersConfig) as { thumbnail }}
    <link rel="prefetch" href="/assets/wallpapers/{thumbnail}.jpg" />
  {/each}
</svelte:head>

<!-- This preload and render the image for browser but invisible to user -->
<img
  src="/assets/wallpapers/{$wallpaper.image}.jpg"
  aria-hidden="true"
  alt=""
  on:load={previewImageOnLoad}
/>

<div class="background-cover" style="background-image: url({visibleBackgroundImage});" />

<style lang="scss">
  img {
    height: 1px;
    width: 1px;
  }

  .background-cover {
    height: 100%;
    width: 100%;

    z-index: -1;
    position: fixed;
    top: 0;
    left: 0;

    will-change: background-image;

    transition: background-image 150ms ease-in;

    background-repeat: no-repeat;
    background-size: cover;
    background-position: center;

    background-image: url(/assets/wallpapers/37-2.jpg);
  }
</style>