<script lang="ts" setup>
import { useSiteConfig, useValaxyConfig, useValaxyDark } from 'valaxy'
import pkg from 'valaxy/package.json'
import { capitalize, computed, ref, onMounted, onUnmounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useThemeConfig } from '../composables'
import NoticeBar from './NoticeBar.vue'
// 自定义链接
const customLinks = [
  {
    name: '开往',
    link: 'https://www.travellings.cn/go-by-clouds.html',
    icon: 'https://www.travellings.cn/assets/logo.gif',
  },
]

// 旅行者一号距离地球的信息
const voyagerDistance = ref<string>('正在加载...')

// 模拟获取旅行者一号距离
const getVoyagerDistance = (): void => {
  const now = new Date()
  const start = new Date('01/17/2024 00:00:00') // 旅行者1号开始计算的时间
  const timeDifferenceInSeconds = (now.getTime() - start.getTime()) / 1000 // 转换为秒
  const distanceInKilometers = Math.trunc(23400000000 + timeDifferenceInSeconds * 17) // 距离=秒数*速度
  const astronomicalUnits = (distanceInKilometers / 149600000).toFixed(6) // 天文单位

  voyagerDistance.value = `旅行者 1 号当前距离地球 ${distanceInKilometers.toLocaleString()} 千米，约为 ${astronomicalUnits} 个天文单位 🚀`
}

// 定时器
let intervalId: NodeJS.Timeout | null = null

onMounted(() => {
  getVoyagerDistance() // 初始化数据
  intervalId = setInterval(getVoyagerDistance, 1000) // 每秒更新一次
})

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId) // 组件卸载时清除定时器
  }
})

// background-image: linear-gradient(120deg, #a1c4fd 0%, #c2e9fb 100%);
const { isDark } = useValaxyDark()
const gradientStyles = computed(() => {
  if (isDark.value) {
    return {
      '--gradient-from': '0 0 0',
      '--gradient-to': '0 0 0',
    }
  }
  return {
    '--gradient-from': '161 196 253',
    '--gradient-to': '194 233 251',
  }
})

const { t } = useI18n()
const config = useValaxyConfig()
const siteConfig = useSiteConfig()
const themeConfig = useThemeConfig()
const year = new Date().getFullYear()

const isThisYear = computed(() => {
  return year === themeConfig.value.footer.since
})

const poweredHtml = computed(() => t('footer.powered', [`<a href="${pkg.repository.url}" target="_blank" rel="noopener">Valaxy</a> <span class="op-60">v${pkg.version}</span>`]))
const footerIcon = computed(() => themeConfig.value.footer.icon || {
  url: pkg.repository.url,
  name: 'i-ri-cloud-line',
  title: pkg.name,
})
</script>

<template>

  <footer
    flex="~ col"
    class="relative yun-footer va-footer px-4 py-4 pt-0 text-$va-c-text-light w-full mt-14"
    bg="white dark:$va-c-bg-soft"
    text="center sm"
  >
    <YunCloud v-if="themeConfig.footer.cloud?.enable" class="absolute top--10 left-0 right-0" />

    <!-- 自定义链接 -->
    <div class="custom-links flex justify-center items-center gap-2" p="1">
      <template v-for="(link, index) in customLinks" :key="index">
        <template v-if="link.icon">
          <a :href="link.link" target="_blank" rel="noopener">
            <img :src="link.icon" :alt="link.name" />
          </a>
        </template>
        <template v-else>
          <a :href="link.link" target="_blank" rel="noopener">{{ link.name }}</a>
        </template>
        <span v-if="index < customLinks.length - 1"> |</span>
      </template>
    </div>

    <!-- 萌ICP备案信息 -->
    <div class="beian" m="y-2">
      <a href="https://icp.gov.moe/?keyword=[萌号]" target="_blank" rel="noopener">萌ICP备号</a>
    </div>

    <!-- 旅行者一号距离地球的信息 -->
    <div class="voyager-distance" m="y-2">
      {{ voyagerDistance }}
    </div>

    <div v-if="themeConfig.footer.beian?.enable && themeConfig.footer.beian.icp" class="beian" m="y-2">
      <a href="https://beian.miit.gov.cn/" target="_blank" rel="noopener">
        {{ themeConfig.footer.beian.icp }}
      </a>
    </div>

    <div class="copyright flex justify-center items-center gap-2" p="1">
      <span>
        &copy;
        <template v-if="!isThisYear">
          {{ themeConfig.footer.since }} -
        </template>
        {{ year }}
      </span>

      <a
        v-if="themeConfig.footer.icon?.enable"
        class="inline-flex"
        :class="themeConfig.footer.icon.animated ? 'animate-pulse' : ''"
        :href="footerIcon.url"
        target="_blank"
        :title="footerIcon.title"
      >
        <div :class="footerIcon.name" />
      </a>
      <span>{{ siteConfig.author.name }}</span>
    </div>

    <div v-if="themeConfig.footer.powered" class="powered" m="2">
      <span v-html="poweredHtml" />
      <span mx-1>|</span>
      <span>
        <span>{{ t('footer.theme') }}</span>
        <span mx-1>-</span>
        <a :href="themeConfig.pkg.repository.url" :title="themeConfig.pkg.name" target="_blank">{{ capitalize(config.theme) }}</a>
        <span class="ml-1 op-60">v{{ themeConfig.pkg.version }}</span>
      </span>
    </div>

    <slot />

    <div class="yun-footer-gradient" :style="gradientStyles" />
  </footer>
  <NoticeBar />
</template>

<style lang="scss">
.yun-footer {
  letter-spacing: 0.05rem;
  line-height: 1.8;
}

.yun-footer-gradient {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  width: 100%;
  height: 300px;
  z-index: 999;
  pointer-events: none;
  background: linear-gradient(to right,rgb(var(--gradient-from) / 0.2) 0,rgb(var(--gradient-to) / .2) 100%);
  mask-image: linear-gradient(#fff0,#000 70%);
  animation: fade-in 2s;
}

.custom-links img {
  height: 20px; /* 根据需要调整图标大小 */
  vertical-align: middle;
}

.voyager-distance {
  font-size: 14px;
  color: #666;
  text-align: center;
}
</style>