<template>
  <div class="container">
    <Accordion :items="items">
      <template #head>
        <h5>Aktien</h5>
        <div>
          <TextInput placeholder="Aktie" v-model="share">
            <template #button
              ><Button sideButton @click="newShare">anlegen</Button></template
            >
          </TextInput>
        </div>
      </template>
      <template
        v-for="share of shares"
        #[stringWithoutSpecialCharacters(share.name)]
      >
        <Modal title="Einstellungen">
          <Button
            class="btn btn-danger"
            @click="
              shares = shares.filter(
                (e) => JSON.stringify(e) != JSON.stringify(share)
              )
            "
            >Aktie Löschen</Button
          >
          <div>
            <NumberInput
              placeholder="Dividende in %"
              v-model="share.percent"
              step="0.01"
            ></NumberInput>
          </div>
          <div>
            <NumberInput
              placeholder="Sparrate pro Monat"
              v-model="share.rate"
              step="0.01"
            >
            </NumberInput>
            <SelectInput
              placeholder="ausschüttung"
              v-model="share.payoutRate"
              :options="['monat', 'quartal', 'halbesJahr', 'jahr']"
              showAll
            >
            </SelectInput>
          </div>
          <template #button>
            <div class="w-100 px-2">
              <Button class="btn btn-primary w-100">Einstellungen</Button>
            </div></template
          >
        </Modal>
        <div class="d-flex px-2">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput
              placeholder="dividende"
              v-model="dividend"
              step="0.01"
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
              step="0.01"
            ></NumberInput>
          </div>
          <div class="col-4 col-sm-5">
            <DateInput placeholder="Datum" v-model="dividend.date"></DateInput>
          </div>
        </div>
      </template>
      <template #Monate>
        <div v-for="month of months">
          {{ month.month }}: {{ month.sum.toFixed(2) }}€
        </div>
      </template>
      <template #Info>
        <div>Geschätzte Steigerung: {{ increase.toFixed(2) }}€ pro Monat</div>
        <div>
          Durchschnittliche Dividende: {{ avgPercent.toFixed(2) }}% pro Jahr
        </div>
        <div class="row">
          <div class="col-3">Jahre</div>
          <div class="col-3">Investiert</div>
          <div class="col-3">Portfolio</div>
          <div class="col-3">Dividende</div>
        </div>
        <div v-for="(year, index) of years" class="row">
          <div class="col-3">{{ year }}:</div>
          <div class="col-3">{{ invests[index] }}€</div>
          <div class="col-3">{{ portfolio[index] }}€</div>
          <div class="col-3">{{ dividende[index] }}€</div>
        </div>
      </template>
      <template #Gesamt>Gesamt({{ all }}€)</template>
      <template #finances><h5>Finanzen</h5></template>
      <template #regular><h6>Monatlich</h6></template>
      <template #income>
        <div class="d-flex">
          <div class="col-4 col-sm-5 me-2">
            <NumberInput
              placeholder="Einahme"
              v-model="income"
              step="0.01"
            ></NumberInput>
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
              step="0.01"
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
            <NumberInput
              placeholder="Ausgabe"
              v-model="outcome"
              step="0.01"
            ></NumberInput>
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
              step="0.01"
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
      <template #erratic><h6>Unregelmäßig</h6></template>
      <template #erraticIncome>
        <div class="row">
          <div class="col-6">
            <NumberInput
              placeholder="Einahme"
              v-model="erraticIncome"
              step="0.01"
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
              step="0.01"
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
              step="0.01"
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
              step="0.01"
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
          {{ month.month }}: {{ month.sum.toFixed(2) }}€
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
  Modal,
  SelectInput,
} from "custom-mbd-components";
import { computed, ref, watch } from "vue";
interface Share {
  name: string;
  dividends: Dividend[];
  rate: string;
  percent: string;
  payoutRate: string;
}
interface Dividend {
  date: string;
  dividend: string;
}
const payoutRates = ref<{ [key: string]: number }>({
  monat: 12,
  quartal: 4,
  halbesJahr: 2,
  jahr: 1,
});
const years = ref([1, 2, 3, 5, 10, 15, 20, 25]);
const items = computed(() => {
  const array: any = [{ title: "", hash: "head", noAccordion: true }];
  shares.value.forEach((e: Share) =>
    array.push({
      title: `${e.name}(${e.dividends
        .reduce((a, b) => a + +b.dividend, 0)
        .toFixed(2)}€)`,
      hash: stringWithoutSpecialCharacters(e.name),
    })
  );
  array.push({ title: "Monate", hash: "Monate" });
  array.push({ title: "Info", hash: "Info" });
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
        ? (object[dividend.date.slice(0, -3)].sum += +dividend.dividend)
        : (object[dividend.date.slice(0, -3)] = {
            month: dividend.date.slice(0, -3),
            sum: +dividend.dividend,
          })
    )
  );
  return object;
});
const increase = computed(() => {
  let eur = 0;
  shares.value.forEach((share) => {
    eur += ((+share.rate ?? 0) * ((+share.percent ?? 0) / 100)) / 12;
  });
  return eur;
});
const avgPercent = computed(() => {
  let eur = shares.value.reduce((a, b) => a + +b.rate, 0);
  return ((increase.value * 12) / eur) * 100 || 0;
});
const all = computed(() => {
  let sum = 0;
  if (Object.values(months.value).length > 0)
    sum = Object.values(months.value).reduce((a, b) => a + b.sum, 0);
  return sum.toFixed(2);
});
const invests = computed(() => {
  const arr = [];
  const sum = shares.value.reduce((a, b) => a + +b.rate * 12, 0);
  for (let i of years.value) {
    arr.push(sum * i);
  }
  return arr;
});
const portfolio = computed(() => {
  let arr = [];
  for (let year of years.value) {
    let months = year * 12;
    let total = 0;
    for (let s of shares.value) {
      console.log(s);
      let sumShare = 0;
      for (let i = 0; i < months; i++) {
        sumShare =
          (sumShare + +s.rate) *
          (1 + +s.percent / payoutRates.value[s.payoutRate] / 100);
      }
      total += sumShare;
    }
    arr.push(total.toFixed(0));
  }
  return arr;
});

const dividende = computed(() => {
  const arr = [];
  for (const index in years.value) {
    arr.push((+portfolio.value[index] - invests.value[index]).toFixed());
  }
  return arr;
});
function newShare() {
  if (!share.value) return;
  shares.value.push({
    name: share.value,
    dividends: [],
    rate: "",
    percent: "",
    payoutRate: "",
  });
  share.value = "";
}
function newDividend(share: Share) {
  if (!date.value || !dividend.value) return;
  share.dividends.push({
    date: date.value,
    dividend: dividend.value,
  });
  date.value = "";
  dividend.value = "";
}
function stringWithoutSpecialCharacters(string: string) {
  return string.replace(/[^a-zA-Z0-9-_]/g, "");
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
  finances.value.income.push({
    amount: parseInt(income.value),
    reason: source.value,
  });
  income.value = "";
  source.value = "";
}
const outcome = ref("");
const reason = ref("");
function newOutcome() {
  if (!outcome.value || !reason.value) return;
  finances.value.outcome.push({
    amount: parseInt(outcome.value),
    reason: reason.value,
  });
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
    amount: parseInt(erraticIncome.value),
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
    amount: parseInt(erraticOutcome.value),
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
