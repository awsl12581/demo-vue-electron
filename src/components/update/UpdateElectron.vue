<template>
  <ElButton @click="checkUpdate">Click to open the Message Box</ElButton>
  <ElButton @click="test">Click to open the Message Box</ElButton>
  <ElButton @click="test">Click to open the Message Box</ElButton>
  <div>{{ updateError?.message }}</div>
</template>

<script setup lang="ts">
import { IpcRendererEvent, ipcRenderer } from 'electron';
import { ref, onMounted, watch } from 'vue';
import type { ProgressInfo } from 'electron-updater';
import console from 'console';

const test = () => {
  console.log('1234');
};

const checking = ref(false);
const updateAvailable = ref(false);
const versionInfo = ref<VersionInfo>();
const updateError = ref<ErrorType>();
const progressInfo = ref<Partial<ProgressInfo>>();
const modalOpen = ref(false);
const modalBtn = ref<{
  cancelText?: string;
  okText?: string;
  onCancel?: () => void;
  onOk?: () => void;
}>({
  onCancel: () => (modalOpen.value = false),
  onOk: () => ipcRenderer.invoke('start-download')
});

watch(updateError, () => {
  console.log(updateError.value);
});

const checkUpdate = async () => {
  checking.value = true;
  console.log('1234');
  /**
   * @type {import('electron-updater').UpdateCheckResult | null | { message: string, error: Error }}
   */
  const result = await ipcRenderer.invoke('check-update');

  progressInfo.value = { percent: 0 };
  checking.value = false;
  modalOpen.value = true;
  if (result?.error) {
    updateAvailable.value = true;
    updateError.value = result?.error;
  }
};

const onUpdateCanAvailable = (_event: IpcRendererEvent, arg1: VersionInfo) => {
  versionInfo.value = arg1;
  updateError.value = undefined;
  // Can be update
  if (arg1.update) {
    modalBtn.value = {
      cancelText: 'Cancel',
      okText: 'Update',
      onOk: () => ipcRenderer.invoke('start-download')
    };
    updateAvailable.value = true;
  } else {
    updateAvailable.value = false;
  }
};

const onUpdateError = (_event: IpcRendererEvent, arg1: ErrorType) => {
  updateAvailable.value = false;
  console.log(arg1);
  updateError.value = arg1;
};

const onDownloadProgress = (_event: IpcRendererEvent, arg1: ProgressInfo) => {
  progressInfo.value = arg1;
};

// eslint-disable-next-line @typescript-eslint/no-unused-vars
const onUpdateDownloaded = (_event: IpcRendererEvent, ..._args: unknown[]) => {
  progressInfo.value = { percent: 100 };
  modalBtn.value = {
    cancelText: 'Later',
    okText: 'Install now',
    onOk: () => ipcRenderer.invoke('quit-and-install')
  };
};

onMounted(() => {
  // Get version information and whether to update
  ipcRenderer.on('update-can-available', onUpdateCanAvailable);
  ipcRenderer.on('update-error', onUpdateError);
  ipcRenderer.on('download-progress', onDownloadProgress);
  ipcRenderer.on('update-downloaded', onUpdateDownloaded);

  return () => {
    ipcRenderer.off('update-can-available', onUpdateCanAvailable);
    ipcRenderer.off('update-error', onUpdateError);
    ipcRenderer.off('download-progress', onDownloadProgress);
    ipcRenderer.off('update-downloaded', onUpdateDownloaded);
  };
});
</script>
