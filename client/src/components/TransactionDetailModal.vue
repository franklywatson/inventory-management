<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && transaction" class="modal-overlay" @click="close">
        <div class="modal-container" @click.stop>
          <div class="modal-header">
            <h3 class="modal-title">Transaction #{{ transaction.id.toString().padStart(3, '0') }}</h3>
            <button class="close-button" type="button" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="modal-body">
            <div class="amount-summary">
              <div class="amount-label">Amount</div>
              <div class="amount-value">{{ currencySymbol }}{{ transaction.amount.toLocaleString() }}</div>
            </div>

            <div class="detail-list">
              <div class="detail-row">
                <span class="detail-key">Description</span>
                <span class="detail-value">{{ transaction.description }}</span>
              </div>
              <div class="detail-row">
                <span class="detail-key">Vendor</span>
                <span class="detail-value">{{ transaction.vendor }}</span>
              </div>
              <div class="detail-row">
                <span class="detail-key">Category</span>
                <span class="detail-value">{{ transaction.category || 'N/A' }}</span>
              </div>
              <div class="detail-row">
                <span class="detail-key">Date</span>
                <span class="detail-value">{{ formatDate(transaction.date) }}</span>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn-secondary" type="button" @click="close">Close</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'
import { useI18n } from '../composables/useI18n'

const { currentCurrency } = useI18n()

const currencySymbol = computed(() => {
  return currentCurrency.value === 'JPY' ? '¥' : '$'
})

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  transaction: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

const formatDate = (dateString) => {
  const date = new Date(dateString)
  if (isNaN(date.getTime())) return dateString
  return date.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

const close = () => {
  emit('close')
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 1rem;
}

.modal-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.15);
  min-width: 400px;
  max-width: 560px;
  width: 100%;
  max-height: 90vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.5rem;
  border-bottom: 1px solid #e2e8f0;
}

.modal-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.close-button {
  background: none;
  border: none;
  color: #64748b;
  cursor: pointer;
  padding: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  transition: all 0.15s ease;
}

.close-button:hover {
  background: #f1f5f9;
  color: #0f172a;
}

.modal-body {
  flex: 1;
  overflow-y: auto;
  padding: 2rem;
}

.amount-summary {
  padding: 1.5rem;
  border-radius: 10px;
  text-align: center;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  color: white;
  margin-bottom: 2rem;
}

.amount-label {
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  opacity: 0.9;
  margin-bottom: 0.5rem;
}

.amount-value {
  font-size: 2.25rem;
  font-weight: 700;
}

.detail-list {
  display: flex;
  flex-direction: column;
  gap: 0;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  overflow: hidden;
}

.detail-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.25rem;
  border-bottom: 1px solid #f1f5f9;
}

.detail-row:last-child {
  border-bottom: none;
}

.detail-key {
  font-size: 0.875rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  flex-shrink: 0;
  margin-right: 1rem;
}

.detail-value {
  font-size: 0.9375rem;
  font-weight: 500;
  color: #0f172a;
  text-align: right;
}

.modal-footer {
  padding: 1.5rem;
  border-top: 1px solid #e2e8f0;
  display: flex;
  justify-content: flex-end;
}

.btn-secondary {
  padding: 0.625rem 1.25rem;
  background: #f1f5f9;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-weight: 500;
  font-size: 0.875rem;
  color: #334155;
  cursor: pointer;
  transition: all 0.15s ease;
  font-family: inherit;
}

.btn-secondary:hover {
  background: #e2e8f0;
  border-color: #cbd5e1;
}

/* Modal transition animations */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-container,
.modal-leave-active .modal-container {
  transition: transform 0.2s ease;
}

.modal-enter-from .modal-container,
.modal-leave-to .modal-container {
  transform: scale(0.95);
}
</style>
