<script setup lang="ts">
import type { IFileData } from "@/components/FileUpload/type";
import type { IImage } from "@/types/storeType";
import localforage from "localforage";
import { v4 as uuidv4 } from "uuid";
import { computed, ref, watch } from "vue";
import { useI18n } from "vue-i18n";
import CustomDialog from "@/components/Dialog/index.vue";
import FileUpload from "@/components/FileUpload/index.vue";
import useStore from "@/store";

const { t } = useI18n();
const limitType = ref("image/*");
const imgUploadToast = ref(0); // 0是不显示，1是成功，2是失败,3是不是图片
const visible = defineModel("visible", {
  type: Boolean,
  required: true,
});
const globalConfig = useStore().globalConfig;
const imageDbStore = localforage.createInstance({
  name: "imgStore",
});
const imageData = ref<IFileData | null>(null);
const props = defineProps<{
  editItem?: IImage | null;
}>();

const fileName = ref("");

watch(
  () => props.editItem,
  (val) => {
    if (val) {
      fileName.value = val.name;
    } else {
      fileName.value = "";
    }
  },
  { immediate: true }
);

const uploadDialogRef = ref();

async function uploadFile(fileData: IFileData | null) {
  if (!fileData) {
    imageData.value = null;
    if (!props.editItem) {
      fileName.value = "";
    }
    return;
  }
  const isImage = /image*/.test(fileData?.type || "");
  if (!isImage) {
    imgUploadToast.value = 3;
    return;
  }
  imageData.value = fileData;
  // 仅在“上传模式”（非编辑模式）且当前名称为空时，才根据文件自动填充名称
  if (fileData.fileName && !props.editItem) {
    fileName.value = fileData.fileName;
  }
}
async function getImageDbStore() {
  const keys = await imageDbStore.keys();
  if (keys.length > 0) {
    imageDbStore.iterate(
      (value: { fileName: string; data: Blob }, key: string) => {
        globalConfig.addImage({
          id: key,
          name: value.fileName,
          url: "Storage",
        });
      }
    );
  }
}
function submitUpload() {
  if (imageData.value) {
    const { data } = imageData.value;
    // 使用输入框中的名称，而非文件的原始名称
    const nameToSave = fileName.value || imageData.value.fileName;
    const isEdit = !!props.editItem;
    const uniqueId = props.editItem?.id || uuidv4();
    imageDbStore
      .setItem(uniqueId, {
        data,
        fileName: nameToSave,
      })
      .then(() => {
        imgUploadToast.value = 1;
        const newImage: IImage = {
          id: uniqueId,
          name: nameToSave,
          url: "Storage",
        };
        if (isEdit) {
          globalConfig.updateImage(newImage);
        } else {
          globalConfig.addImage(newImage);
        }
      })
      .catch(() => {
        imgUploadToast.value = 2;
      });
  } else if (props.editItem && fileName.value !== props.editItem.name) {
    // 仅修改名称的情况
    imageDbStore.getItem(props.editItem.id).then((val: any) => {
      if (val) {
        imageDbStore
          .setItem(props.editItem!.id, {
            ...val,
            fileName: fileName.value,
          })
          .then(() => {
            imgUploadToast.value = 1;
            globalConfig.updateImage({
              ...props.editItem!,
              name: fileName.value as string,
            });
          });
      }
    });
  }
}
watch(visible, (newVal) => {
  if (newVal) {
    imageData.value = null;
    uploadDialogRef.value.showDialog();
  }
});
</script>

<template>
  <div class="toast toast-top toast-end">
    <div v-if="imgUploadToast === 2" class="alert alert-error">
      <span>{{ t("error.uploadFail") }}</span>
    </div>
    <div v-if="imgUploadToast === 1" class="alert alert-success">
      <span>{{ t("error.uploadSuccess") }}</span>
    </div>
    <div v-if="imgUploadToast === 3" class="alert alert-error">
      <span>{{ t("error.notImage") }}</span>
    </div>
  </div>
  <CustomDialog
    ref="uploadDialogRef"
    v-model:visible="visible"
    :title="editItem ? t('button.edit') : t('dialog.uploadImageTitle')"
    :submit-func="submitUpload"
    class=""
  >
    <template #content>
      <div class="flex flex-col items-center gap-6 w-full px-12">
        <FileUpload
          v-if="visible"
          :limit-type="limitType"
          @upload-file="uploadFile"
        />
        <input
          v-model="fileName"
          :disabled="imageData === null"
          type="text"
          :placeholder="t('placeHolder.imageName')"
          class="input w-full"
        />
      </div>
    </template>
  </CustomDialog>
</template>

<style scoped></style>
