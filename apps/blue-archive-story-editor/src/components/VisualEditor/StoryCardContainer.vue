<script setup lang="ts">
import type { StoryNode } from '~/types/visual-editor.ts'
import { StoryNodeType } from '~/types/visual-editor'

const emits = defineEmits(['remove'])
const storyNode = defineModel<StoryNode>({ required: true })

function handleSelect(value: StoryNodeType) {
  storyNode.value.type = value
  if (storyNode.value.type === StoryNodeType.DialogNode) {
    storyNode.value.dialog = storyNode.value.dialog || {
      text: '',
      speaker: '',
    }
  }
  else if (storyNode.value.type === StoryNodeType.CharacterNode) {
    storyNode.value.characters = storyNode.value.characters || [null, null, null, null, null]
  }
}

const StoryNodeDescription = {
  [StoryNodeType.DialogNode]: '对话节点',
  [StoryNodeType.CharacterNode]: '角色节点',
  [StoryNodeType.BackgroundNode]: '背景节点',
  [StoryNodeType.TitleNode]: '标题节点',
  [StoryNodeType.BgmNode]: 'BGM 节点',
  [StoryNodeType.WaitNode]: '等待节点',
  [StoryNodeType.NaNode]: '文本节点',
}
</script>

<template>
  <div
    class="story-card-container" card w-auto select-none
  >
    <div class="header" flex="~ items-center" gap-2>
      <span class="drag-handle" i-material-symbols:drag-indicator inline-block cursor-pointer p-1 px4 text-size-xl />
      <div class="story-id" inline-block w-8 text="center">
        {{ storyNode.id }}
      </div>
      <TheSelect :model-value="storyNode.type" flex-1 @update:model-value="handleSelect($event as StoryNodeType)">
        <TheSelectOption v-for="storyNodeType in StoryNodeType" :key="storyNodeType" :value="storyNodeType">
          {{ StoryNodeDescription[storyNodeType] }}
        </TheSelectOption>
      </TheSelect>
      <button i-material-symbols:cancel-outline px3 icon-btn @click="emits('remove')" />
    </div>
    <DialogCard
      v-if="storyNode.type === StoryNodeType.DialogNode"
      v-model="storyNode"
    />
    <CharacterCard
      v-else-if="storyNode.type === StoryNodeType.CharacterNode"
      v-model="storyNode"
    />
    <BackgroundCard
      v-else-if="storyNode.type === StoryNodeType.BackgroundNode"
      v-model="storyNode"
    />
    <BgmCard
      v-else-if="storyNode.type === StoryNodeType.BgmNode"
      v-model="storyNode"
    />
    <WaitCard
      v-else-if="storyNode.type === StoryNodeType.WaitNode"
      v-model="storyNode"
    />
    <NaCard
      v-else-if="storyNode.type === StoryNodeType.NaNode"
      v-model="storyNode"
    />
    <TitleCard
      v-else-if="storyNode.type === StoryNodeType.TitleNode"
      v-model="storyNode"
    />
  </div>
</template>

<style scoped>
</style>
