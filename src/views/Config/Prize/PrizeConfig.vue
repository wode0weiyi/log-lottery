<script setup lang="ts">
import { Grip } from "lucide-vue-next";
import { nextTick, ref } from "vue";
import { VueDraggable } from "vue-draggable-plus";
import { useI18n } from "vue-i18n";
import EditSeparateDialog from "@/components/NumberSeparate/EditSeparateDialog.vue";
import PageHeader from "@/components/PageHeader/index.vue";
import ImageSync from "@/components/ImageSync/index.vue";
import GuaranteedWinnersDialog from "./GuaranteedWinnersDialog.vue";
import { usePrizeConfig } from "./usePrizeConfig";

const {
  addPrize,
  resetDefault,
  delAll,
  delItem,
  prizeList,
  currentPrize,
  selectedPrize,
  submitData,
  changePrizePerson,
  changePrizeStatus,
  selectPrize,
  localImageList,
  selectedGuaranteedPrize,
  openGuaranteedDialog,
  updateGuaranteedWinners,
} = usePrizeConfig();
const { t } = useI18n();

const guaranteedDialogRef = ref();

// 打开内定设置对话框
async function handleOpenGuaranteedDialog(item: any) {
  openGuaranteedDialog(item);
  await nextTick();
  guaranteedDialogRef.value?.showDialog();
}

// 更新内定名单
function handleUpdateGuaranteedWinners(guaranteedWinners: number[]) {
  if (selectedGuaranteedPrize.value) {
    updateGuaranteedWinners(
      selectedGuaranteedPrize.value.id,
      guaranteedWinners
    );
  }
}
</script>

