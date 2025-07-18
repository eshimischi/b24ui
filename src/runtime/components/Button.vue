<script lang="ts">
import type { AppConfig } from '@nuxt/schema'
import theme from '#build/b24ui/button'
import type { UseComponentIconsProps } from '../composables/useComponentIcons'
import type { LinkProps, AvatarProps } from '../types'
import type { ComponentConfig } from '../types/utils'

type Button = ComponentConfig<typeof theme, AppConfig, 'button'>

export interface ButtonProps extends Omit<UseComponentIconsProps, 'trailing' | 'trailingIcon'>, Omit<LinkProps, 'raw' | 'custom'> {
  label?: string
  /**
   * @defaultValue 'default'
   */
  color?: Button['variants']['color']
  activeColor?: Button['variants']['color']
  /**
   * @defaultValue 'normal'
   */
  depth?: Button['variants']['depth']
  activeDepth?: Button['variants']['depth']
  /**
   * @defaultValue 'md'
   */
  size?: Button['variants']['size']
  /**
   * Rounds the corners of the button
   * @defaultValue false
   */
  rounded?: boolean
  /**
   * Render the button full width
   * @defaultValue false
   */
  block?: boolean
  /**
   * Set loading state automatically based on the `@click` promise state
   * @defaultValue false
   */
  loadingAuto?: boolean
  /**
   * Disable uppercase label
   * @defaultValue false
   */
  normalCase?: boolean
  /**
   * Shows LoaderWaitIcon in loading mode
   * @defaultValue false
   */
  useWait?: boolean
  /**
   * Shows LoaderClockIcon icon in loading mode
   * @defaultValue false
   */
  useClock?: boolean
  /**
   * Shows icons.chevronDown on the right side
   * @defaultValue false
   */
  useDropdown?: boolean
  onClick?: ((event: MouseEvent) => void | Promise<void>) | Array<((event: MouseEvent) => void | Promise<void>)>
  class?: any
  /**
   * The class to apply when the link is active
   * @defaultValue ''
   */
  activeClass?: string
  /**
   * The class to apply when the link is inactive
   * @defaultValue ''
   */
  inactiveClass?: string
  b24ui?: Button['slots']
}

export interface ButtonSlots {
  leading(props?: {}): any
  default(props?: {}): any
  trailing(props?: {}): any
}
</script>

<script setup lang="ts">
import { type Ref, computed, ref, inject } from 'vue'
import { defu } from 'defu'
import { useForwardProps } from 'reka-ui'
import { useAppConfig } from '#imports'
import { useComponentIcons } from '../composables/useComponentIcons'
import { useButtonGroup } from '../composables/useButtonGroup'
import { formLoadingInjectionKey } from '../composables/useFormField'
import { omit } from '../utils'
import { tv } from '../utils/tv'
import { pickLinkProps } from '../utils/link'
import B24Avatar from './Avatar.vue'
import B24Link from './Link.vue'
import B24LinkBase from './LinkBase.vue'
import ChevronDownIcon from '@bitrix24/b24icons-vue/actions/ChevronDownIcon'
import LoaderWaitIcon from '@bitrix24/b24icons-vue/animated/LoaderWaitIcon'
import LoaderClockIcon from '@bitrix24/b24icons-vue/animated/LoaderClockIcon'
import SpinnerIcon from '@bitrix24/b24icons-vue/specialized/SpinnerIcon'

const props = withDefaults(defineProps<ButtonProps>(), {
  type: 'button',
  active: undefined,
  activeClass: '',
  inactiveClass: ''
})

const slots = defineSlots<ButtonSlots>()

const appConfig = useAppConfig() as Button['AppConfig']

const { orientation, size: buttonSize, noSplit } = useButtonGroup<ButtonProps>(props)

const linkProps = useForwardProps(pickLinkProps(props))

/**
 * @memo Problem use at Docs (vue): omit not exports
 */
