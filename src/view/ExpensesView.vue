<script setup>
import { computed, onMounted, ref, watch } from 'vue'

const STORAGE_KEY = 'budgeter2026_mock_expenses_v2'

const expenses = ref([])
const errorMessage = ref('')
const startingBalancePounds = ref('2971.00')

const safeParse = (json) => {
  try {
    return JSON.parse(json)
  } catch {
    return null
  }
}

// Spreadsheet style:
// - `amount_pence` corresponds to "ktg-ek" (true full price for logging/reporting)
// - `active_amount_pence` corresponds to "részem" (net personal deduction used for running balance)
// Back-compat: we still read `cost_pence` + `share_pence` if present in localStorage.
const seedExpenses = () => [
  { id: 'exp_hsbc_loan', name: 'HSBC loan', amount_pence: 22957, active_amount_pence: null, day_of_month: null, currency: 'GBP' },
  { id: 'exp_aviva_pension', name: 'Aviva pension', amount_pence: 8000, active_amount_pence: null, day_of_month: 1, currency: 'GBP' },
  { id: 'exp_telo', name: 'telo', amount_pence: 900, active_amount_pence: null, day_of_month: 1, currency: 'GBP' },
  { id: 'exp_digitalocean', name: 'Digitalocean', amount_pence: 1000, active_amount_pence: null, day_of_month: 1, currency: 'GBP' },
  { id: 'exp_kaja1_ebay', name: 'kaja1+ ebay', amount_pence: 10000, active_amount_pence: null, day_of_month: 1, currency: 'GBP' },
  { id: 'exp_fuel', name: 'fuel', amount_pence: 6000, active_amount_pence: null, day_of_month: 1, currency: 'GBP' },
  { id: 'exp_aa_breakdown', name: 'AA breakdown', amount_pence: 589, active_amount_pence: null, day_of_month: 9, currency: 'GBP' },
  { id: 'exp_spotify', name: 'Spotify', amount_pence: 1200, active_amount_pence: null, day_of_month: 11, currency: 'GBP' },
  { id: 'exp_aviva_car_insurance', name: 'Aviva car insurance', amount_pence: 2453, active_amount_pence: null, day_of_month: 13, currency: 'GBP' },
  { id: 'exp_kaja2_ebay', name: 'kaja2+ ebay', amount_pence: 10000, active_amount_pence: null, day_of_month: null, currency: 'GBP' },
  { id: 'exp_uszi', name: 'uszi', amount_pence: 2000, active_amount_pence: null, day_of_month: null, currency: 'GBP' },
  { id: 'exp_moises', name: 'moises', amount_pence: 399, active_amount_pence: null, day_of_month: 18, currency: 'GBP' },
  { id: 'exp_rezsi', name: 'rezsi', amount_pence: 16500, active_amount_pence: null, day_of_month: 17, currency: 'GBP' },
  { id: 'exp_council_tax', name: 'council tax', amount_pence: 18400, active_amount_pence: null, day_of_month: 1, currency: 'GBP' },
  { id: 'exp_hsbc_life_ins', name: 'HSBC life ins', amount_pence: 546, active_amount_pence: null, day_of_month: 16, currency: 'GBP' },
  { id: 'exp_kaja3_ebay', name: 'kaja3+ebay', amount_pence: 20000, active_amount_pence: 20000, day_of_month: null, currency: 'GBP' },
  { id: 'exp_net', name: 'net', amount_pence: 5400, active_amount_pence: 0, day_of_month: null, currency: 'GBP' },
  { id: 'exp_lakas', name: 'lakas', amount_pence: 89500, active_amount_pence: 41000, day_of_month: 26, currency: 'GBP' },
  { id: 'exp_freeagent', name: 'FreeAgent', amount_pence: 2052, active_amount_pence: 2052, day_of_month: 30, currency: 'GBP' },
  { id: 'exp_zenelek', name: 'zenelek', amount_pence: 1200, active_amount_pence: 1200, day_of_month: 30, currency: 'GBP' },
]

const hydrate = () => {
  if (typeof window === 'undefined') return
  const raw = window.localStorage.getItem(STORAGE_KEY)
  const parsed = raw ? safeParse(raw) : null
  if (parsed && Array.isArray(parsed.expenses)) {
    expenses.value = parsed.expenses
    if (typeof parsed.starting_balance_pounds === 'string') {
      startingBalancePounds.value = parsed.starting_balance_pounds
    }
  } else {
    expenses.value = Array.isArray(parsed) ? parsed : seedExpenses()
  }
}

