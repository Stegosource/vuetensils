<template>
  <transition :name="bgTransition">
    <div
      v-if="showing"
      @click="onClick"
      @keydown="onKeydown"
      class="vts-modal"
    >
      <transition
        :name="transition"
        appear
      >
        <component
          :is="tag"
          ref="content"
          :style="{width: width, maxWidth: maxWidth}"
          class="vts-modal__content"
          tabindex="-1"
          role="dialog"
        >
          <!-- @slot Content that exists within the modal. -->
          <slot />
        </component>
      </transition>
    </div>
  </transition>
</template>

<script>
import KEYCODES from "../../data/keycodes"
import FOCUSABLE from "../../data/focusable"

/**
 * A modal/dialogue component for showing users content which overlays the rest of the applications. When opened, it traps the user's focus so that keyboard navigation will remain within the modal until it is closed. It supports being closed by clicking outside the modal content or pressing the ESC key.
 */
export default {
  model: {
    prop: "showing",
    event: "change"
  },

  props: {
    /**
     * @model
     */
    showing: Boolean,
    /**
     * HTML component for the modal content.
     */
    tag: {
      type: String,
      default: "div"
    },
    /**
     * Flag to enable/prevent the modal from being closed.
     */
    dismissible: {
      type: Boolean,
      default: true
    },
    /**
     * CSS width to set the modal to.
     */
    width: String,
    /**
     * CSS max-width to set the modal to.
     */
    maxWidth: String,
    /**
     * Prevents the page from being scrolled while the modal is open.
     */
    noScroll: {
      type: Boolean,
      default: false
    },
    /**
     * Transition name to apply to the modal.
     */
    transition: String,
    /**
     * Transition name to apply to the background.
     */
    bgTransition: String
  },

  watch: {
    showing: {
      handler(next, prev) {
        if (typeof window !== "undefined") {
          if (next && next != prev) {
            this.noScroll && document.body.style.setProperty("overflow", "hidden")
            this.$nextTick(() => {
              this.$refs.content.focus()
            })
          } else {
            this.noScroll && document.body.style.removeProperty("overflow")
          }
        }
      }
    }
  },

  methods: {
    show() {
      /**
       * Fired when the modal opens.
       * @event show
       * @type { boolean }
       */
      this.$emit("show")
      this.$emit("change", true)
    },
    hide() {
      /**
       * Fired when the modal closes.
       * @event hide
       * @type { boolean }
       */
      this.$emit("hide")
      this.$emit("change", false)
    },
    toggle() {
      const event = this.showing ? "hide" : "show"
      this.$emit(event, !this.showing)
      /**
       * Fired whenever the modal opens or closes.
       * @event change
       * @type { boolean }
       */
      this.$emit("change", !this.showing)
    },
    onClick(event) {
      if (event.target.classList.contains("vts-modal") && this.dismissible) {
        this.hide()
      }
    },

    onKeydown(event) {
      if (event.keyCode === KEYCODES.ESC) {
        this.hide()
      }
      if (event.keyCode === KEYCODES.TAB) {
        const content = this.$refs.content
        const focusable = Array.from(content.querySelectorAll(FOCUSABLE))

        if (this.showing && content && !content.contains(document.activeElement) && focusable) {
          event.preventDefault()
          focusable[0].focus()
        } else {
          const focusedItemIndex = focusable.indexOf(document.activeElement)

          if (event.shiftKey && focusedItemIndex === 0) {
            focusable[focusable.length - 1].focus()
            event.preventDefault()
          }

          if (!event.shiftKey && focusedItemIndex === focusable.length - 1) {
            focusable[0].focus()
            event.preventDefault()
          }
        }
      }
    }
  }
}
</script>

<style>
.vts-modal {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  z-index: 100;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.2);
}

.vts-modal [tabindex="-1"]:focus {
  outline: 0;
}

.vts-modal__content {
  overflow: auto;
  max-width: 70vw;
  max-height: 80vh;
  background: #fff;
}
</style>