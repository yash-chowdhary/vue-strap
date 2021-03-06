<template>
  <div role="dialog"
    :class="[
    {
      'modal':true,
      'fade':effect === 'fade',
      'zoom':effect === 'zoom'
    }, addClass]">
    <div v-bind:class="{'modal-dialog':true,'modal-lg':largeBool,'modal-sm':smallBool,'modal-dialog-centered':centerBool}" role="document"
      v-bind:style="{width: optionalWidth}">
      <div class="modal-content">
        <slot name="header">
          <div class="modal-header">
            <h4 class="modal-title">
              <span name="title">
                <slot name="_header"></slot>
              </span>
            </h4>
            <button type="button" class="close" @click="close"><span>&times;</span></button>
          </div>
        </slot>
          <div class="modal-body">
              <slot></slot>
          </div>
        <slot name="footer">
          <div class="modal-footer" v-if="showOkButton">
            <button type="button" class="btn btn-primary" v-if="showOkButton" @click="close">{{ okText }}</button>
          </div>
        </slot>
      </div>
    </div>
  </div>
</template>

<script>
import {globalEventBus} from './GlobalEventBus.js'
import {toBoolean, getScrollBarWidth} from './utils/utils.js'
import $ from './utils/NodeList.js'

export default {
  props: {
    okText: {
      type: String,
      default: ''
    },
    width: {
      default: null
    },
    effect: {
      type: String,
      default: 'zoom'
    },
    backdrop: {
      type: Boolean,
      default: true
    },
    large: {
      type: Boolean,
      default: false
    },
    small: {
      type: Boolean,
      default: false
    },
    center: {
      type: Boolean,
      default: false
    },
    name: {
      type: String
    },
    id: {
      type: String
    },
    addClass: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      show: false
    }
  },
  computed: {
    backdropBool () {
      return toBoolean(this.backdrop)
    },
    largeBool () {
      return toBoolean(this.large)
    },
    smallBool () {
      return toBoolean(this.small)
    },
    centerBool () {
      return toBoolean(this.center)
    },
    optionalWidth () {
      if (this.width === null) {
        return null
      } else if (Number.isInteger(this.width)) {
        return this.width + 'px'
      }
      return this.width
    },
    showOkButton () {
      return this.okText.length !== 0;
    }
  },
  watch: {
    show (val) {
      const el = this.$el
      const body = document.body
      const scrollBarWidth = getScrollBarWidth()
      if (val) {
        $(el).find('.modal-content').focus()
        el.style.display = 'block'
        setTimeout(() => $(el).addClass('show in'), 0)
        $(body).addClass('modal-open')
        if (scrollBarWidth !== 0) {
          body.style.paddingRight = scrollBarWidth + 'px'
        }
        if (this.backdropBool) {
          $(el).on('click', e => {
            if (e.target === el) this.show = false
          })
        }
      } else {
        body.style.paddingRight = null
        $(body).removeClass('modal-open')
        $(el).removeClass('show in').on('transitionend', () => {
          $(el).off('click transitionend')
          el.style.display = 'none'
        })
      }
    }
  },
  created () {
    globalEventBus.$on('trigger:bind', this.bindTrigger)
  },
  methods: {
    bindTrigger (trigger, popover) {
      if (popover === this.id) {
        trigger.setTriggerBy(this)
      }
    },
    showModal (name) {
      if (name === this.name) {
        this.show = true
      }
    },
    close () {
      this.show = false
    },
    toggle () {
      this.show = true
    }
  },
  beforeDestroy () {
    globalEventBus.$off('trigger:bind', this.bindTrigger)
  }
}
</script>
<style>
.modal {
  transition: all 0.3s ease;
}
.modal.in {
  background-color: rgba(0,0,0,0.5);
}
.modal.zoom .modal-dialog {
  -webkit-transform: scale(0.1);
  -moz-transform: scale(0.1);
  -ms-transform: scale(0.1);
  transform: scale(0.1);
  top: 300px;
  opacity: 0;
  -webkit-transition: all 0.3s;
  -moz-transition: all 0.3s;
  transition: all 0.3s;
}
.modal.zoom.in .modal-dialog {
  -webkit-transform: scale(1);
  -moz-transform: scale(1);
  -ms-transform: scale(1);
  transform: scale(1);
  -webkit-transform: translate3d(0, -300px, 0);
  transform: translate3d(0, -300px, 0);
  opacity: 1;
}
</style>
