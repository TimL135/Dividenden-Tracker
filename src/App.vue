<template>
  <div class="container">
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
      <template #dividends>Dividenden</template>
      <template v-for="share of shares" #[stringWithoutSpace(share.name)]>
        <div class="d-flex">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput
              placeholder="dividende"
              v-model="dividend"
            ></NumberInput>
          </div>
          <div class="col-4 col-sm-5 me-2">
            <DateInput placeholder="Datum" v-model="date"></DateInput>
          </div>
          <div class="col-2 mt-3">
            <Button @click="newDividend(share)">erstellen</Button>
          </div>
        </div>
        <div v-for="dividend of share.dividends" class="d-flex">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput
              placeholder="dividende"
              v-model="dividend.dividend"
            ></NumberInput>
          </div>
          <div class="col-4 col-sm-5">
            <DateInput placeholder="Datum" v-model="dividend.date"></DateInput>
          </div>
        </div>
      </template>
      <template #Monate>
        <div v-for="month of months">{{ month.month }}: {{ month.sum }}€</div>
      </template>
      <template #Gesamt>Gesamt({{ all }}€)</template>
      <template #finances>Finanzen</template>
      <template #regular>Monatlich</template>
      <template #income>
        <div class="d-flex">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput placeholder="Einahme" v-model="income"></NumberInput>
          </div>
          <div class="col-4 col-sm-5 me-2">
            <TextInput placeholder="Quelle" v-model="source"></TextInput>
          </div>
          <div class="mt-3"><Button @click="newIncome">erstellen</Button></div>
        </div>
        <div v-for="(income, index) of finances.income" class="d-flex">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput
              placeholder="Einahme"
              v-model="income.amount"
            ></NumberInput>
          </div>
          <div class="col-4 col-sm-5 me-2">
            <TextInput placeholder="Quelle" v-model="income.reason"></TextInput>
          </div>
          <div class="mt-3">
            <Button
              class="btn btn-danger"
              @click="
                () =>
                  (finances.income = finances.income.filter(
                    (e, i) => i != index
                  ))
              "
              >löschen</Button
            >
          </div>
        </div>
      </template>
      <template #outcome>
        <div class="d-flex">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput placeholder="Ausgabe" v-model="outcome"></NumberInput>
          </div>
          <div class="col-4 col-sm-5 me-2">
            <TextInput placeholder="Grund" v-model="reason"></TextInput>
          </div>
          <div class="mt-3"><Button @click="newOutcome">erstellen</Button></div>
        </div>
        <div v-for="(outcome, index) of finances.outcome" class="d-flex">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput
              placeholder="Ausgabe"
              v-model="outcome.amount"
            ></NumberInput>
          </div>
          <div class="col-4 col-sm-5 me-2">
            <TextInput placeholder="Grund" v-model="outcome.reason"></TextInput>
          </div>
          <div class="mt-3">
            <Button
              class="btn btn-danger"
              @click="
                () =>
                  (finances.outcome = finances.outcome.filter(
                    (e, i) => i != index
                  ))
              "
              >löschen</Button
            >
          </div>
        </div>
      </template>
      <template #differenz
        >Differenz: {{ (incomes - outcomes).toFixed(2) }}€</template
      >
      <template #erratic>Unregelmäßig</template>
      <template #erraticIncome>
        <div class="row">
          <div class="col-6">
            <NumberInput
              placeholder="Einahme"
              v-model="erraticIncome"
            ></NumberInput>
          </div>
          <div class="col-6">
            <TextInput placeholder="Quelle" v-model="erraticSource"></TextInput>
          </div>
          <div class="col-6">
            <DateInput placeholder="Tag" v-model="erraticDate"></DateInput>
          </div>
          <div class="col-6 mt-3">
            <Button @click="newErraticIncome">erstellen</Button>
          </div>
        </div>
        <div
          v-for="(erraticIncome, index) of finances.erraticIncome"
          class="row g-1"
        >
          <div class="col-3">
            <NumberInput
              placeholder="Einahme"
              v-model="erraticIncome.amount"
            ></NumberInput>
          </div>
          <div class="col-4">
            <TextInput
              placeholder="Quelle"
              v-model="erraticIncome.reason"
            ></TextInput>
          </div>
          <div class="col-5">
            <DateInput
              placeholder="Tag"
              v-model="erraticIncome.date"
            ></DateInput>
          </div>
        </div>
      </template>
      <template #erraticOutcome>
        <div class="row">
          <div class="col-6">
            <NumberInput
              placeholder="Ausgabe"
              v-model="erraticOutcome"
            ></NumberInput>
          </div>
          <div class="col-6">
            <TextInput placeholder="Grund" v-model="erraticReason"></TextInput>
          </div>
          <div class="col-6">
            <DateInput placeholder="Tag" v-model="erraticOutDate"></DateInput>
          </div>
          <div class="col-6 mt-3">
            <Button @click="newErraticOutcome">erstellen</Button>
          </div>
        </div>
        <div
          v-for="(erraticOutcome, index) of finances.erraticOutcome"
          class="row g-1"
        >
          <div class="col-3">
            <NumberInput
              placeholder="Ausgabe"
              v-model="erraticOutcome.amount"
            ></NumberInput>
          </div>
          <div class="col-4">
            <TextInput
              placeholder="Grund"
              v-model="erraticOutcome.reason"
            ></TextInput>
          </div>
          <div class="col-5">
            <DateInput
              placeholder="Tag"
              v-model="erraticOutcome.date"
            ></DateInput>
          </div></div
      ></template>
      <template #erraticMonate>
        <div v-for="month of erraticMonths">
          {{ month.month }}: {{ month.sum }}€
        </div>
      </template>
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
interface Share {
  name: string;
  dividends: Dividend[];
}
interface Dividend {
  date: string;
  dividend: number;
}
const items = computed(() => {
  const array: any = [{ title: "", hash: "head", noAccordion: true }];
  array.push({ title: "", hash: "dividends", noAccordion: true });
  shares.value.forEach((e: Share) =>
    array.push({
      title: `${e.name}(${e.dividends.reduce((a, b) => a + b.dividend, 0)}€)`,
      hash: stringWithoutSpace(e.name),
    })
  );
  array.push({ title: "Monate", hash: "Monate" });
  array.push({
    title: "Gesamt",
    hash: "Gesamt",
    noAccordion: true,
  });
  array.push({ title: "", hash: "finances", noAccordion: true });
  array.push({ title: "", hash: "regular", noAccordion: true });
  array.push({ title: `Einnahmen(${incomes.value}€)`, hash: "income" });
  array.push({ title: `Ausgaben(${outcomes.value}€)`, hash: "outcome" });
  array.push({ title: "", hash: "differenz", noAccordion: true });
  array.push({ title: "", hash: "erratic", noAccordion: true });
  array.push({ title: `Einnahmen`, hash: "erraticIncome" });
  array.push({ title: `Ausgaben`, hash: "erraticOutcome" });
  array.push({ title: `Monate`, hash: "erraticMonate" });

  return array;
});

