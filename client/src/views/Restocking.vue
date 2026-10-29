<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking</h2>
      <p>Set your available budget and review recommended items based on demand forecasts.</p>
    </div>

    <div v-if="loading" class="loading">Loading...</div>
    <div v-else-if="error && !submitted" class="error">{{ error }}</div>
    <div v-else>
      <div class="card budget-card">
        <div class="budget-row">
          <label class="budget-label">Available Budget</label>
          <span class="budget-value">${{ budget.toLocaleString('en-US') }}</span>
        </div>
        <input
          type="range"
          v-model.number="budget"
          min="1000"
          max="500000"
          step="1000"
          class="budget-slider"
        />
        <p class="budget-hint">Drag to adjust. Min: $1,000 — Max: $500,000</p>
      </div>

      <div class="card" v-if="recommendedItems.length > 0">
        <div class="card-header">
          <h3 class="card-title">Recommended Items ({{ recommendedItems.length }})</h3>
        </div>
        <div class="table-container">
          <table class="restock-table">
            <thead>
              <tr>
                <th>SKU</th>
                <th>Item Name</th>
                <th>Qty to Restock</th>
                <th>Unit Cost</th>
                <th>Subtotal</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendedItems" :key="item.sku">
                <td><code class="sku">{{ item.sku }}</code></td>
                <td>{{ item.name }}</td>
                <td>{{ item.quantity.toLocaleString() }}</td>
                <td>${{ item.unit_cost.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</td>
                <td><strong>${{ item.total_cost.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong></td>
              </tr>
            </tbody>
            <tfoot>
              <tr class="tfoot-row">
                <td colspan="4" class="tfoot-label">Total Spend</td>
                <td class="tfoot-value">
                  <strong>${{ totalSpend.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong>
                </td>
              </tr>
              <tr class="tfoot-row">
                <td colspan="4" class="tfoot-label">Budget Remaining</td>
                <td :class="['tfoot-value', budgetRemaining >= 0 ? 'budget-remaining-positive' : 'budget-remaining-negative']">
                  <strong>${{ budgetRemaining.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong>
                </td>
              </tr>
            </tfoot>
          </table>
        </div>
      </div>

      <div v-else class="card empty-state">
        <p>No items can be restocked within the current budget.</p>
      </div>

      <div class="action-area">
        <div v-if="submitted" class="success-box">
          <div class="success-title">Order Placed</div>
          <div class="success-order-number">{{ lastOrder.order_number }}</div>
          <div class="success-detail">Estimated delivery: 7-10 business days</div>
          <button class="btn-reset" @click="resetOrder">Place another order</button>
        </div>
        <button
          v-else
          class="btn-primary"
          :disabled="submitting || submitted || recommendedItems.length === 0"
          @click="placeOrder"
        >
          {{ submitting ? 'Placing Order...' : 'Place Order' }}
        </button>
        <div v-if="error && submitted" class="error">{{ error }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'

export default {
  name: 'Restocking',
  setup() {
    const forecasts = ref([])
    const itemCostMap = ref(new Map())
    const budget = ref(50000)
    const submitting = ref(false)
    const submitted = ref(false)
    const lastOrder = ref(null)
    const loading = ref(true)
    const error = ref(null)

    const recommendedItems = computed(() => {
      const sorted = [...forecasts.value].sort((a, b) => b.forecasted_demand - a.forecasted_demand)
      let remaining = budget.value
      const result = []
      for (const f of sorted) {
        const unitCost = itemCostMap.value.get(f.item_sku) ?? 0
        if (unitCost === 0) continue
        const lineCost = f.forecasted_demand * unitCost
        if (lineCost <= remaining) {
          result.push({ sku: f.item_sku, name: f.item_name, quantity: f.forecasted_demand, unit_cost: unitCost, total_cost: lineCost })
          remaining -= lineCost
        }
      }
      return result
    })

    const totalSpend = computed(() => recommendedItems.value.reduce((s, i) => s + i.total_cost, 0))

    const budgetRemaining = computed(() => budget.value - totalSpend.value)

    const placeOrder = async () => {
      submitting.value = true
      try {
        lastOrder.value = await api.createRestockingOrder({
          items: recommendedItems.value,
          total_value: totalSpend.value
        })
        submitted.value = true
      } catch (err) {
        error.value = 'Failed to place order'
        console.error(err)
      } finally {
        submitting.value = false
      }
    }

    const resetOrder = () => {
      submitted.value = false
      lastOrder.value = null
      budget.value = 50000
    }

    onMounted(async () => {
      try {
        const [forecastData, inventoryData] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory()
        ])
        forecasts.value = forecastData
        const costMap = new Map()
        for (const item of inventoryData) {
          costMap.set(item.sku, item.unit_cost)
        }
        itemCostMap.value = costMap
      } catch (err) {
        error.value = 'Failed to load data'
        console.error(err)
      } finally {
        loading.value = false
      }
    })

    return {
      forecasts,
      budget,
      submitting,
      submitted,
      lastOrder,
      loading,
      error,
      recommendedItems,
      totalSpend,
      budgetRemaining,
      placeOrder,
      resetOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 2rem;
}

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: #e2e8f0;
  margin: 0 0 0.25rem;
}

.page-header p {
  color: #64748b;
  margin: 0;
  font-size: 0.9rem;
}

.card {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.budget-card {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.budget-row {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.budget-label {
  font-size: 0.9rem;
  color: #94a3b8;
  font-weight: 500;
}

.budget-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #e2e8f0;
}

.budget-slider {
  width: 100%;
  accent-color: #3b82f6;
  cursor: pointer;
}

.budget-hint {
  font-size: 0.78rem;
  color: #64748b;
  margin: 0;
}

.card-header {
  margin-bottom: 1rem;
}

.card-title {
  font-size: 1rem;
  font-weight: 600;
  color: #e2e8f0;
  margin: 0;
}

.table-container {
  overflow-x: auto;
}

.restock-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.restock-table thead tr {
  border-bottom: 1px solid #334155;
}

.restock-table th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  color: #64748b;
  font-weight: 500;
  font-size: 0.78rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.restock-table tbody tr {
  border-bottom: 1px solid #1e293b;
}

.restock-table tbody tr:last-child {
  border-bottom: none;
}

.restock-table td {
  padding: 0.625rem 0.75rem;
  color: #cbd5e1;
}

.restock-table tfoot {
  border-top: 1px solid #334155;
}

.tfoot-row td {
  padding: 0.5rem 0.75rem;
}

.tfoot-label {
  color: #94a3b8;
  font-size: 0.85rem;
  text-align: right;
}

.tfoot-value {
  color: #e2e8f0;
  font-size: 0.9rem;
}

.budget-remaining-positive {
  color: #22c55e;
}

.budget-remaining-negative {
  color: #ef4444;
}

.sku {
  font-family: monospace;
  font-size: 0.8rem;
  color: #93c5fd;
  background: #0f172a;
  padding: 0.1rem 0.35rem;
  border-radius: 3px;
}

.empty-state {
  color: #64748b;
  font-size: 0.9rem;
}

.action-area {
  margin-top: 0.5rem;
}

.btn-primary {
  background: #3b82f6;
  color: white;
  border: none;
  padding: 0.625rem 1.5rem;
  border-radius: 6px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-primary:hover:not(:disabled) {
  background: #2563eb;
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.success-box {
  background: #14291a;
  border: 1px solid #22c55e;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  color: #86efac;
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  max-width: 420px;
}

.success-title {
  font-weight: 700;
  font-size: 0.95rem;
}

.success-order-number {
  font-size: 1.1rem;
  font-weight: 700;
  color: #4ade80;
}

.success-detail {
  font-size: 0.85rem;
  color: #86efac;
}

.btn-reset {
  margin-top: 0.5rem;
  background: transparent;
  border: 1px solid #22c55e;
  color: #4ade80;
  padding: 0.35rem 0.875rem;
  border-radius: 5px;
  font-size: 0.8rem;
  cursor: pointer;
  align-self: flex-start;
}

.btn-reset:hover {
  background: #1a3d24;
}

.loading {
  color: #64748b;
  padding: 2rem 0;
}

.error {
  color: #f87171;
  padding: 0.75rem 0;
  font-size: 0.9rem;
}
</style>
