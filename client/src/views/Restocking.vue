<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div class="restocking-container">
      <!-- Budget Section -->
      <div class="budget-section card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budgetLabel') }}</h3>
        </div>
        <div class="budget-controls">
          <div class="slider-container">
            <input
              v-model.number="budget"
              type="range"
              min="0"
              max="10000"
              step="100"
              class="budget-slider"
            />
          </div>
          <div class="budget-display">
            <div class="budget-value">
              <span class="label">{{ t('restocking.budgetLabel') }}:</span>
              <span class="amount">${{ budget.toLocaleString() }}</span>
            </div>
            <div class="budget-remaining" v-if="recommendations.length > 0">
              <span class="label">{{ t('restocking.remainingBudget') }}:</span>
              <span class="amount">{{ remainingBudgetFormatted }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Recommendations Section -->
      <div class="recommendations-section">
        <div class="section-header">
          <h3>{{ t('restocking.recommendedItems') }}</h3>
          <div v-if="recommendations.length > 0" class="summary-stats">
            <div class="stat">
              <span class="label">{{ t('restocking.totalItems') }}:</span>
              <span class="value">{{ totalItemsCount }}</span>
            </div>
            <div class="stat">
              <span class="label">{{ t('restocking.totalCost') }}:</span>
              <span class="value">{{ totalCostFormatted }}</span>
            </div>
          </div>
        </div>

        <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
        <div v-else-if="error" class="error">{{ error }}</div>
        <div v-else-if="recommendations.length === 0" class="empty-state">
          {{ t('restocking.noRecommendations') }}
        </div>
        <div v-else class="recommendations-grid">
          <div v-for="item in recommendations" :key="item.id" class="recommendation-card">
            <div class="card-content">
              <div class="item-header">
                <div class="item-info">
                  <div class="item-name">{{ item.item_name }}</div>
                  <div class="item-sku">SKU: {{ item.item_sku }}</div>
                </div>
              </div>

              <div class="demand-info">
                <div class="demand-row">
                  <span class="label">{{ t('restocking.currentDemand') }}:</span>
                  <span class="value">{{ item.current_demand }}</span>
                </div>
                <div class="demand-row">
                  <span class="label">{{ t('restocking.forecastedDemand') }}:</span>
                  <span class="value highlighted">{{ item.forecasted_demand }}</span>
                </div>
              </div>

              <div class="pricing-info">
                <div class="price-row">
                  <span class="label">{{ t('restocking.unitCost') }}:</span>
                  <span class="value">${{ item.unit_cost?.toFixed(2) || '0.00' }}</span>
                </div>
              </div>

              <div class="order-info">
                <div class="order-row">
                  <span class="label">{{ t('restocking.quantity') }}:</span>
                  <span class="value quantity-value">{{ item.quantity_to_order }}</span>
                </div>
                <div class="order-row subtotal">
                  <span class="label">{{ t('restocking.subtotal') }}:</span>
                  <span class="value">{{ item.subtotal_formatted }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Action Section -->
      <div class="action-section" v-if="recommendations.length > 0">
        <button
          @click="submitOrder"
          :disabled="submitting || recommendations.length === 0"
          class="place-order-btn"
        >
          <span v-if="!submitting">{{ t('restocking.placeOrder') }}</span>
          <span v-else>{{ t('common.submitting') }}</span>
        </button>
        <div v-if="orderMessage" :class="['order-message', orderMessageType]">
          {{ orderMessage }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t } = useI18n()
    const budget = ref(5000)
    const loading = ref(true)
    const error = ref(null)
    const submitting = ref(false)
    const orderMessage = ref('')
    const orderMessageType = ref('success')

    const allForecasts = ref([])
    const inventoryItems = ref([])
    const recommendations = ref([])

    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    const remainingBudget = computed(() => {
      const total = recommendations.value.reduce((sum, item) => sum + item.subtotal, 0)
      return budget.value - total
    })

    const remainingBudgetFormatted = computed(() => {
      const remaining = remainingBudget.value
      return `$${remaining.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
    })

    const totalItemsCount = computed(() => {
      return recommendations.value.reduce((sum, item) => sum + item.quantity_to_order, 0)
    })

    const totalCostFormatted = computed(() => {
      const total = recommendations.value.reduce((sum, item) => sum + item.subtotal, 0)
      return `$${total.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
    })

    const generateRecommendations = () => {
      // Get inventory items that match current filters
      let validSkus = new Set()

      if (selectedLocation.value === 'all' && selectedCategory.value === 'all') {
        validSkus = new Set(inventoryItems.value.map(item => item.sku))
      } else {
        const filtered = inventoryItems.value.filter(item => {
          const locationMatch = selectedLocation.value === 'all' || item.warehouse === selectedLocation.value
          const categoryMatch = selectedCategory.value === 'all' || item.category.toLowerCase() === selectedCategory.value.toLowerCase()
          return locationMatch && categoryMatch
        })
        validSkus = new Set(filtered.map(item => item.sku))
      }

      // Filter and sort forecasts by demand
      const filtered = allForecasts.value
        .filter(f => validSkus.has(f.item_sku))
        .sort((a, b) => b.forecasted_demand - a.forecasted_demand)

      // Calculate quantities based on budget
      const items = []
      let remaining = budget.value

      for (const forecast of filtered) {
        if (remaining <= 0) break

        const unitCost = forecast.unit_cost || 100
        const quantity = Math.floor(remaining / unitCost)

        if (quantity > 0) {
          const subtotal = quantity * unitCost
          items.push({
            ...forecast,
            quantity_to_order: quantity,
            subtotal: subtotal,
            subtotal_formatted: `$${subtotal.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
          })
          remaining -= subtotal
        }
      }

      recommendations.value = items
    }

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null

        const [forecastsData, inventoryData] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory({
            warehouse: 'all',
            category: 'all'
          })
        ])

        allForecasts.value = forecastsData
        inventoryItems.value = inventoryData
        generateRecommendations()
      } catch (err) {
        error.value = 'Failed to load demand forecasts'
        console.error('Load error:', err)
      } finally {
        loading.value = false
      }
    }

    const submitOrder = async () => {
      if (recommendations.value.length === 0) return

      try {
        submitting.value = true
        orderMessage.value = ''

        const items = recommendations.value.map(item => ({
          sku: item.item_sku,
          name: item.item_name,
          quantity: item.quantity_to_order,
          unit_price: item.unit_cost || 100
        }))

        const totalValue = recommendations.value.reduce((sum, item) => sum + item.subtotal, 0)

        await api.createRestockingOrder(
          items,
          totalValue,
          selectedLocation.value === 'all' ? 'All Warehouses' : selectedLocation.value,
          selectedCategory.value === 'all' ? 'All Categories' : selectedCategory.value
        )

        orderMessage.value = t('restocking.placeOrderSuccess')
        orderMessageType.value = 'success'

        // Reset form
        budget.value = 5000
        recommendations.value = []

        // Reload data after short delay
        setTimeout(() => {
          loadData()
        }, 1500)
      } catch (err) {
        orderMessage.value = t('restocking.orderError')
        orderMessageType.value = 'error'
        console.error('Order error:', err)
      } finally {
        submitting.value = false
      }
    }

    onMounted(() => {
      loadData()
    })

    watch([selectedLocation, selectedCategory, budget], () => {
      generateRecommendations()
    })

    return {
      t,
      budget,
      loading,
      error,
      submitting,
      orderMessage,
      orderMessageType,
      recommendations,
      remainingBudgetFormatted,
      totalItemsCount,
      totalCostFormatted,
      submitOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 20px;
}

.page-header {
  margin-bottom: 30px;
}

.page-header h2 {
  font-size: 1.8em;
  color: #0f172a;
  margin-bottom: 8px;
}

.page-header p {
  color: #64748b;
  font-size: 0.95em;
}

.restocking-container {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

.card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.card-header {
  margin-bottom: 15px;
  border-bottom: 2px solid #f1f5f9;
  padding-bottom: 12px;
}

.card-title {
  color: #0f172a;
  font-size: 1.1em;
  margin: 0;
  font-weight: 600;
}

.budget-section {
  background: linear-gradient(135deg, #f8f9fa 0%, #f1f5f9 100%);
}

.budget-controls {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.slider-container {
  display: flex;
  align-items: center;
  gap: 15px;
}

.budget-slider {
  flex: 1;
  height: 8px;
  border-radius: 5px;
  background: #e2e8f0;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(59, 130, 246, 0.3);
}

.budget-slider::-moz-range-thumb {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 4px rgba(59, 130, 246, 0.3);
}

.budget-display {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.budget-value,
.budget-remaining {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 15px;
  background: white;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.budget-value .label,
.budget-remaining .label {
  color: #64748b;
  font-weight: 500;
  font-size: 0.9em;
}

.budget-value .amount,
.budget-remaining .amount {
  color: #3b82f6;
  font-weight: 700;
  font-size: 1.1em;
}

.recommendations-section {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 2px solid #f1f5f9;
}

.section-header h3 {
  color: #0f172a;
  font-size: 1.1em;
  margin: 0;
  font-weight: 600;
}

.summary-stats {
  display: flex;
  gap: 30px;
}

.stat {
  display: flex;
  gap: 8px;
  align-items: center;
}

.stat .label {
  color: #64748b;
  font-size: 0.9em;
  font-weight: 500;
}

.stat .value {
  color: #0f172a;
  font-weight: 700;
}

.loading {
  text-align: center;
  padding: 40px 20px;
  color: #64748b;
}

.error {
  background: #fee2e2;
  border: 1px solid #fecaca;
  color: #dc2626;
  padding: 15px;
  border-radius: 6px;
  text-align: center;
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: #94a3b8;
  font-size: 0.95em;
}

.recommendations-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 15px;
  margin-top: 15px;
}

.recommendation-card {
  background: #f8f9fa;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  padding: 15px;
  transition: all 0.2s ease;
}

.recommendation-card:hover {
  border-color: #3b82f6;
  box-shadow: 0 4px 6px rgba(59, 130, 246, 0.1);
}

.card-content {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.item-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.item-info {
  flex: 1;
}

.item-name {
  color: #0f172a;
  font-weight: 600;
  font-size: 0.95em;
}

.item-sku {
  color: #64748b;
  font-size: 0.8em;
  margin-top: 3px;
}

.demand-info,
.pricing-info,
.order-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 0.9em;
}

.demand-row,
.price-row,
.order-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.label {
  color: #64748b;
  font-weight: 500;
}

.value {
  color: #0f172a;
  font-weight: 600;
}

.value.highlighted {
  color: #3b82f6;
  font-weight: 700;
}

.quantity-value {
  background: #dbeafe;
  color: #1e40af;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 0.9em;
}

.order-row.subtotal {
  border-top: 1px solid #cbd5e1;
  padding-top: 8px;
  font-weight: 700;
  color: #3b82f6;
}

.action-section {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
}

.place-order-btn {
  padding: 12px 30px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 200px;
}

.place-order-btn:hover:not(:disabled) {
  background: #2563eb;
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
  transform: translateY(-1px);
}

.place-order-btn:disabled {
  background: #cbd5e1;
  cursor: not-allowed;
}

.order-message {
  padding: 12px 20px;
  border-radius: 6px;
  text-align: center;
  font-weight: 500;
  width: 100%;
  max-width: 400px;
}

.order-message.success {
  background: #dcfce7;
  color: #166534;
  border: 1px solid #bbf7d0;
}

.order-message.error {
  background: #fee2e2;
  color: #dc2626;
  border: 1px solid #fecaca;
}

@media (max-width: 768px) {
  .budget-display {
    grid-template-columns: 1fr;
  }

  .summary-stats {
    flex-direction: column;
    gap: 10px;
  }

  .recommendations-grid {
    grid-template-columns: 1fr;
  }

  .section-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
}
</style>