onMounted(() => {
  hydrate()
})

watch(
  [expenses, startingBalancePounds],
  () => {
    if (typeof window === 'undefined') return
    window.localStorage.setItem(
      STORAGE_KEY,
      JSON.stringify({
        expenses: expenses.value,
        starting_balance_pounds: startingBalancePounds.value,
      }),
    )
  },
  { deep: true },
)

const formatMoney = (amountPence, currency) => {
  const amount = (Number(amountPence) || 0) / 100
  // For now we only format as currency when it’s a 2dp currency.
  return new Intl.NumberFormat('en-GB', {
    style: 'currency',
    currency: currency || 'GBP',
  }).format(amount)
}

const visibleExpenses = computed(() => expenses.value)

const startingBalancePence = computed(() => {
  const pounds = Number(startingBalancePounds.value)
  return Number.isFinite(pounds) ? Math.round(pounds * 100) : 0
})

const rowsWithBalance = computed(() => {
  let running = startingBalancePence.value
  return visibleExpenses.value.map((expense) => {
    const amountPence = Math.abs(Number(expense.amount_pence ?? expense.cost_pence) || 0)

    const activeRaw = expense.active_amount_pence ?? expense.share_pence
    const activeAmountPence =
      activeRaw === null || activeRaw === undefined ? null : Math.abs(Number(activeRaw) || 0)

    running -= activeAmountPence ?? 0
    return { expense, amountPence, activeAmountPence, balanceAfterPence: running }
  })
})

const totalCostLabel = computed(() => {
  const totalPence = visibleExpenses.value.reduce(
    (sum, e) => sum + Math.abs(Number(e.amount_pence ?? e.cost_pence) || 0),
    0,
  )
  return formatMoney(totalPence, 'GBP')
})

const endingBalanceLabel = computed(() => {
  const totalSharePence = visibleExpenses.value.reduce((sum, e) => {
    const activeRaw = e.active_amount_pence ?? e.share_pence
    if (activeRaw === null || activeRaw === undefined) return sum
    return sum + Math.abs(Number(activeRaw) || 0)
  }, 0)
  return formatMoney(startingBalancePence.value - totalSharePence, 'GBP')
})

const totalShareLabel = computed(() => {
  const totalSharePence = visibleExpenses.value.reduce((sum, e) => {
    const activeRaw = e.active_amount_pence ?? e.share_pence
    if (activeRaw === null || activeRaw === undefined) return sum
    return sum + Math.abs(Number(activeRaw) || 0)
  }, 0)
  return formatMoney(totalSharePence, 'GBP')
})
</script>

<template>
  <div class="page expensesPage">
    <div class="sheetTop">
      <div class="sheetTopLeft">
        <div class="sheetTopLabel">Total HSBC</div>
        <input v-model="startingBalancePounds" class="input sheetTopInput" inputmode="decimal" />
      </div>
    </div>

    <p v-if="errorMessage" class="message error">{{ errorMessage }}</p>

    <div class="tableWrap">
      <table class="sheetTable">
        <thead>
          <tr>
            <th class="colExpense">levonasok</th>
            <th class="colCost">ktg-ek</th>
            <th class="colReszem">reszem</th>
            <th class="colDate">date</th>
            <th class="colBalance">balance</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in rowsWithBalance" :key="row.expense.id">
            <td class="colExpense cellName">
              {{ row.expense.name }}
            </td>
            <td class="colCost cellNumber">
              {{ formatMoney(row.amountPence, row.expense.currency).replace(/£/g, '').trim() }}
            </td>
            <td class="colReszem cellNumber">
              {{
                row.activeAmountPence === null
                  ? ''
                  : formatMoney(row.activeAmountPence, row.expense.currency).replace(/£/g, '').trim()
              }}
            </td>
            <td class="colDate cellNumber">
              {{ row.expense.day_of_month ?? '' }}
            </td>
            <td class="colBalance cellNumber cellBalance">
              {{ formatMoney(row.balanceAfterPence, 'GBP').replace(/£/g, '').trim() }}
            </td>
          </tr>

          <tr class="sheetTotalRow">
            <td class="colExpense cellName">Total</td>
            <td class="colCost cellNumber sheetTotal">{{ totalCostLabel.replace(/£/g, '').trim() }}</td>
            <td class="colReszem cellNumber sheetTotal">
              {{ totalShareLabel.replace(/£/g, '').trim() }}
            </td>
            <td class="colDate"></td>
            <td class="colBalance"></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

