<template>
  <div class="q-mb-xl bg-primary q-pa-md">
    <q-input v-model="localEditMember.name" label="姓名" />
    <q-input v-model="localEditMember.age" label="年齡" />
    <q-btn-group class="q-mt-md flex gap-2">
      <q-btn color="green" class="q-mr-md" @click="handleEdit">確認修改</q-btn>
      <q-btn color="grey" class="q-ml-md" @click="cancelEdit">取消修改</q-btn>
    </q-btn-group>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';

const props = defineProps({
  editMember: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(['update', 'close']);

const localEditMember = ref({ ...props.editMember });

watch(
  () => props.editMember,
  (newVal) => {
    localEditMember.value = { ...newVal };
  }
);

const handleEdit = async () => {
  emit('update', localEditMember.value);
};

const cancelEdit = () => {
  emit('close');
  console.log('取消修改');
};
</script>

<style scoped lang="scss">
</style>

