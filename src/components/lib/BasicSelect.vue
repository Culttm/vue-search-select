<template>
  <div class="dropdown"
       :class="{ 'active visible':showMenu, 'error': isError, 'disabled': !options.length || options.length === 0 }"
       @click="openOptions">
    <slot name="trigger-icon"></slot>
    <input class="search"
           autocomplete="off"
           tabindex="0"
           v-model="searchText"
           v-if="options.length > 0"
           ref="input"
           @blur="blurInput"
           @keydown.up="prevItem"
           @keydown.down="nextItem"
           @keyup.enter="enterItem"
           @keydown.delete="deleteTextOrItem"
    />
    <div class="text"
         :class="textClass">{{inputText}}
    </div>
    <div class="menu"
         ref="menu"
         @mousedown.prevent
         :class="menuClass"
         :style="menuStyle"
         tabindex="-1">
      <div class="menu__inner">
        <template v-for="(option, idx) in filteredOptions">
          <div class="item"
               :class="{ 'selected': option.selected, 'current': pointer === idx }"
               @click.stop="selectItem(option)"
               @mousedown="mousedownItem"
               @mouseenter="pointerSet(idx)">
            {{option.text}}
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
  /* event : select */
  import common from './common'
  import commonMixin from './commonMixin'

  export default {
    mixins: [commonMixin],
    mounted(){
        console.log(this.$slots)
    },
    props: {
      disabled: {
        type: Boolean
      },
      options: {
        type: Array
      },
      selectedOption: {
        type: Object,
        default: () => {
          return { value: '', text: '' }
        }
      }
    },
    data () {
      return {
        showMenu: false,
        searchText: '',
        mousedownState: false, // mousedown on option menu
        pointer: 0
      }
    },
    watch: {
      filteredOptions () {
        this.pointerAdjust()
      }
    },
    computed: {
      inputText () {
        if (this.searchText) {
          return ''
        } else {
          let text = this.placeholder
          if (this.selectedOption.text) {
            text = this.selectedOption.text
          }
          return text
        }
      },
      textClass () {
        if (!this.selectedOption.text && this.placeholder) {
          return 'default'
        } else {
          return ''
        }
      },
      menuClass () {
        return {
          visible: this.showMenu,
          hidden: !this.showMenu
        }
      },
      menuStyle () {
        return {
          display: this.showMenu ? 'block' : 'none'
        }
      },
      filteredOptions () {
        if (this.searchText) {
          return this.options.filter((option) => {
            try {
              return this.filterPredicate(option.text, this.searchText)
            } catch (e) {
              return true
            }
          })
        } else {
          return this.options
        }
      }
    },
    methods: {
      deleteTextOrItem () {
        if (!this.searchText && this.selectedOption) {
          this.selectItem({})
          this.openOptions()
        }
      },
      openOptions () {
        if (this.options.length > 0) {
          common.openOptions(this)
        }
      },
      blurInput () {
        common.blurInput(this)
      },
      closeOptions () {
        common.closeOptions(this)
      },
      prevItem () {
        common.prevItem(this)
      },
      nextItem () {
        common.nextItem(this)
      },
      enterItem () {
        common.enterItem(this)
      },
      pointerSet (index) {
        common.pointerSet(this, index)
      },
      pointerAdjust () {
        common.pointerAdjust(this)
      },
      mousedownItem () {
        common.mousedownItem(this)
      },
      selectItem (option) {
        this.searchText = '' // reset text when select item
        this.closeOptions()
        this.$emit('select', option)
      }
    }
  }
</script>
