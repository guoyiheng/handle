<script setup lang="ts">
import { showCustomize, isIdiomLegal } from '~/state'
import { t } from '~/i18n'
import { WORD_LENGTH } from '~/logic'


const inputIdiom = ref('')
const word = ref('')
const showAnswer = ref(false)
const inputRef = ref<HTMLInputElement>()
const shareUrl = ref('')
const illegal = ref(false)

function focus() {
  inputRef.value?.focus()
}
watchEffect(() => {
  if (showCustomize.value) {
    focus()
  }
})

function handleInput() {
  showAnswer.value = false
  word.value = Array.from(inputIdiom.value).filter(i => /\p{Script=Han}/u.test(i))
    .slice(0, 4)
    .join('')
}

function handleSubmit() {
  if (isIdiomLegal(word.value)) {
    shareUrl.value = window.location.origin + `/?idiom=${window.btoa(encodeURIComponent(word.value)!)}`
    showAnswer.value = true
  } else {
    illegal.value = true
  }
}

function handleOpenCustomizeUrl() {
  window.location.assign(shareUrl.value)
}


const share = useShare(computed(() => ({
  title: t('name'),
  text: shareUrl.value,
})))
const clipboard = useClipboard()

const shareDialog = ref(false)

function handleShareCustomizeUrl() {
  if (share.isSupported) {
    share.share()
    shareDialog.value = true
    clipboard.copy(shareUrl.value)
  }
}

</script>

<template>
  <div p="x5 y10">
    <div text-xl font-serif mb5>{{ t("fake-title") }}</div>
    <div flex="~ col gap-2" items-center>
      <WordBlocks v-if="!showAnswer" :word="word" @click="focus()" />
      <WordBlocks v-else :word="word" @click="focus()" :revealed="true" :answer="word" />
      <input
        ref="inputRef"
        v-model="inputIdiom"
        type="text"
        autocomplete="false"
        outline-none
        :placeholder="t('input-placeholder')"
        w-86
        p3
        border="2 base"
        text="center"
        bg="transparent"
        @input="handleInput"
        @keydown.enter="handleSubmit"
      />
      <button
        class="btn"
        :disabled="word.length !== WORD_LENGTH"
        @click="handleSubmit"
      >{{ t('ok') }}</button>
      <div w-86 flex justify-between v-if="shareUrl !== ''">
        <button
          overflow-hidden
          text-ellipsis
          whitespace-nowrap
          text-primary
          op80
          hover:op100
          mr-4
          @click="handleOpenCustomizeUrl"
        >{{ shareUrl }}</button>
        <button
          shrink-0
          text-primary
          op80
          hover:op100
          @click="handleShareCustomizeUrl"
        >{{ t("share") }}</button>
      </div>
    </div>
  </div>
  <Modal v-model="shareDialog" direction="top">
    <div flex="~ col" p5 items-center>
      <p text-xl font-serif mb4>
        <b>{{ t('share') }}</b>
      </p>
      <p text-center mb4>{{ t('fake-share-copied') }}</p>
    </div>
  </Modal>
  <Modal v-model="illegal" direction="top">
    <div flex="~ col" p5 items-center>
      <p text-xl font-serif mb4>
        <b>{{ t('fake-illegal') }}</b>
      </p>
      <p text-center mb4>{{ t('fake-illegal-taunt') }}</p>
    </div>
  </Modal>
</template>