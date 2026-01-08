<script setup lang="ts">
import type { IPersonConfig, IPrizeConfig } from "@/types/storeType";
import { computed, ref, watch } from "vue";
import { useI18n } from "vue-i18n";
import useStore from "@/store";

const props = defineProps<{
  prize: IPrizeConfig | null;
}>();

const emit = defineEmits<{
  (e: "update:guaranteedWinners", value: number[]): void;
}>();

const { t } = useI18n();
const personConfig = useStore().personConfig;
const allPersonList = computed(() => personConfig.getAllPersonList);

const dialogRef = ref<HTMLDialogElement>();
const selectedPersonIds = ref<number[]>([]);
const searchKeyword = ref("");

// 过滤后的人员列表
const filteredPersonList = computed(() => {
  if (!searchKeyword.value) {
    return allPersonList.value;
  }
  const keyword = searchKeyword.value.toLowerCase();
  return allPersonList.value.filter(
    (person) =>
      person.name.toLowerCase().includes(keyword) ||
      person.uid.toLowerCase().includes(keyword) ||
      person.department.toLowerCase().includes(keyword)
  );
});

// 检查某人是否被选中
function isPersonSelected(personId: number) {
  return selectedPersonIds.value.includes(personId);
}

// 切换人员选择状态
function togglePerson(personId: number) {
  const index = selectedPersonIds.value.indexOf(personId);
  if (index > -1) {
    selectedPersonIds.value.splice(index, 1);
  } else {
    selectedPersonIds.value.push(personId);
  }
}

// 移除人员
function removePerson(personId: number) {
  const index = selectedPersonIds.value.indexOf(personId);
  if (index > -1) {
    selectedPersonIds.value.splice(index, 1);
  }
}

// 确认选择
function confirmSelection() {
  emit("update:guaranteedWinners", [...selectedPersonIds.value]);
  dialogRef.value?.close();
}

// 取消
function cancel() {
  dialogRef.value?.close();
}

// 打开对话框
function showDialog() {
  if (props.prize) {
    selectedPersonIds.value = [...(props.prize.guaranteedWinners || [])];
    dialogRef.value?.showModal();
  }
}

// 获取人员信息
function getPersonById(personId: number): IPersonConfig | undefined {
  return allPersonList.value.find((p) => p.id === personId);
}

defineExpose({
  showDialog,
});
</script>

<template>
  <dialog ref="dialogRef" class="modal">
    <div class="modal-box max-w-4xl">
      <h3 class="text-lg font-bold mb-4">
        {{ t("dialog.titleSetGuaranteed") }}
      </h3>

      <!-- 提示信息 -->
      <div role="alert" class="alert alert-info mb-4">
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
        <span>{{ t("dialog.guaranteedWinnersTip") }}</span>
      </div>

      <!-- 已选择的人员列表 -->
      <div v-if="selectedPersonIds.length > 0" class="mb-4">
        <h4 class="font-semibold mb-2">
          {{ t("button.addGuaranteed") }} ({{ selectedPersonIds.length }})
        </h4>
        <div class="flex flex-wrap gap-2">
          <div
            v-for="personId in selectedPersonIds"
            :key="personId"
            class="badge badge-primary badge-lg gap-2"
          >
            <span>{{ getPersonById(personId)?.name }}</span>
            <button
              class="btn btn-ghost btn-xs btn-circle"
              @click="removePerson(personId)"
            >
              ✕
            </button>
          </div>
        </div>
      </div>

      <!-- 搜索框 -->
      <div class="form-control mb-4">
        <input
          v-model="searchKeyword"
          type="text"
          :placeholder="
            t('table.name') +
            ' / ' +
            t('table.number') +
            ' / ' +
            t('table.department')
          "
          class="input input-bordered"
        />
      </div>

      <!-- 人员列表 -->
      <div class="overflow-x-auto max-h-96">
        <table class="table table-zebra table-pin-rows">
          <thead>
            <tr>
              <th>{{ t("table.number") }}</th>
              <th>{{ t("table.name") }}</th>
              <th>{{ t("table.department") }}</th>
              <th>{{ t("table.identity") }}</th>
              <th>{{ t("table.operation") }}</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="person in filteredPersonList"
              :key="person.id"
              :class="{ 'bg-primary/20': isPersonSelected(person.id) }"
            >
              <td>{{ person.uid }}</td>
              <td>{{ person.name }}</td>
              <td>{{ person.department }}</td>
              <td>{{ person.identity }}</td>
              <td>
                <button
                  class="btn btn-sm"
                  :class="
                    isPersonSelected(person.id) ? 'btn-error' : 'btn-primary'
                  "
                  @click="togglePerson(person.id)"
                >
                  {{
                    isPersonSelected(person.id)
                      ? t("button.removeGuaranteed")
                      : t("button.addGuaranteed")
                  }}
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- 底部按钮 -->
      <div class="modal-action">
        <button class="btn" @click="cancel">
          {{ t("button.cancel") }}
        </button>
        <button class="btn btn-primary" @click="confirmSelection">
          {{ t("button.confirm") }}
        </button>
      </div>
    </div>
    <form method="dialog" class="modal-backdrop">
      <button>close</button>
    </form>
  </dialog>
</template>

<style scoped></style>
