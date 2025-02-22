<template>
  <div class="flex-vertical fill-screen" v-if="!storySelected">
    <error-screen
      v-if="fetchError"
      :error-message="fetchErrorMessage"
      :route-path="route.path"
    />
    <div class="loading-container" v-if="!ready">
      <neu-progress-bar :show-percentage="true" :progress="initProgress" />
    </div>
    <!-- eslint-disable vue/require-v-for-key -->
    <div
      class="story-title-container flex-vertical"
      v-for="(story, index) in storyIndex.abstracts"
    >
      <!-- eslint-enable vue/require-v-for-key -->
      <student-archive-title
        :avatar="studentAvatar"
        :title="story.title"
        :language="language"
        :index="index"
        @clicked="handleOpenIndex(index)"
        :is-active="activeIndex.includes(index)"
      />
      <router-link :to="`/archive/${studentId}/story/${story.groupId}`">
        <story-brief-block
          v-show="activeIndex.includes(index)"
          :title="getStoryTitle(userLanguage, story.title)"
          :description="story.abstract"
        />
      </router-link>
    </div>
  </div>
  <router-view />
</template>

<script setup lang="ts">
import axios, { AxiosError } from "axios";
import { computed, ref } from "vue";
import { useRoute } from "vue-router";
import StoryBriefBlock from "../story/StoryBriefBlock.vue";
import ErrorScreen from "../widgets/ErrorScreen.vue";
import NeuProgressBar from "../widgets/NeuUI/NeuProgressBar.vue";
import StudentArchiveTitle from "../widgets/StudentArchiveTitle.vue";
import { Language } from "@/types/Settings";
import { CommonStoryTextObject, StoryIndex } from "@/types/StoryJson";
import { useSettingsStore } from "@store/settings";
import { useStudentStore } from "@store/students";

const storyIndex = ref<StoryIndex>({} as StoryIndex);
const activeIndex = ref<number[]>([]);

const storySelected = computed(() => !/\/story\/?$/.test(route.path));

const settingsStore = useSettingsStore();
const language = computed(() => settingsStore.getLang);
const studentStore = useStudentStore();
const route = useRoute();

const ready = ref(false);
const initProgress = ref(0);
// get 请求是否出错
const fetchError = ref(false);
const fetchErrorMessage = ref<{
  message: string;
  response: { status: number };
}>({
  message: "",
  response: {
    status: 0,
  },
});

const studentId = computed(() => parseInt(route.params.id as string));
const studentAvatar = studentStore.getStudentAvatar(studentId.value);
const userLanguage = computed(() => settingsStore.getLang);

axios
  .get(`/story/favor/${studentId.value}/index.json`, {
    onDownloadProgress: progressEvent => {
      const total = progressEvent.total || progressEvent.loaded + 100;
      initProgress.value = Math.floor((progressEvent.loaded * 100) / total);
    },
  })
  .then(res => {
    if (typeof res.data !== "object") throw new AxiosError("404", "404");
    storyIndex.value = res.data;
  })
  .catch((err: AxiosError) => {
    return axios
      .get(`/story/ai/favor/${studentId.value}/index.json`, {
        onDownloadProgress: progressEvent => {
          const total = progressEvent.total || progressEvent.loaded + 100;
          initProgress.value = Math.floor((progressEvent.loaded * 100) / total);
        },
      })
      .then(res => {
        storyIndex.value = res.data;
      })
      .catch((err: AxiosError) => {
        console.error(err);
        fetchError.value = true;
        /* eslint-disable indent */
        fetchErrorMessage.value =
          404 === err.response?.status
            ? {
                message: "Story not found",
                response: {
                  status: 1919,
                },
              }
            : err;
        /* eslint-enable indent */
      });
  })
  .finally(() => {
    ready.value = true;
  });

function handleOpenIndex(index: number) {
  if (activeIndex.value.includes(index)) {
    activeIndex.value = activeIndex.value.filter(i => i !== index);
  } else {
    activeIndex.value.push(index);
  }
}

function getStoryTitle(userLanguage: Language, titles: CommonStoryTextObject) {
  return (
    Reflect.get(
      titles,
      "Text" + userLanguage[0].toUpperCase() + userLanguage.slice(1)
    ) || titles.TextJp
  );
}
</script>

<style scoped lang="scss">
.title {
  margin-bottom: 1rem;
}

.story-title-container {
  align-items: stretch;
  gap: 1rem;
  width: 30rem;
}

a {
  text-decoration: none;
}

@media (max-width: 768px) {
  .story-title-container {
    max-width: calc(100vw - 2rem);
  }
}
</style>
