<template>
  <div
    v-if="backdropShown"
    class="backdrop pa-6"
    :class="{ opaque: contentShown }"
    @mousedown.self="close"
    @keydown.esc="close"
    tabindex="0">
    <v-slide-y-transition>
    <v-sheet
      v-if="contentShown"
      class="dialog pa-6 rounded-lg"
      elevation="24" max-width="650">
      <div class="text-h5 text-truncate mb-6">
        {{ title }}
      </div>
      <slot></slot>
      <div class="d-flex mt-6">
        <slot name="left-buttons"/>
        <v-spacer></v-spacer>
        <v-btn
          large depressed
          @click="close">
          {{ $t('CLOSE') }}
        </v-btn>
        <slot name="right-buttons"/>
      </div>
    </v-sheet>
    </v-slide-y-transition>
  </div>
</template>

<style scoped>
.backdrop {
  background-color: rgba(128, 128, 128, 0.3);
  backdrop-filter: blur(5px);
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1000;
  overflow-x: hidden;
  overflow-y: hidden;
  opacity: 0.0;
  transition: opacity 0.15s ease-out;
}
.opaque {
  opacity: 1.0;
  /* The scrollbar shows up only when the dialog is taller than the viewport:
     with a reserved one the dialog would sit off the center of the page */
  overflow-y: auto;
}
.dialog {
  margin-top: auto;
  margin-bottom: auto;
  margin-right: auto;
  margin-left: auto;
}
</style>

<script>
// While the dialog is shown, the page below the app bar stays put: the
// scrollbar keeps its reserved gutter and the backdrop holds the scrollable
// content out of reach, so no compensation of the layout is needed

export default {
  props: {
    title: String,
    value: Boolean
  },
  emits: ['input', 'closed'],
  data: () => ({
    ready: false
  }),
  computed: {
    backdropShown () {
      return this.value || this.ready
    },
    contentShown () {
      return this.value && this.ready
    }
  },
  watch: {
    value (value) {
      clearTimeout(this.readyTimeout)
      this.readyTimeout = setTimeout(() => {
        this.ready = value
      }, value ? 0 : 200)
    },
    backdropShown (value) {
      if (!value) this.$emit('closed')
    }
  },
  methods: {
    handleKeyPress (event) {
      if (this.value && (event.which === 27)) this.close()
    },
    close () {
      this.$emit('input', undefined)
    }
  },
  created () {
    window.addEventListener('keydown', this.handleKeyPress)
  },
  beforeDestroy () {
    window.removeEventListener('keydown', this.handleKeyPress)
  }
}
</script>
