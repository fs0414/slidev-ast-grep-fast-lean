<script setup lang="ts">
import GradientHeading from './GradientHeading.vue'
import SocialLinks from './SocialLinks.vue'

interface Props {
  title: string
  subtitle?: string
  event?: string
  author?: string
  social?: {
    github?: string
    twitter?: string
    linkedin?: string
  }
  gradient?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  gradient: true,
})
</script>

<template>
  <div class="cover-slide">
    <div class="cover-content mt-20">
      <h1 v-if="!gradient" class="cover-title">
        {{ title }}
      </h1>
      <GradientHeading v-else tag="h1" class="cover-title">
        {{ title }}
      </GradientHeading>

      <div v-if="subtitle" class="mt-8 cover-subtitle">
        {{ subtitle }}
      </div>

      <div v-if="event || author" class="mt-16 cover-meta">
        <span v-if="event">{{ event }}</span>
        <span v-if="event && author"> / </span>
        <span v-if="author">{{ author }}</span>
      </div>
    </div>

    <div v-if="social" class="cover-social">
      <SocialLinks
        :github="social.github"
        :twitter="social.twitter"
        :linkedin="social.linkedin"
        size="xl"
      />
    </div>
  </div>
</template>

<style>
/* グローバルスタイル：CoverSlideを含むスライドのレイアウトをリセット */
.slidev-layout:has(.cover-slide) {
  display: block !important;
  justify-content: initial !important;
  padding: 0 !important;
}
</style>

<style scoped>
.cover-slide {
  width: 100%;
  height: 100%;
  position: relative;
  padding: 7% 3%;
  display: flex;
  flex-direction: column;
}

.cover-content {
  max-width: 100%;
  flex: 1;
}

.cover-title {
  margin-bottom: 0;
  font-size: clamp(2rem, 5vw, 3.5rem) !important;
  line-height: 1.1;
}

.cover-subtitle {
  font-size: clamp(1rem, 2.5vw, 1.25rem);
  line-height: 1.6;
}

.cover-meta {
  font-size: 0.875rem;
  text-align: right;
  color: oklch(0.5 0.05 270);
}

.cover-social {
  margin-top: auto;
  align-self: flex-end;
  padding-top: 2rem;
  padding-bottom: 0.5rem;
}
</style>
