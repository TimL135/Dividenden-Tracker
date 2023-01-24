<template>
  <div>
    <Accordion :items="items">
      <template #head>
        <div>
          <TextInput placeholder="Aktie" v-model="share">
            <template #button
              ><Button sideButton @click="newShare">anlegen</Button></template
            >
          </TextInput>
        </div>
      </template>
      <template v-for="share of shares" #[stringWithoutSpace(share.name)]>
        <div class="d-flex">
          <div class="col-5 col-sm-4 me-2">
            <NumberInput
              placeholder="dividende"
              v-model="dividend"
            ></NumberInput>
          </div>
          <div class="col-5 col-sm-4 me-2">
            <DateInput placeholder="Datum" v-model="date"></DateInput>
          </div>
          <div class="col-2 col mt-3">
            <Button @click="newDividend(share)">erstellen</Button>
          </div>
        </div>
        <div v-for="dividend of share.dividends" class="d-flex">
          <div class="col-5 col-sm-4 me-2">
            <NumberInput
              placeholder="dividende"
              v-model="dividend.dividend"
            ></NumberInput>
          </div>
          <div class="col-5 col-sm-4">
            <DateInput placeholder="Datum" v-model="dividend.date"></DateInput>
          </div>
        </div>
      </template>
      <template #Monate>
        <div v-for="month of months">{{ month.month }}:{{ month.sum }}</div>
      </template>
      <template #Gesamt>Gesamt({{ all }})</template>
    </Accordion>
  </div>
</template>
<script setup lang="ts">
import {
  Accordion,
  TextInput,
  Button,
  NumberInput,
  DateInput,
} from "custom-mbd-components";
import { computed, ref, watch } from "vue";
const items = computed(() => {
  const array: any = [{ title: "", hash: "head", noAccordion: true }];
  shares.value.forEach((e: any) =>
    array.push({ title: e.name, hash: stringWithoutSpace(e.name) })
  );
  array.push({ title: "Monate", hash: "Monate" });
  array.push({
    title: "Gesamt",
    hash: "Gesamt",
    noAccordion: true,
  });
  return array;
});

const share = ref("");
const shares = ref<any>(JSON.parse(localStorage.getItem("shares") || "[]"));
const dividend = ref("");
const date = ref("");
const months = computed(() => {
  const object = {} as any;
  shares.value.forEach((share: any) =>
    share.dividends.forEach((dividend: any) =>
      object[dividend.date.slice(0, -3)]
        ? (object[dividend.date.slice(0, -3)].sum += dividend.dividend)
        : (object[dividend.date.slice(0, -3)] = {
            month: dividend.date.slice(0, -3),
            sum: dividend.dividend,
          })
    )
  );
  return object;
});
const all = computed(() => {
  let sum = 0;
  if (Object.values(months.value).length > 0)
    sum = Object.values(months.value).reduce((a: any, b: any) => a + b.sum, 0);
  return sum;
});
function newShare() {
  shares.value.push({ name: share.value, dividends: [] });
  share.value = "";
}
function newDividend(share: any) {
  if (!date.value || !dividend.value) return;
  share.dividends.push({ date: date.value, dividend: dividend.value });
  date.value = "";
  dividend.value = "";
}
function stringWithoutSpace(string: string) {
  return string.replaceAll(" ", "");
}
watch(
  () => shares.value,
  (newValue, oldValue) => {
    localStorage.setItem("shares", JSON.stringify(newValue));
  },
  { deep: true }
);
</script>
<style scoped></style>
