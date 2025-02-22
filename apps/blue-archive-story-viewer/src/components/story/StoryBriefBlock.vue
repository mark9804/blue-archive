<script setup lang="ts">
import { computed, onMounted } from "vue";
import { CommonStoryTextObject } from "@/types/StoryJson";
import { useSettingsStore } from "@store/settings";
import NeuTag from "@widgets/NeuUI/NeuTag.vue";

const settingsStore = useSettingsStore();

const props = withDefaults(
  defineProps<{
    title: string;
    avatar?: string;
    description: CommonStoryTextObject;
    storyOrder?: number;
    isBeforeBattle?: boolean;
    isAfterBattle?: boolean;
  }>(),
  {
    avatar: "",
  }
);

const currentLanguage = computed(() => settingsStore.getLang);
const username = computed(() => settingsStore.getUsername);

const languageMap = {
  cn: "TextCn",
  en: "TextEn",
  jp: "TextJp",
  tw: "TextTw",
  th: "TextTh",
  kr: "TextKr",
};

function replaceDefaultNamePlaceholder(content: string | undefined): string {
  if (!content) {
    return "";
  }
  return content.replaceAll(/\[USERNAME]/gi, username.value);
}

const selectedLangDescription = computed(() => {
  const rawTitle = Reflect.get(
    props.description,
    Reflect.get(languageMap, currentLanguage.value)
  );
  return replaceDefaultNamePlaceholder(rawTitle);
});

// 用于指定语言无对应文本时返回 fallback 文本
function getFallbackDescription() {
  for (const lang in languageMap) {
    const text = Reflect.get(props.description, Reflect.get(languageMap, lang));
    if (text) {
      return replaceDefaultNamePlaceholder(text);
    }
  }
  return "NoFallbackText";
}

const storyOrder = computed(() => {
  if (undefined !== props.storyOrder) {
    return `${(props.storyOrder + 1).toString().padStart(2, "0")} `;
  }
  return "";
});

onMounted(() => import("@/components/StoryViewer.vue"));
</script>

<template>
  <div class="story-brief-block rounded-medium shadow-near">
    <img
      v-if="avatar"
      :src="avatar"
      class="avatar rounded-small"
      alt="avatar"
    />
    <div class="text-wrapper">
      <div class="title">
        <span>{{ storyOrder }}{{ title }}</span>
        <neu-tag v-if="isBeforeBattle" size="small">战斗前</neu-tag>
        <neu-tag v-if="isAfterBattle" size="small">战斗后</neu-tag>
      </div>
      <div class="description">
        {{ selectedLangDescription || `${getFallbackDescription()}` }}
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.story-brief-block {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  transition: all 0.375s ease-in-out;
  cursor: pointer;
  margin-bottom: 1rem;
  background: var(--color-title-container);
  padding: 1rem;
  width: 100%;
  color: var(--color-text-main);
  font-weight: bold;
  font-size: 1.25rem;
  user-select: none;

  img {
    flex: none;
    margin-right: 1rem;
    width: 8rem;
    object-fit: cover;
  }

  .text-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: flex-start;
    width: 100%;
    height: 100%;

    .title {
      transition: border-bottom 0.375s ease-in-out;
      border-bottom: solid 1px var(--color-text-decoration);
      width: 100%;
      font-weight: bold;
      font-size: 1.25rem;
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }

    .description {
      margin-top: 0.5rem;
      color: gray;
      font-weight: normal;
      font-size: 1rem;
    }
  }
}

@media screen and (max-width: 768px) {
  .story-brief-block {
    flex-direction: column;
    align-items: flex-start;

    img {
      display: none;
    }
  }
}
</style>
