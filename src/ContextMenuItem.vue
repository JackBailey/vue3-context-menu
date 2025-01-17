<template>
  <div class="mx-context-menu-item-wrapper">
    <!--Custom render-->
    <VNodeRender v-if="globalHasSlot('itemRender')" :vnode="() => globalRenderSlot('itemRender', getItemDataForChildren())" />
    <VNodeRender v-else-if="customRender" :vnode="customRender" :data="getItemDataForChildren()" />
    <!--Default item-->
    <div 
      v-else
      :class="[
        'mx-context-menu-item',
        (disabled ? 'disabled' : ''),
        (customClass ? (' ' + customClass) : ''),
        (showSubMenu ? 'open' : ''),
      ]"
      @click="onClick"
      @mouseenter="onMouseEnter"
    >
      <slot>
        <slot name="icon">
          <VNodeRender v-if="globalHasSlot('itemIconRender')" :vnode="() => globalRenderSlot('itemIconRender', getItemDataForChildren())" />
          <svg v-else-if="typeof svgIcon === 'string' && svgIcon" class="icon svg" v-bind="svgProps">
            <use :xlink:href="svgIcon"></use>
          </svg>
          <i v-else-if="typeof icon === 'string'" :class="icon + ' icon '+ iconFontClass + ' ' + globalIconFontClass"></i>
          <VNodeRender v-else :vnode="icon" :data="icon" />
        </slot>
        <slot name="label">
          <VNodeRender v-if="globalHasSlot('itemLabelRender')" :vnode="() => globalRenderSlot('itemLabelRender', getItemDataForChildren())" />
          <span v-else-if="typeof label === 'string'">{{ label }}</span>
          <VNodeRender v-else :vnode="label" :data="label" />
        </slot>
        <slot v-if="showRightArrow" name="rightArrow">
          <VNodeRender v-if="globalHasSlot('itemRightArrowRender')" :vnode="() => globalRenderSlot('itemRightArrowRender', getItemDataForChildren())" />
          <span class="mx-right-arrow" />
        </slot>
      </slot>
    </div>
    
    <!--Sub menu render-->
    <slot v-if="showSubMenu" name="submenu"></slot>
  </div>
</template>

<script lang="ts">
import { defineComponent, inject, PropType, ref, SVGAttributes, toRefs } from 'vue'
import { SubMenuParentContext } from './ContextSubMenu.vue'
import { GlobalHasSlot, GlobalRenderSlot } from './ContextMenu.vue'
import { VNodeRender } from './ContextMenuUtils'

/**
 * Menu Item
 */
export default defineComponent({
  name: 'ContextMenuItem',
  components: {
    VNodeRender,
  },
  props: {
    /**
     * Is this menu disabled? 
     */
    disabled: {
      type: Boolean,
      default: false
    },
    customRender: {
      type: Function,
      default: null
    },
    customClass: {
      type: String,
      default: ''
    },
    clickHandler: {
      type: Function as PropType<() => void>,
      default: null
    },
    /**
     * Menu label
     */
    label: {
      type: [String, Object],
      default: ''
    },
    /**
     * Menu icon (for icon class)
     */
    icon: {
      type: [String, Object],
      default: ''
    },
    iconFontClass: {
      type: String,
      default: 'iconfont'
    },
    /**
     * Menu icon (for svg)
     */
    svgIcon: {
      type: String,
      default: ''
    },
    svgProps: {
      type: Object as PropType<SVGAttributes>,
      default: null
    },
    /**
     * Show right arrow on this menu?
     */
    showRightArrow: {
      type: Boolean,
      default: false
    },
    hasChildren: {
      type: Boolean,
      default: false
    },
    /**
     * Should close menu when Click this menu item ?
     */
    clickClose: {
      type: Boolean,
      default: true
    },
    /**
     * When there are subitems in this item, is it allowed to trigger its own click event? Default is false
     */
    clickableWhenHasChildren: {
      type: Boolean,
      default: false
    },
  },
  emits: [
    'click'
  ],
  setup(props, context) {

    const { 
      clickHandler, clickClose, clickableWhenHasChildren, disabled,
      label, icon, iconFontClass,
      showRightArrow,
      hasChildren,
    } = toRefs(props);
    const showSubMenu = ref(false);

    const globalHasSlot = inject('globalHasSlot') as GlobalHasSlot;
    const globalRenderSlot = inject('globalRenderSlot') as GlobalRenderSlot;
    const menuContext = inject('menuContext') as SubMenuParentContext;
    const globalCloseMenu = inject('globalCloseMenu') as () => void;

    function onClick(e: MouseEvent) {
      if (
        (e.target as HTMLElement).classList.contains('mx-context-no-clickable')
        || disabled.value
      )
        return;
      if (hasChildren.value) {//Has submenu
        if (clickableWhenHasChildren.value && typeof clickHandler.value === 'function') 
          clickHandler.value();
      } else {
        //Call hander from options
        if (typeof clickHandler.value === 'function') 
          clickHandler.value();
        if (clickClose.value) {
          //emit close
          globalCloseMenu();
        }
      }
      context.emit('click');
    }
    function onMouseEnter() {
      if (!menuContext.checkCloseOtherSubMenuTimeOut())
        menuContext.closeOtherSubMenu();
      if (!disabled.value && hasChildren.value) {
        menuContext.addOpenedSubMenu(() => {
          showSubMenu.value = false;
        });
        showSubMenu.value = true;
      }
    }

    const globalTheme = inject('globalTheme') as string;
    const globalIconFontClass = inject('globalIconFontClass') as string;

    return {
      //Data for custom render
      getItemDataForChildren() {
        return {
          disabled: disabled.value,
          label: label.value,
          icon: icon.value,
          iconFontClass: iconFontClass.value,
          showRightArrow: showRightArrow.value,
          clickClose: clickClose.value,
          clickableWhenHasChildren: clickableWhenHasChildren.value,
          theme: globalTheme,
          isOpen: showSubMenu,
          hasChildren: hasChildren,
          onClick,
          onMouseEnter,
        }
      },
      showSubMenu,
      globalHasSlot,
      globalRenderSlot,
      globalIconFontClass,
      onMouseEnter,
      onClick,
    }
  },
})
</script>

<style>
</style>