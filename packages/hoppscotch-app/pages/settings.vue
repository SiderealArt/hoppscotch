<template>
  <div>
    <div class="divide-y divide-dividerLight container space-y-8">
      <div class="md:grid md:gap-4 md:grid-cols-3">
        <div class="p-8 md:col-span-1">
          <h3 class="heading">
            {{ $t("settings.theme") }}
          </h3>
          <p class="mt-1 text-secondaryLight">
            {{ $t("settings.theme_description") }}
          </p>
        </div>
        <div class="space-y-8 p-8 md:col-span-2">
          <section>
            <h4 class="font-semibold text-secondaryDark">
              {{ $t("settings.background") }}
            </h4>
            <div class="mt-1 text-secondaryLight">
              <ColorScheme placeholder="..." tag="span">
                {{ $t(getColorModeName($colorMode.preference)) }}
                <span v-if="$colorMode.preference === 'system'">
                  ({{ $t(getColorModeName($colorMode.value)) }})
                </span>
              </ColorScheme>
            </div>
            <div class="mt-4">
              <SmartColorModePicker />
            </div>
          </section>
          <section>
            <h4 class="font-semibold text-secondaryDark">
              {{ $t("settings.accent_color") }}
            </h4>
            <div class="mt-1 text-secondaryLight">
              {{ active.charAt(0).toUpperCase() + active.slice(1) }}
            </div>
            <div class="mt-4">
              <SmartAccentModePicker />
            </div>
          </section>
          <section>
            <h4 class="font-semibold text-secondaryDark">
              {{ $t("settings.font_size") }}
            </h4>
            <div class="mt-4">
              <SmartFontSizePicker />
            </div>
          </section>
          <section>
            <h4 class="font-semibold text-secondaryDark">
              {{ $t("settings.language") }}
            </h4>
            <div class="mt-4">
              <SmartChangeLanguage />
            </div>
          </section>
          <section>
            <h4 class="font-semibold text-secondaryDark">
              {{ $t("settings.experiments") }}
            </h4>
            <div class="mt-1 text-secondaryLight">
              {{ $t("settings.experiments_notice") }}
              <SmartLink
                class="link"
                to="https://github.com/hoppscotch/hoppscotch/issues/new/choose"
                blank
              >
                {{ $t("app.contact_us") }} </SmartLink
              >.
            </div>
            <div class="space-y-4 py-4">
              <div class="flex items-center">
                <SmartToggle :on="TELEMETRY_ENABLED" @change="showConfirmModal">
                  {{ $t("settings.telemetry") }}
                </SmartToggle>
              </div>
              <div class="flex items-center">
                <SmartToggle
                  :on="EXPAND_NAVIGATION"
                  @change="toggleSetting('EXPAND_NAVIGATION')"
                >
                  {{ $t("settings.expand_navigation") }}
                </SmartToggle>
              </div>
              <div class="flex items-center">
                <SmartToggle
                  :on="SIDEBAR_ON_LEFT"
                  @change="toggleSetting('SIDEBAR_ON_LEFT')"
                >
                  {{ $t("settings.sidebar_on_left") }}
                </SmartToggle>
              </div>
              <div class="flex items-center">
                <SmartToggle :on="ZEN_MODE" @change="toggleSetting('ZEN_MODE')">
                  {{ $t("layout.zen_mode") }}
                </SmartToggle>
              </div>
            </div>
          </section>
        </div>
      </div>

      <div class="md:grid md:gap-4 md:grid-cols-3">
        <div class="p-8 md:col-span-1">
          <h3 class="heading">
            {{ $t("settings.interceptor") }}
          </h3>
          <p class="mt-1 text-secondaryLight">
            {{ $t("settings.interceptor_description") }}
          </p>
        </div>
        <div class="space-y-8 p-8 md:col-span-2">
          <section>
            <h4 class="font-semibold text-secondaryDark">
              {{ $t("settings.extensions") }}
            </h4>
            <div class="mt-1 text-secondaryLight">
              <span v-if="extensionVersion != null">
                {{
                  `${$t("settings.extension_version")}: v${
                    extensionVersion.major
                  }.${extensionVersion.minor}`
                }}
              </span>
              <span v-else>
                {{ $t("settings.extension_version") }}:
                {{ $t("settings.extension_ver_not_reported") }}
              </span>
            </div>
            <div class="flex flex-col space-y-2 py-4">
              <span>
                <SmartItem
                  to="https://addons.mozilla.org/en-US/firefox/addon/hoppscotch"
                  blank
                  svg="brands/firefox"
                  label="Firefox"
                  :info-icon="hasFirefoxExtInstalled ? 'check_circle' : ''"
                  :active-info-icon="hasFirefoxExtInstalled"
                  outline
                />
              </span>
              <span>
                <SmartItem
                  to="https://chrome.google.com/webstore/detail/hoppscotch-browser-extens/amknoiejhlmhancpahfcfcfhllgkpbld"
                  blank
                  svg="brands/chrome"
                  label="Chrome"
                  :info-icon="hasChromeExtInstalled ? 'check_circle' : ''"
                  :active-info-icon="hasChromeExtInstalled"
                  outline
                />
              </span>
            </div>
            <div class="space-y-4 py-4">
              <div class="flex items-center">
                <SmartToggle
                  :on="EXTENSIONS_ENABLED"
                  @change="toggleSetting('EXTENSIONS_ENABLED')"
                >
                  {{ $t("settings.extensions_use_toggle") }}
                </SmartToggle>
              </div>
            </div>
          </section>
          <section>
            <h4 class="font-semibold text-secondaryDark">
              {{ $t("settings.proxy") }}
            </h4>
            <div class="mt-1 text-secondaryLight">
              {{
                `${$t("settings.official_proxy_hosting")} ${$t(
                  "settings.read_the"
                )}`
              }}
              <SmartLink
                class="link"
                to="https://docs.hoppscotch.io/privacy"
                blank
              >
                {{ $t("app.proxy_privacy_policy") }} </SmartLink
              >.
            </div>
            <div class="space-y-4 py-4">
              <div class="flex items-center">
                <SmartToggle
                  :on="PROXY_ENABLED"
                  @change="toggleSetting('PROXY_ENABLED')"
                >
                  {{ $t("settings.proxy_use_toggle") }}
                </SmartToggle>
              </div>
            </div>
            <div class="flex space-x-2 py-4 items-center">
              <div class="flex flex-1 flex-col relative">
                <input
                  id="url"
                  v-model="PROXY_URL"
                  class="input floating-input"
                  placeholder=" "
                  type="url"
                  autocomplete="off"
                  :disabled="!PROXY_ENABLED"
                />
                <label for="url">
                  {{ $t("settings.proxy_url") }}
                </label>
              </div>
              <ButtonSecondary
                v-tippy="{ theme: 'tooltip' }"
                :title="$t('settings.reset_default')"
                :svg="clearIcon"
                outline
                class="rounded"
                @click.native="resetProxy"
              />
            </div>
          </section>
        </div>
      </div>
    </div>
    <SmartConfirmModal
      :show="confirmRemove"
      :title="`${$t('confirm.remove_telemetry')} ${$t(
        'settings.telemetry_helps_us'
      )}`"
      @hide-modal="confirmRemove = false"
      @resolve="
        () => {
          toggleSetting('TELEMETRY_ENABLED')
          confirmRemove = false
        }
      "
    />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "@nuxtjs/composition-api"
import {
  hasExtensionInstalled,
  hasChromeExtensionInstalled,
  hasFirefoxExtensionInstalled,
} from "~/helpers/strategies/ExtensionStrategy"
import {
  applySetting,
  toggleSetting,
  defaultSettings,
  useSetting,
} from "~/newstore/settings"
import type { KeysMatching } from "~/types/ts-utils"
import { getLocalConfig } from "~/newstore/localpersistence"

type SettingsType = typeof defaultSettings

export default defineComponent({
  setup() {
    return {
      PROXY_ENABLED: useSetting("PROXY_ENABLED"),
      PROXY_URL: useSetting("PROXY_URL"),
      EXTENSIONS_ENABLED: useSetting("EXTENSIONS_ENABLED"),
      TELEMETRY_ENABLED: useSetting("TELEMETRY_ENABLED"),
      EXPAND_NAVIGATION: useSetting("EXPAND_NAVIGATION"),
      SIDEBAR_ON_LEFT: useSetting("SIDEBAR_ON_LEFT"),
      ZEN_MODE: useSetting("ZEN_MODE"),
    }
  },
  data() {
    return {
      extensionVersion: hasExtensionInstalled()
        ? window.__POSTWOMAN_EXTENSION_HOOK__.getVersion()
        : null,

      hasChromeExtInstalled: hasChromeExtensionInstalled(),
      hasFirefoxExtInstalled: hasFirefoxExtensionInstalled(),

      clearIcon: "rotate-ccw",

      active: getLocalConfig("THEME_COLOR") || "blue",
      confirmRemove: false,
    }
  },
  head() {
    return {
      title: `${this.$t("navigation.settings")} • Hoppscotch`,
    }
  },
  computed: {
    proxySettings(): { url: string } {
      return {
        url: this.PROXY_URL,
      }
    },
  },
  watch: {
    ZEN_MODE(ZEN_MODE) {
      this.applySetting("EXPAND_NAVIGATION", !ZEN_MODE)
    },
    proxySettings: {
      deep: true,
      handler({ url }) {
        this.applySetting("PROXY_URL", url)
      },
    },
  },
  methods: {
    showConfirmModal() {
      if (this.TELEMETRY_ENABLED) this.confirmRemove = true
      else toggleSetting("TELEMETRY_ENABLED")
    },
    applySetting<K extends keyof SettingsType>(key: K, value: SettingsType[K]) {
      applySetting(key, value)
    },
    toggleSetting<K extends KeysMatching<SettingsType, boolean>>(key: K) {
      if (key === "EXTENSIONS_ENABLED" && this.PROXY_ENABLED) {
        toggleSetting("PROXY_ENABLED")
      }
      if (key === "PROXY_ENABLED" && this.EXTENSIONS_ENABLED) {
        toggleSetting("EXTENSIONS_ENABLED")
      }
      toggleSetting(key)
    },
    toggleSettings<K extends KeysMatching<SettingsType, boolean>>(
      name: K,
      value: SettingsType[K]
    ) {
      this.applySetting(name, value)
    },
    resetProxy() {
      applySetting("PROXY_URL", `https://proxy.hoppscotch.io/`)
      this.clearIcon = "check"
      this.$toast.success(`${this.$t("state.cleared")}`)
      setTimeout(() => (this.clearIcon = "rotate-ccw"), 1000)
    },
    getColorModeName(colorMode: string) {
      switch (colorMode) {
        case "system":
          return "settings.system_mode"
        case "light":
          return "settings.light_mode"
        case "dark":
          return "settings.dark_mode"
        case "black":
          return "settings.black_mode"
        default:
          return "settings.system_mode"
      }
    },
  },
})
</script>
