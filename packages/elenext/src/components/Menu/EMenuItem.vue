<template>
  <li
    :class="{
      'el-menu-item': true,
      'is-active': state.isActive,
    }"
    :style="styles"
    @mouseenter="mouseenterHandler"
    @mouseleave="mouseleaveHandler"
    @click="clickHandler"
  >
    <slot />
  </li>
</template>

<script lang="ts">
import {
  computed,
  CSSProperties,
  defineComponent,
  getCurrentInstance,
  inject,
  onMounted,
  onUnmounted,
  reactive,
  watchEffect,
} from 'vue'
import vptypes from 'vptypes'
import { MENU_IJK, MENU_ITEM_PADDING, MENU_TYPE } from './core'

const EMenuItem = defineComponent({
  name: 'EMenuItem',
  props: {
    path: vptypes.string(),
  },
  emits: ['click'],
  setup(props, { emit }) {
    const self = getCurrentInstance()

    const menuProvider = inject(MENU_IJK)

    const state = reactive({
      root: menuProvider!.root,
      path: props.path,
      type: MENU_TYPE.ITEM,
      uid: self!.uid,
      uidPath: [...menuProvider!.uidPath, self!.uid],
      deep: menuProvider!.deep + 1,
      isOpen: false,
      isHover: false,
      isActive: menuProvider!.root.activeId === self!.uid,
      isPopper: menuProvider!.root.mode !== 'vertical' && menuProvider!.type === MENU_TYPE.SUB,
    })
    watchEffect(() => {
      state.isActive = menuProvider!.root.activeId === self!.uid
      state.isPopper = menuProvider!.root.mode !== 'vertical' && menuProvider!.type === MENU_TYPE.SUB
    })

    const styles = computed<CSSProperties>(() => {
      return {
        color: state.isActive ? state.root.activeTextColor : state.root.textColor,
        backgroundColor:
          state.isActive || state.isHover ? state.root.activeBackgroundColor : state.root.backgroundColor,
        borderColor: !state.isPopper && state.isActive ? state.root.activeTextColor : 'transparent',
        paddingLeft: state.isPopper ? MENU_ITEM_PADDING + 'px' : state.deep * MENU_ITEM_PADDING + 'px',
      }
    })

    const clickHandler = (event: MouseEvent) => {
      state.root.methods.select(state)
      emit('click', event)
      const router = self!.appContext.config.globalProperties.$router
      if (router && props.path) {
        router.push(props.path)
      }
    }
    const mouseenterHandler = (event: MouseEvent) => (state.isHover = true)
    const mouseleaveHandler = (event: MouseEvent) => (state.isHover = false)

    onMounted(() => {
      menuProvider!.root.children.push(state)
    })

    onUnmounted(() => {
      const index = menuProvider!.root.children.indexOf(state)
      menuProvider!.root.children.splice(index, 1)
    })
    return {
      state,
      styles,
      clickHandler,
      mouseenterHandler,
      mouseleaveHandler,
    }
  },
})

export default EMenuItem
</script>
