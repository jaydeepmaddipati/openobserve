<template>
  <div
    class="functions-toolbar tw-pb-1.5 tw-w-full tw-flex tw-justify-between tw-items-center"
  >
    <div class="tw-flex tw-items-center">
      <div class="tw-mr-2 add-function-back-btn">
        <div
          data-test="add-function-back-btn"
          class="flex justify-center items-center cursor-pointer"
          style="
            border: 1.5px solid;
            border-radius: 50%;
            width: 22px;
            height: 22px;
          "
          title="Go Back"
          @click="redirectToFunctions"
        >
          <q-icon name="arrow_back_ios_new" size="14px" />
        </div>
      </div>
      <div class="tw-text-lg tw-w-full add-function-title q-mr-md">
        Add Function
      </div>
      <q-form ref="addFunctionForm" class="o2-input">
        <div class="tw-flex tw-items-center">
          <q-input
            data-test="add-function-name-input"
            v-model.trim="functionName"
            :label="t('function.name')"
            color="input-border"
            bg-color="input-bg"
            class="q-pa-none tw-w-full"
            stack-label
            outlined
            filled
            dense
            v-bind:readonly="disableName"
            v-bind:disable="disableName"
            :rules="[
              (val: any) => !!val || 'Field is required!',
              isValidMethodName,
            ]"
            no-error-icon
            tabindex="0"
            style="min-width: 300px"
            @update:model-value="onUpdate"
            @blur="onUpdate"
          />
          <q-icon
            :key="functionName"
            v-if="isValidMethodName() !== true && showInputError"
            :name="outlinedInfo"
            size="20px"
            class="q-ml-xs cursor-pointer"
            :class="store.state.theme === 'dark' ? 'text-red-5' : 'text-red-7'"
          >
            <q-tooltip
              anchor="center right"
              self="center left"
              max-width="300px"
              :offset="[2, 0]"
              class="tw-text-[12px]"
            >
              {{ isValidMethodName() }}
            </q-tooltip>
          </q-icon>
        </div>
      </q-form>
    </div>
    <div class="add-function-actions flex justify-center">
      <q-btn
            v-if="config.isEnterprise == 'true' && !isAddFunctionComponent && store.state.zoConfig.ai_enabled"
            :ripple="false"
            @click="emit('open:chat',!store.state.isAiChatEnabled)"
            data-test="menu-link-ai-item"
            no-caps
            :borderless="true"
            flat
            dense
            class="o2-button ai-hover-btn q-px-sm q-py-sm"
            :class="store.state.isAiChatEnabled ? 'ai-btn-active' : ''"
            style="border-radius: 100%;"
            @mouseenter="isHovered = true"
            @mouseleave="isHovered = false"

          >
            <div class="row items-center no-wrap tw-gap-2  ">
              <img  :src="getBtnLogo" class="header-icon ai-icon" />
            </div>
          </q-btn>
      <q-btn
        data-test="add-function-fullscreen-btn"
        v-close-popup="true"
        class="text-bold tw-border-primary add-function-fullscreen-btn"
        :label="t('common.fullscreen')"
        :text-color="store.state.theme === 'dark' ? 'grey-1' : 'primary'"
        padding="sm"
        no-caps
        icon="fullscreen"
        @click="handleFullScreen"
      />
      <q-btn
        data-test="add-function-test-btn"
        :label="t('function.testFunction')"
        class="text-bold no-border tw-ml-[12px] add-function-test-btn"
        color="primary"
        padding="sm md"
        no-caps
        @click="emit('test')"
      />
      <q-btn
        data-test="add-function-save-btn"
        :label="t('function.save')"
        class="text-bold no-border tw-ml-[12px] add-function-save-btn"
        color="secondary"
        padding="sm md"
        type="submit"
        no-caps
        @click="onSave"
      />
      <q-btn
        data-test="add-function-cancel-btn"
        class="cancel-btn text-bold tw-ml-[12px] tw-border-3 tw-border-red-600 add-function-cancel-btn"
        :label="t('function.cancel')"
        text-color="negative"
        padding="sm md"
        no-caps
        @click="emit('cancel')"
      />
    </div>
  </div>
</template>
<script setup lang="ts">
import {
  defineComponent,
  ref,
  onMounted,
  computed,
  watch,
  defineEmits,
} from "vue";
import { useI18n } from "vue-i18n";
import { useQuasar } from "quasar";
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import { outlinedInfo } from "@quasar/extras/material-icons-outlined";
import config from "../../aws-exports";
import { getImageURL } from "@/utils/zincutils";
const { t } = useI18n();

const q = useQuasar();

const router = useRouter();

const store = useStore();

const props = defineProps({
  name: {
    type: String,
    required: true,
  },
  disableName: {
    type: Boolean,
    default: false,
  },
  isAddFunctionComponent: {
    type: Boolean,
    default: true,
  },
});

const emit = defineEmits(["test", "save", "update:name", "back", "cancel", "open:chat"]);

const addFunctionForm = ref(null);

const showInputError = ref(false);

const isHovered = ref(false);
const isValidMethodName = () => {
  if (!functionName.value) return "Field is required!";
  const methodPattern = /^[A-Z_][A-Z0-9_]*$/i;
  return (
    methodPattern.test(functionName.value) ||
    "Invalid method name. Must start with a letter or underscore. Use only letters, numbers, and underscores."
  );
};

const onUpdate = () => {
  showInputError.value = true;
};

const functionName = computed({
  get: () => props.name,
  set: (value) => emit("update:name", value),
});

const isAddFunctionComponent = computed(() => router.currentRoute.value.path.includes('functions'))
const handleFullScreen = () => {
  q.fullscreen.toggle();
};

const redirectToFunctions = () => {
  emit("back");
};

const onSave = () => {
  showInputError.value = true;
  emit("save");
};

const getBtnLogo = computed(() => {
      if (isHovered.value || store.state.isAiChatEnabled) {
        return getImageURL('images/common/ai_icon_dark.svg')
      }

      return store.state.theme === 'dark'
        ? getImageURL('images/common/ai_icon_dark.svg')
        : getImageURL('images/common/ai_icon.svg')
    })

defineExpose({ addFunctionForm });
</script>
<style scoped lang="scss">
.functions-toolbar {
  :deep(.q-field__bottom) {
    display: none;
  }

  .add-function-actions {
    :deep(.q-btn) {
      padding-top: 4px !important;
      padding-bottom: 4px !important;
      font-size: 13px;
    }
    :deep(.q-btn .q-icon) {
      margin-right: 2px;
    }

    :deep(.block) {
      font-weight: lighter;
    }

    :deep(.cancel-btn)::before {
      border: 1px solid var(--q-negative) !important;
    }
  }
}
</style>
