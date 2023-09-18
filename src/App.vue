<script setup lang="ts">
import { onLaunch, onShow, onHide } from '@dcloudio/uni-app'
import { type Ref, ref } from 'vue'

/* 允许程序后台运行，以持续获取GPS位置 */
const g_wakelock: Ref<any> = ref({})
// #ifdef APP-PLUS
const wakeLock = () => {
  const main = plus.android.runtimeMainActivity() as any
  const Context = plus.android.importClass('android.content.Context') as any
  const PowerManager = plus.android.importClass('android.os.PowerManager') as any
  const pm = main.getSystemService(Context.POWER_SERVICE)
  g_wakelock.value = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, 'ANY_NAME')
  g_wakelock.value.acquire()
}

/* 结束程序后台运行 */
const releaseWakeLock = () => {
  if (g_wakelock.value && g_wakelock.value.isHeld()) {
    g_wakelock.value.release()
    g_wakelock.value = null
  }
}

onLaunch(() => {
  console.log('App Launch')
  try {
    wakeLock()
  } catch (error) {
    console.log(error)
  }
})
onShow(() => {
  console.log('App Show')
  // releaseWakeLock()
})
onHide(() => {
  console.log('App Hide')
  // wakeLock()
})
// #endif
</script>
<style></style>
