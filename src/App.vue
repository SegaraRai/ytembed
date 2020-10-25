<script lang="ts">
/* eslint-disable vue/no-multiple-template-root */

import { computed, ref } from 'vue';

function extractVideoId(videoURL: string): string | undefined {
  const match = videoURL.match(
    /(?:^|\/\/)(?:(?:www\.)?(?:youtu\.be\/|youtube\.com\/embed\/|youtube\.com\/watch\?v=))?([\w-]{10,13})(?:[?&/]|$)/
  );
  if (!match) {
    return;
  }
  return match[1];
}

export default {
  name: 'App',
  components: {},
  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  setup() {
    const searchParams = new URLSearchParams(location.search);

    const navBarVisible = ref(true);
    const loop = ref(true);
    const hide = ref(false);
    const videoURL = ref(searchParams.get('v') || '');

    const embedURL = computed(() => {
      const videoId = extractVideoId(videoURL.value);
      if (!videoId) {
        return;
      }

      const escapedVideoId = encodeURIComponent(videoId);

      return `https://www.youtube.com/embed/${escapedVideoId}?loop=${
        loop.value ? '1' : '0'
      }&playlist=${escapedVideoId}&rel=0&autoplay=1`;
    });

    return {
      navBarVisible$$q: navBarVisible,
      loop$$q: loop,
      hide$$q: hide,
      videoURL$$q: videoURL,
      embedURL$$q: embedURL,
    };
  },
};
</script>

<template>
  <main class="fixed w-full h-full">
    <template v-if="embedURL$$q">
      <div
        class="w-full h-full"
        :style="{ display: hide$$q ? 'none' : 'block' }"
      >
        <iframe
          :src="embedURL$$q"
          class="w-full h-full"
          frameborder="0"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen
        ></iframe>
      </div>
    </template>
  </main>
  <header class="fixed w-full flex flex-no-wrap flex-no-wrap items-stretch">
    <div
      class="bg-gray-800 w-6"
      :class="[!navBarVisible$$q && 'h-12']"
      @click.stop="navBarVisible$$q = !navBarVisible$$q"
    ></div>
    <div
      v-show="navBarVisible$$q"
      class="bg-gray-900 flex-grow flex flex-wrap items-center justify-between px-6 py-2 w-full h-full"
    >
      <input
        v-model="videoURL$$q"
        class="bg-white text-black block w-full mb-2 sm:mb-0 sm:w-auto flex-grow border px-2 py-1 appearance-none leading-tight"
        type="url"
      />
      <label class="flex items-center flex-grow-0 ml-8 font-medium">
        <input v-model="hide$$q" class="leading-tight" type="checkbox" />
        Hide
      </label>
      <label class="flex items-center flex-grow-0 mx-8 font-medium">
        <input v-model="loop$$q" class="mr-2 leading-tight" type="checkbox" />
        Loop
      </label>
      <button
        class="block border px-4 py-1 appearance-none leading-tight"
        @click.stop="navBarVisible$$q = false"
      >
        Close
      </button>
    </div>
  </header>
</template>