const proxyLinkProps = computed(() => {
  return omit(linkProps.value, ['type', 'disabled', 'onClick'])
})
const loadingAutoState = ref(false)
const formLoading = inject<Ref<boolean> | undefined>(formLoadingInjectionKey, undefined)

async function onClickWrapper(event: MouseEvent) {
  loadingAutoState.value = true
  const callbacks = Array.isArray(props.onClick) ? props.onClick : [props.onClick]
  try {
    await Promise.all(callbacks.map(fn => fn?.(event)))
  } finally {
    loadingAutoState.value = false
  }
}

const isLoading = computed(() => {
  return props.loading || (props.loadingAuto && (loadingAutoState.value || (formLoading?.value && props.type === 'submit')))
})

const { isLeading, leadingIconName } = useComponentIcons(
  computed(() => ({ ...props, loading: false }))
)

const isLabel = computed(() => {
  let isUseSlot = false

  if (slots && !!slots.default) {
    isUseSlot = true
  }

  return (props?.label || '').length > 0 || isUseSlot
})

const b24ui = computed(() => tv({
  extend: tv(theme),
  ...defu({
    variants: {
      active: {
        true: {
          base: props.activeClass
        },
        false: {
          base: props.inactiveClass
        }
      }
    }
  }, appConfig.b24ui?.button || {})
})({
  color: props.color,
  depth: props.depth,
  size: buttonSize.value,
  noSplit: Boolean(noSplit.value),
  loading: Boolean(isLoading.value),
  useLabel: Boolean(isLabel.value),
  block: Boolean(props.block),
  normalCase: Boolean(props.normalCase),
  rounded: Boolean(props.rounded),
  useDropdown: Boolean(props.useDropdown),
  useWait: Boolean(props.useWait),
  useClock: Boolean(props.useClock),
  leading: Boolean(isLeading.value),
  buttonGroup: orientation.value
}))
</script>

<template>
  <B24Link
    v-slot="{ active, ...slotProps }"
    :type="type"
    :disabled="disabled || isLoading"
    v-bind="proxyLinkProps"
    custom
  >
    <B24LinkBase
      v-bind="slotProps"
      :class="b24ui.base({
        class: [props.b24ui?.base, props.class],
        active,
        ...(active && activeDepth ? { depth: activeDepth } : {}),
        ...(active && activeColor ? { color: activeColor } : {})
      })"
      @click="onClickWrapper"
    >
      <div
        v-if="isLoading"
        class="h-full w-full absolute inset-0 flex flex-row flex-nowrap items-center justify-center"
      >
        <LoaderWaitIcon v-if="useWait" class="w-[28px] h-[28px]" aria-hidden="true" />
        <LoaderClockIcon v-else-if="useClock" class="w-[28px] h-[28px]" aria-hidden="true" />
        <SpinnerIcon v-else class="size-lg animate-spin stroke-2" aria-hidden="true" />
      </div>
      <div
        :class="[
          b24ui.baseLine({ class: [props.b24ui?.baseLine] }),
          isLoading ? 'invisible' : ''
        ]"
      >
        <slot name="leading">
          <Component
            :is="leadingIconName"
            v-if="isLeading && (typeof leadingIconName !== 'undefined')"
            :class="b24ui.leadingIcon({ class: props.b24ui?.leadingIcon })"
          />
          <B24Avatar
            v-else-if="!!avatar"
            :size="((props.b24ui?.leadingAvatarSize || b24ui.leadingAvatarSize()) as AvatarProps['size'])"
            v-bind="avatar"
            :class="b24ui.leadingAvatar({ class: props.b24ui?.leadingAvatar })"
          />
        </slot>

        <slot>
          <span v-if="label !== undefined && label !== null" :class="b24ui.label({ class: props.b24ui?.label, active })">
            {{ label }}
          </span>
        </slot>

        <slot name="trailing">
          <ChevronDownIcon
            v-if="useDropdown"
            :class="b24ui.trailingIcon({ class: props.b24ui?.trailingIcon })"
            aria-hidden="true"
          />
        </slot>
      </div>
    </B24LinkBase>
  </B24Link>
</template>