const share = ref("");
const shares = ref<Share[]>(JSON.parse(localStorage.getItem("shares") || "[]"));
const dividend = ref("");
const date = ref("");
interface Month {
  month: string;
  sum: number;
}
const months = computed(() => {
  const object = {} as { [key: string]: Month };
  shares.value.forEach((share: Share) =>
    share.dividends.forEach((dividend: Dividend) =>
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
    sum = Object.values(months.value).reduce((a, b) => a + b.sum, 0);
  return sum;
});
function newShare() {
  if (!share.value) return;
  shares.value.push({ name: share.value, dividends: [] });
  share.value = "";
}
function newDividend(share: Share) {
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
interface Finances {
  income: IoCome[];
  outcome: IoCome[];
  erraticIncome: erraticIoCome[];
  erraticOutcome: erraticIoCome[];
}
interface IoCome {
  amount: number;
  reason: string;
}
interface erraticIoCome {
  amount: number;
  reason: string;
  date: string;
}
const finances = ref<Finances>(
  JSON.parse(localStorage.getItem("finances") || '{"income":[],"outcome":[]}')
);
finances.value.erraticIncome =
  finances.value.erraticIncome?.filter(
    (e) =>
      new Date(e.date).toISOString() >
      new Date(new Date().setMonth(new Date().getMonth() - 3)).toISOString()
  ) || [];
finances.value.erraticOutcome =
  finances.value.erraticOutcome?.filter(
    (e) =>
      new Date(e.date).toISOString() >
      new Date(new Date().setMonth(new Date().getMonth() - 3)).toISOString()
  ) || [];

watch(
  () => finances.value,
  (newValue, oldValue) => {
    localStorage.setItem("finances", JSON.stringify(newValue));
  },
  { deep: true }
);
const income = ref("");
const source = ref("");
const incomes = computed(() =>
  finances.value.income.reduce((a, b) => a + b.amount * 1, 0).toFixed(2)
);
const outcomes = computed(() =>
  finances.value.outcome.reduce((a, b) => a + b.amount * 1, 0).toFixed(2)
);
function newIncome() {
  if (!income.value || !source.value) return;
  finances.value.income.push({ amount: income.value, reason: source.value });
  income.value = "";
  source.value = "";
}
const outcome = ref("");
const reason = ref("");
function newOutcome() {
  if (!outcome.value || !reason.value) return;
  finances.value.outcome.push({ amount: outcome.value, reason: reason.value });
  outcome.value = "";
  reason.value = "";
}
const erraticIncome = ref("");
const erraticSource = ref("");
const erraticDate = ref("");
function newErraticIncome() {
  if (!erraticIncome.value || !erraticSource.value || !erraticDate.value)
    return;
  finances.value.erraticIncome.push({
    amount: erraticIncome.value,
    reason: erraticSource.value,
    date: erraticDate.value,
  });
  erraticIncome.value = "";
  erraticSource.value = "";
  erraticDate.value = "";
}
const erraticOutcome = ref("");
const erraticReason = ref("");
const erraticOutDate = ref("");
function newErraticOutcome() {
  if (!erraticOutcome.value || !erraticReason.value || !erraticOutDate.value)
    return;
  finances.value.erraticOutcome.push({
    amount: erraticOutcome.value,
    reason: erraticReason.value,
    date: erraticOutDate.value,
  });
  erraticOutcome.value = "";
  erraticReason.value = "";
  erraticOutDate.value = "";
}
const erraticMonths = computed(() => {
  const object = {} as { [key: string]: Month };
  finances.value.erraticIncome.forEach((income: erraticIoCome) =>
    object[income.date.slice(0, -3)]
      ? (object[income.date.slice(0, -3)].sum += income.amount)
      : (object[income.date.slice(0, -3)] = {
          month: income.date.slice(0, -3),
          sum: income.amount,
        })
  );
  finances.value.erraticOutcome.forEach((outcome: erraticIoCome) =>
    object[outcome.date.slice(0, -3)]
      ? (object[outcome.date.slice(0, -3)].sum -= outcome.amount)
      : (object[outcome.date.slice(0, -3)] = {
          month: outcome.date.slice(0, -3),
          sum: -outcome.amount,
        })
  );
  return object;
});
</script>
<style scoped></style>
