<script lang="ts">
import type { TooltipRootProps, TooltipRootEmits, TooltipContentProps, TooltipContentEmits, TooltipArrowProps, TooltipTriggerProps } from 'reka-ui'
import type { AppConfig } from '@nuxt/schema'
import theme from '#build/b24ui/tooltip'
import type { KbdProps } from '../types'
import type { EmitsToProps, ComponentConfig } from '../types/utils'

type Tooltip = ComponentConfig<typeof theme, AppConfig, 'tooltip'>

export interface TooltipProps extends TooltipRootProps {
  /** The text content of the tooltip. */
  text?: string
  /** The keyboard keys to display in the tooltip. */
  kbds?: KbdProps['value'][] | KbdProps[]
  /**
   * The content of the tooltip.
   * @defaultValue { side: 'bottom', sideOffset: 8, collisionPadding: 8 }
   */
  content?: Omit<TooltipContentProps, 'as' | 'asChild'> & Partial<EmitsToProps<TooltipContentEmits>>
  /**
   * Display an arrow alongside the tooltip.
   * @defaultValue false
   */
  arrow?: boolean | Omit<TooltipArrowProps, 'as' | 'asChild'>
  /**
   * Render the tooltip in a portal.
   * @defaultValue true
   */
  portal?: boolean | string | HTMLElement
  reference?: TooltipTriggerProps['reference']
  class?: any
  b24ui?: Tooltip['slots']
}

export interface TooltipEmits extends TooltipRootEmits {}

export interface TooltipSlots {
  default(props: { open: boolean }): any
  content(props?: {}): any
}
</script>

<script setup lang="ts">
import { computed, toRef } from 'vue'
import { defu } from 'defu'
import { TooltipRoot, TooltipTrigger, TooltipPortal, TooltipContent, TooltipArrow, useForwardPropsEmits } from 'reka-ui'
import { reactivePick } from '@vueuse/core'
import { useAppConfig } from '#imports'
import { usePortal } from '../composables/usePortal'
import { tv } from '../utils/tv'
import B24Kbd from './Kbd.vue'

const props = withDefaults(defineProps<TooltipProps>(), {
  portal: true
})
const emits = defineEmits<TooltipEmits>()
const slots = defineSlots<TooltipSlots>()

const appConfig = useAppConfig() as Tooltip['AppConfig']

const rootProps = useForwardPropsEmits(reactivePick(props, 'defaultOpen', 'open', 'delayDuration', 'disableHoverableContent', 'disableClosingTrigger', 'disabled', 'ignoreNonKeyboardFocus'), emits)
const portalProps = usePortal(toRef(() => props.portal))
const contentProps = toRef(() => defu(props.content, { side: 'bottom', sideOffset: 8, collisionPadding: 8 }) as TooltipContentProps)
const arrowProps = toRef(() => props.arrow as TooltipArrowProps)

// eslint-disable-next-line vue/no-dupe-keys
const b24ui = computed(() => tv({ extend: tv(theme), ...(appConfig.b24ui?.tooltip || {}) })({
  side: contentProps.value.side
}))
</script>

<template>
  <TooltipRoot v-slot="{ open }" v-bind="rootProps">
    <TooltipTrigger
      v-if="!!slots.default || !!reference"
      v-bind="$attrs"
      as-child
      :reference="reference"
      :class="props.class"
    >
      <slot :open="open" />
    </TooltipTrigger>

    <TooltipPortal v-bind="portalProps">
      <TooltipContent v-bind="contentProps" :class="b24ui.content({ class: [!slots.default && props.b24ui?.content, props.class] })">
        <slot name="content">
          <span v-if="text" :class="b24ui.text({ class: props.b24ui?.text })">{{ text }}</span>

          <span v-if="kbds?.length" :class="b24ui.kbds({ class: props.b24ui?.kbds })">
            <B24Kbd
              v-for="(kbd, index) in kbds"
              :key="index"
              :size="((props.b24ui?.kbdsSize || b24ui.kbdsSize()) as KbdProps['size'])"
              :depth="((props.b24ui?.kbdsDepth || b24ui.kbdsDepth()) as KbdProps['depth'])"
              v-bind="typeof kbd === 'string' ? { value: kbd } : kbd"
            />
          </span>
        </slot>

        <TooltipArrow v-if="!!arrow" v-bind="arrowProps" :class="b24ui.arrow({ class: props.b24ui?.arrow })" />
      </TooltipContent>
    </TooltipPortal>
  </TooltipRoot>
</template>
