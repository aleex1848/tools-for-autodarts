<template>
  <div
    v-if="config"
    class="adt-container transition-transform hover:-translate-y-0.5"
  >
    <div class="relative z-10 flex h-full flex-col justify-between space-y-4">
      <div class="flex items-start justify-between">
        <div>
          <h3 class="mb-1 font-bold uppercase">
            Webhooks
          </h3>
          <p class="max-w-[18rem] text-sm text-white/70">
            Sende Spielinformationen an einen externen Endpoint. Wähle aus, ob du Match-Details
            oder jede einzelne Aufnahme erhalten möchtest.
          </p>
        </div>
        <div class="flex">
          <div @click="$emit('toggle', 'webhooks')" class="absolute inset-y-0 left-12 right-0 cursor-pointer" />
          <AppToggle
            @update:model-value="toggleFeature"
            v-model="config.webhooks.enabled"
          />
        </div>
      </div>

      <div v-if="config.webhooks.enabled" class="space-y-4 border-t border-white/10 pt-4 text-sm text-white/70">
        <AppInput
          v-model="config.webhooks.url"
          label="Webhook URL"
          placeholder="https://example.com/webhook"
          helper-text="HTTPS-Endpoint, der POST-JSON empfängt."
        />
        <AppInput
          v-model="config.webhooks.token"
          label="Optionale Webhook-Token"
          placeholder="z. B. Bearer-Token oder beliebiger Geheimtext"
          helper-text="Wird als Authorization: Bearer <Token> gesendet."
        />
        <div class="space-y-2">
          <p class="text-xs uppercase text-white/60">
            Daten auswählen
          </p>
          <div
            v-for="option in payloadOptions"
            :key="option.key"
            class="flex flex-col rounded border border-white/10 bg-white/5 p-3 text-white/70"
          >
            <div class="flex items-center justify-between">
              <div>
                <p class="font-semibold text-white">
                  {{ option.label }}
                </p>
                <p class="text-xs text-white/60">
                  {{ option.description }}
                </p>
              </div>
              <AppToggle v-model="config.webhooks.payloadTypes[option.key]" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import AppInput from "../AppInput.vue";
import AppToggle from "../AppToggle.vue";
import type { IConfig } from "@/utils/storage";
import { AutodartsToolsConfig } from "@/utils/storage";

type PayloadKey = keyof IConfig["webhooks"]["payloadTypes"];

const emit = defineEmits([ "toggle", "settingChange" ]);
const config = ref<IConfig>();

const payloadOptions: Array<{
  key: PayloadKey;
  label: string;
  description: string;
}> = [
  {
    key: "match",
    label: "Alle Spieldaten",
    description: "Match-Zustand inklusive Scores, Legs und Spielerlisten bei jeder Änderung.",
  },
  {
    key: "throws",
    label: "Jeder Pfeil",
    description: "Ein JSON-Payload pro Dartwurf mit Segment, Koordinaten und Spieler.",
  },
];

async function toggleFeature() {
  if (!config.value) return;

  const wasEnabled = config.value.webhooks.enabled;
  config.value.webhooks.enabled = !wasEnabled;

  if (!wasEnabled) {
    await nextTick();
    emit("toggle", "webhooks");
  }
}

onMounted(async () => {
  config.value = await AutodartsToolsConfig.getValue();
});

watch(config, async (_, oldValue) => {
  if (!oldValue) return;

  await AutodartsToolsConfig.setValue(toRaw(config.value!));
  emit("settingChange");
  console.log("Webhooks setting changed");
}, { deep: true });
</script>