<template>
  <div>
    <PageHeader :title="t('viewTitle.prizeManagement')">
      <template #buttons>
        <div class="flex w-full gap-3">
          <button class="btn btn-info btn-sm" @click="addPrize">
            {{ t("button.add") }}
          </button>
          <button class="btn btn-info btn-sm" @click="resetDefault">
            {{ t("button.resetDefault") }}
          </button>
          <button class="btn btn-error btn-sm" @click="delAll">
            {{ t("button.allDelete") }}
          </button>
        </div>
      </template>
      <template #alerts>
        <div role="alert" class="w-full my-4 alert alert-info">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            class="w-6 h-6 stroke-current shrink-0"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
            />
          </svg>
          <span>{{ t("dialog.tipResetPrize") }}</span>
        </div>
      </template>
    </PageHeader>
    <VueDraggable
      v-model="prizeList"
      :animation="150"
      handle=".handle"
      class="p-0 m-0"
    >
      <div
        v-for="item in prizeList"
        :key="item.id"
        class="flex flex-wrap items-start justify-between gap-x-4 gap-y-4 py-6 px-4 border-b border-slate-700/50 last:border-b-0 mx-auto max-max-7xl"
        :class="
          currentPrize.id === item.id
            ? 'bg-slate-800/50 rounded-xl border-dashed border-2 border-primary/30 my-2'
            : null
        "
      >
        <!-- 拖动图标 -->
        <div class="flex items-center self-center h-full pt-10 handle">
          <Grip
            class="w-5 h-5 cursor-move handle text-slate-500 hover:text-slate-300 transition-colors"
          />
        </div>

        <!-- 名称 - 缩小比例，自适应 -->
        <div
          class="flex flex-col items-center min-w-[120px] flex-[1.5] self-start"
        >
          <div
            class="text-[14px] text-slate-400 mb-2 uppercase tracking-wider font-medium text-center w-full h-10 flex items-center justify-center"
          >
            {{ t("table.prizeName") }}
          </div>
          <div class="flex flex-col items-center w-full">
            <input
              v-model="item.name"
              type="text"
              :placeholder="t('placeHolder.name')"
              class="w-full input-sm input input-bordered text-center focus:input-primary transition-all text-xs"
            />
            <div class="mt-1 w-full text-center">
              <span
                class="text-[10px] text-slate-500 truncate inline-block max-w-full italic"
                :title="item.name"
                >{{ item.name }}</span
              >
            </div>
          </div>
        </div>

        <!-- 抽奖人数 -->
        <div class="flex flex-col items-center flex-1 min-w-[80px] self-start">
          <div
            class="text-[14px] text-slate-400 mb-2 uppercase tracking-wider font-medium text-center w-full h-10 flex items-center justify-center"
          >
            {{ t("table.numberParticipants") }}
          </div>
          <input
            v-model="item.count"
            type="number"
            class="w-full max-w-[70px] input-sm input input-bordered text-center font-bold text-xs"
            @change="changePrizePerson(item)"
          />
        </div>

        <!-- 已抽取 -->
        <div class="flex flex-col items-center flex-1 min-w-[60px] self-start">
          <div
            class="text-[14px] text-slate-400 mb-2 uppercase tracking-wider font-medium text-center w-full h-10 flex items-center justify-center"
          >
            {{ t("table.isDone") }}
          </div>
          <div class="flex items-center justify-center pt-1">
            <input
              type="checkbox"
              :checked="item.isUsed"
              class="checkbox checkbox-secondary checkbox-sm border-2"
              @change="changePrizeStatus(item)"
            />
          </div>
        </div>

        <!-- 图片 -->
        <div class="flex flex-col items-center flex-1 min-w-[80px] self-start">
          <div
            class="text-[14px] text-slate-400 mb-2 uppercase tracking-wider font-medium text-center w-full h-10 flex items-center justify-center"
          >
            {{ t("table.image") }}
          </div>
          <div
            class="relative group cursor-pointer"
            @click="(e) => (e.target as any).closest('.relative').querySelector('select').click()"
          >
            <div
              class="w-10 h-10 rounded-lg border-2 border-slate-600 overflow-hidden bg-slate-700 flex items-center justify-center hover:border-primary transition-all shadow-md"
            >
              <ImageSync
                v-if="item.picture.url"
                :img-item="item.picture"
                class="w-full h-full object-cover"
              />
              <div v-else class="text-lg">🖼️</div>
            </div>
            <select
              v-model="item.picture"
              class="absolute inset-0 opacity-0 cursor-pointer w-full h-full"
            >
              <option :value="{ id: '', name: '', url: '' }">
                ❌ {{ t("button.reset") }}
              </option>
              <option
                v-for="picItem in localImageList"
                :key="picItem.id"
                :value="picItem"
              >
                {{ picItem.name }}
              </option>
            </select>
          </div>
        </div>

        <!-- 单次抽取个数 -->
        <div
          v-if="item.separateCount"
          class="flex flex-col items-center flex-1 min-w-[100px] self-start"
        >
          <div
            class="text-[14px] text-slate-400 mb-2 uppercase tracking-wider font-medium text-center w-full h-10 flex items-center justify-center"
          >
            {{ t("table.onceNumber") }}
          </div>
          <div
            class="flex flex-wrap justify-center gap-1 cursor-pointer hover:scale-105 transition-transform py-1 px-1 rounded bg-slate-700/30 w-full min-h-[28px] items-center"
            @click="selectPrize(item)"
          >
            <template v-if="item.separateCount.countList.length">
              <span
                v-for="se in item.separateCount.countList"
                :key="se.id"
                class="badge badge-secondary badge-xs scale-90 origin-center font-bold"
                :class="
                  se.isUsedCount >= se.count ? 'badge-ghost opacity-50' : ''
                "
              >
                {{ se.count }}
              </span>
            </template>
            <span v-else class="text-xs font-bold text-primary">{{
              item.count
            }}</span>
          </div>
        </div>

        <!-- 内定名额 -->
        <div class="flex flex-col items-center flex-1 min-w-[100px] self-start">
          <div
            class="text-[14px] text-slate-400 mb-2 uppercase tracking-wider font-medium text-center w-full h-10 flex items-center justify-center"
          >
            {{ t("table.guaranteedWinners") }}
          </div>
          <div class="relative inline-block mt-0.5">
            <button
              class="btn btn-secondary btn-xs whitespace-nowrap px-3 h-7 min-h-[28px]"
              @click="handleOpenGuaranteedDialog(item)"
            >
              {{ t("button.setGuaranteed") }}
            </button>
            <div
              v-if="item.guaranteedWinners && item.guaranteedWinners.length > 0"
              class="tooltip absolute -top-1.5 -right-1.5 z-10"
              :data-tip="
                t('tooltip.guaranteedCount', {
                  count: item.guaranteedWinners.length,
                })
              "
            >
              <div
                class="badge badge-primary badge-xs scale-90 border-slate-900 font-bold"
              >
                {{ item.guaranteedWinners.length }}
              </div>
            </div>
          </div>
        </div>

        <!-- 操作 - 保持固定宽度 -->
        <div class="flex flex-col items-center w-16 self-start">
          <div
            class="text-[14px] text-slate-400 mb-2 uppercase tracking-wider font-medium text-center w-full h-10 flex items-center justify-center"
          >
            {{ t("table.operation") }}
          </div>
          <button
            class="btn btn-error btn-xs btn-square h-7 min-h-[28px] w-7"
            @click="delItem(item)"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-3.5 w-3.5"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
              />
            </svg>
          </button>
        </div>
      </div>
    </VueDraggable>
    <EditSeparateDialog
      :total-number="selectedPrize?.count"
      :separated-number="selectedPrize?.separateCount.countList"
    />
    <GuaranteedWinnersDialog
      ref="guaranteedDialogRef"
      :prize="selectedGuaranteedPrize"
      @update:guaranteed-winners="handleUpdateGuaranteedWinners"
    />
  </div>
</template>

<style lang="scss" scoped></style>
