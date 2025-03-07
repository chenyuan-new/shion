<script setup lang="ts">
const timeStore = useTimerStore()
const historyStore = useHistoryStore()
const updateStore = useUpdateStore()

const { time, running, text } = storeToRefs(timeStore)
const { requesting, progress } = storeToRefs(historyStore)
const { needUpdate } = storeToRefs(updateStore)

const { start: startUpdate } = updateStore

const route = useRoute()
const { xs, sm } = useTailwindBreakpoints()
const router = useRouter()
const { refresh, registered, loading } = usePageRefresh()
const { dragged, isGrid, toggleDrag } = layoutInject()

const help = ref(false)

const timerText = computed(() => route.fullPath == '/timer' ? text.value : `#${text.value}# ${time.value}`)

const page = computed(() => {
  const path = route.fullPath
  if (path == '/')
    return 'chart'
  else if (path == '/timeline')
    return 'timeline'
  else if (path == '/timeblock')
    return 'timeblock'
  else if (path == '/collection/plan' || path == '/collection/label' || path == '/timer')
    return 'manual'
  else if (path == '/collection/monitor')
    return 'automatic'
  else if (path == '/collection/dimension')
    return 'dimension'
})

function navigateTimer() {
  router.push('/timer')
}

async function update() {
  const open = await startUpdate(true)
  open?.()
}

function openHelpDialog() {
  help.value = true
}
</script>

<template>
  <div v-show="xs" id="status-bar-xs" px-4 flex h-full items-center relative />
  <div v-show="sm" px-4 flex items-center space-x-1>
    <div flex-1 />
    <status-bar-button
      :tooltip="$t(`statusBar.help.tooltip.${page}`)" :text="$t('statusBar.help.text')"
      icon="i-mdi:help" @click="openHelpDialog"
    />
    <status-bar-button
      v-if="needUpdate" :tooltip="$t('statusBar.update.tooltip')" :text="$t('statusBar.update.text')"
      icon="i-mdi:new-box" color="info" variant="flat" @click="update"
    />
    <status-bar-button v-if="requesting" :tooltip="$t('statusBar.timeline.history.tooltip')">
      <div flex items-center space-x-1>
        <div>{{ $t('statusBar.timeline.history.text') }}</div>
        <div class="w-[60px]">
          <v-progress-linear color="primary" :model-value="progress" rounded height="8" />
        </div>
      </div>
    </status-bar-button>
    <div id="status-bar-sm" flex items-center />
    <status-bar-button
      v-if="running" :tooltip="$t('statusBar.timer')" :text="timerText" icon="i-mdi:timer-outline"
      @click="navigateTimer"
    />
    <status-bar-button
      v-if="isGrid"
      :tooltip="dragged ? $t('statusBar.drag.tooltip.lock') : $t('statusBar.drag.tooltip.unlock')"
      :text="$t('statusBar.drag.text')" :icon="dragged ? 'i-mdi:lock-open-variant-outline' : 'i-mdi:lock-outline'"
      @click="() => toggleDrag()"
    />
    <status-bar-button
      v-if="registered" :tooltip="$t('statusBar.refresh.tooltip')" :text="$t('statusBar.refresh.text')"
      :loading="loading" icon="i-mdi:refresh" loader-icon="i-mdi:refresh animate-spin animate-duration-300"
      @click="() => refresh()"
    />
  </div>
  <help-dialog v-model:visible="help" :page="page" />
</template>
