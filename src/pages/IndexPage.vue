<template>
  <q-page class="row items-center justify-center">
    <q-file
      filled
      bottom-slots
      v-model="file"
      label="Attach some file"
      counter
      @update:model-value="onFileSelected"
      class="col-10"
    >
      <template v-slot:prepend>
        <q-icon name="fa fa-paperclip" @click.stop.prevent />
      </template>
      <template v-slot:append>
        <q-icon
          name="fa fa-xmark"
          @click.stop.prevent="file = null"
          class="cursor-pointer"
        />
      </template>
    </q-file>
    <q-list bordered separator class="col-10 text-center">
      <q-item v-for="file in fileList" :key="file.uri" clickable v-ripple>
        <q-item-section @click="share(file)">
          {{ file.name }}
        </q-item-section>
      </q-item>
    </q-list>
  </q-page>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { FileInfo, Filesystem } from '@capacitor/filesystem';
import { Directory } from '@capacitor/filesystem';
import { Buffer } from 'buffer';
import { Share } from '@capacitor/share';
// import { FileOpener } from '@capawesome-team/capacitor-file-opener';

const subdir = '';
const file = ref<File | null>(null);
const onFileSelected = async () => {
  if (file.value) {
    try {
      await Filesystem.writeFile({
        path: subdir + file.value.name,
        data: Buffer.from(await file.value.arrayBuffer()).toString('base64'),
        directory: Directory.Data,
        recursive: true,
      });
      file.value = null;
      refreshFileList();
    } catch (error) {
      console.log(error);
    }
  }
};
const share = async (file: FileInfo) => {
  await Share.share({
    url: file.uri,
  });
};

const fileList = ref<FileInfo[]>();
const refreshFileList = async () => {
  fileList.value = (
    await Filesystem.readdir({
      path: subdir,
      directory: Directory.Data,
    })
  ).files;
};
onMounted(() => refreshFileList());

defineOptions({
  name: 'IndexPage',
});
</script>
