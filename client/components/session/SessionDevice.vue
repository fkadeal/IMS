<script lang="ts" setup>

export interface Props {
  session: api.Session
}

const props = defineProps<Props>()
const emit = defineEmits<{ (event: 'refresh'): void }>()
const api = useApi()
const dayjs = useDayjs()

const type = computed((): string => {
  if (props.session.device.platform.includes('macOS'))
    return 'mac'
  if (props.session.device.platform.includes('OS X'))
    return 'mac'
  if (props.session.device.platform.includes('Windows'))
    return 'windows'
  if (props.session.device.platform.includes('Linux'))
    return 'linux'
  // if (this.session.device.platform.includes('iOS')) return 'ios'
  if (props.session.agent.includes('iPhone'))
    return 'iphone'
  if (props.session.agent.includes('iPad'))
    return 'ipad'
  if (props.session.agent.includes('Android'))
    return 'android'
  if (props.session.agent.includes('Chrome'))
    return 'chrome'
  if (props.session.agent.includes('Safari'))
    return 'chrome'
  if (props.session.agent.includes('Edge'))
    return 'edge'
  return 'other'
})

const source = computed((): string => {
  if (props.session.source === 'actions')
    return 'Github Actions'
  if (props.session.source === 'pipelines')
    return 'Bitbucket Pipelines'
  if (props.session.source === 'cli')
    return 'Fume CLI'
  if (props.session.source === 'ci')
    return 'CI / CD'
  return 'Unknown'
})

const name = computed((): string => {
  if (props.session.device.name)
    return props.session.device.name
  if (props.session.device.platform)
    return props.session.device.platform
  return props.session.agent
})

function confirm(session: api.Session) {
  useConfirm().confirm(
    'Delete Session',
    'Are you sure you want to delete this session ?',
    'Delete Session',
    () => revoke(session),
  )
}

async function revoke(session: api.Session) {
  if (session.current)
    return api.logout()
  const response = await api.delete(`/session/${session.token}`)
  console.log(response)
  emit('refresh')
}

</script>

<template>
  <div class="bg-white dark:bg-gray-800 rounded-lg shadow relative">
    <div class="absolute right-0 top-0 m-2">
      <u-button size="xs" @click="confirm(session)" color="white">
        <u-icon
          name="i-mdi-trash"
          class="w-4 h-4 text-red-400"
        />
      </u-button>
    </div>
    <div class="flex items-center justify-center py-4 border-gray-200 border-b dark:border-gray-600">
      <div :class="`device device-${type}`" />
    </div>
    <div class="p-4">
      <div>
        <div class="flex items-center justify-between text-gray-900 dark:text-gray-300 mb-2">
          {{ name }}
          <u-icon
            v-if="session.current"
            name="i-mdi-check-decagram"
            class="w-4 h-4 mr-1.5 text-green-400"
          />
        </div>
        <div class="flex items-center text-gray-500 text-sm mb-1">
          <u-icon
            name="i-mdi-application-outline"
            class="w-4 h-4 mr-1.5 text-gray-400"
          />
          {{ session.device.browser ? session.device.browser : source }}
        </div>
        <div class="flex items-center text-gray-500 text-sm mb-1">
          <u-icon
            name="i-mdi-map"
            class="w-4 h-4 mr-1.5 text-gray-400"
          />
          {{ session.location.city }}, {{ session.location.state }}, {{ session.location.country }}
        </div>
        <div
          v-if="session.source === 'google'"
          class="flex items-center text-gray-500 text-sm mb-1"
        >
          <u-icon
            name="i-mdi-google"
            class="w-4 h-4 mr-1.5"
          />
          Verified through Google
        </div>
        <div
          v-if="session.source === 'github'"
          class="flex items-center text-gray-500 text-sm mb-1"
        >
          <u-icon name="i-fa-brands:github" class="w-4 h-4 mr-1.5" />
          Verified through Github
        </div>
        <div
          v-if="session.source === 'email'"
          class="flex items-center text-gray-500 text-sm mb-1"
        >
          <u-icon name="i-mdi-email" class="w-4 h-4 mr-1.5" />
          Verified through E-mail
        </div>
        <div class="flex items-center text-gray-500 text-sm mb-1">
          <u-icon
            name="i-mdi-clock"
            class="w-4 h-4 mr-1.5 text-gray-400"
          />
          Created {{ dayjs(session.created_at).fromNow() }}
        </div>
        <div class="flex items-center text-gray-500 text-sm mb-1">
          <u-icon
            name="i-mdi-clock"
            class="w-4 h-4 mr-1.5 text-gray-400"
          />
          Last activity {{ dayjs(session.updated_at).fromNow() }}
        </div>
      </div>
    </div>
  </div>
</template>
