<template>
  <div v-if="visible" class="modal-overlay">
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="comment-edit-modal-title"
    >
      <h3 id="comment-edit-modal-title" class="modal-title">
        {{ t("comments.editModal.title") }}
      </h3>
      <div v-if="form" class="modal-body">
        <div class="form-item">
          <label class="form-label" for="comment-edit-name">
            {{ t("comments.editModal.name") }}
          </label>
          <input
            id="comment-edit-name"
            v-model="form.name"
            class="form-input"
            type="text"
          />
        </div>
        <div class="form-item">
          <label class="form-label" for="comment-edit-email">
            {{ t("comments.editModal.email") }}
          </label>
          <input
            id="comment-edit-email"
            v-model="form.email"
            class="form-input"
            type="email"
          />
        </div>
        <div class="form-item">
          <label class="form-label" for="comment-edit-url">
            {{ t("comments.editModal.url") }}
          </label>
          <input
            id="comment-edit-url"
            v-model="form.url"
            class="form-input"
            type="text"
          />
        </div>
        <div class="form-item">
          <label class="form-label" for="comment-edit-post-slug">
            {{ t("comments.editModal.postSlug") }}
          </label>
          <input
            id="comment-edit-post-slug"
            v-model="form.postSlug"
            class="form-input"
            type="text"
          />
        </div>
        <div class="form-item">
          <label class="form-label" for="comment-edit-post-url">
            {{ t("comments.editModal.postUrl") }}
          </label>
          <input
            id="comment-edit-post-url"
            v-model="form.postUrl"
            class="form-input"
            type="text"
          />
        </div>
        <div class="form-item">
          <label class="form-label" for="comment-edit-content">
            {{ t("comments.editModal.content") }}
          </label>
          <textarea
            id="comment-edit-content"
            v-model="form.contentText"
            class="form-input"
            rows="4"
          ></textarea>
        </div>
        <div class="form-item">
          <label class="form-label" for="comment-edit-status">
            {{ t("comments.editModal.status") }}
          </label>
          <select id="comment-edit-status" v-model="form.status" class="form-input">
            <option value="approved">{{ t("comments.statusFilter.approved") }}</option>
            <option value="pending">{{ t("comments.statusFilter.pending") }}</option>
            <option value="rejected">{{ t("comments.statusFilter.rejected") }}</option>
          </select>
        </div>
        <div class="form-item">
          <label class="form-label" for="comment-edit-priority">
            {{ t("comments.editModal.priority") }}
          </label>
          <input
            id="comment-edit-priority"
            v-model.number="form.priority"
            class="form-input"
            type="number"
            min="1"
          />
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn secondary" type="button" @click="handleClose">
          {{ t("comments.editModal.cancel") }}
        </button>
        <button
          class="modal-btn primary"
          type="button"
          :disabled="saving"
          @click="handleSubmit"
        >
          <span v-if="saving">{{ t("comments.editModal.saving") }}</span>
          <span v-else>{{ t("comments.editModal.save") }}</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from "vue-i18n";

const { t } = useI18n();

interface EditForm {
  id: number;
  name: string;
  email: string;
  url: string;
  postSlug: string;
  postUrl: string;
  contentText: string;
  status: string;
  priority: number;
}

defineProps<{
  visible: boolean;
  form: EditForm | null;
  saving: boolean;
}>();

const emit = defineEmits<{
  (e: "close"): void;
  (e: "submit"): void;
}>();

function handleClose() {
  emit("close");
}

function handleSubmit() {
  emit("submit");
}
</script>

<style scoped lang="less">
.modal-overlay {
  position: fixed;
  inset: 0;
  display: flex;
  align-items: stretch;
  justify-content: flex-end;
  z-index: 2000;
  padding-left: var(--admin-space-4);
  background-color: var(--admin-overlay);
}

.modal {
  width: 100%;
  max-width: 600px;
  margin: 0;
  padding: var(--admin-space-5);
  overflow-y: auto;
  overscroll-behavior: contain;
  background-color: var(--bg-card);
  border-left: 1px solid var(--border-color);
  box-shadow: var(--admin-shadow-md);
  display: flex;
  flex-direction: column;
  gap: var(--admin-space-4);
  animation: drawer-in 0.2s ease-out;
}

.modal-title {
  margin: 0;
  color: var(--text-primary);
  font-size: 16px;
  font-weight: 600;
  line-height: 1.4;
}

.modal-body {
  display: flex;
  flex-direction: column;
  gap: var(--admin-space-3);
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: var(--admin-space-2);
  margin-top: var(--admin-space-1);
}

.modal-btn {
  display: inline-flex;
  min-height: var(--admin-control-height);
  align-items: center;
  justify-content: center;
  padding: 0 14px;
  border: 1px solid transparent;
  border-radius: var(--admin-radius-sm);
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition:
    background-color var(--admin-transition),
    border-color var(--admin-transition),
    color var(--admin-transition);
}

.modal-btn.primary {
  color: var(--text-inverse);
  background-color: var(--primary-color);
  border-color: var(--primary-color);
}

.modal-btn.primary:hover:not(:disabled) {
  background-color: var(--admin-primary-hover);
  border-color: var(--admin-primary-hover);
}

.modal-btn.secondary {
  color: var(--text-primary);
  background-color: var(--bg-card);
  border-color: var(--border-color);
}

.modal-btn.secondary:hover:not(:disabled) {
  background-color: var(--bg-hover);
  border-color: var(--border-hover);
}

.modal-btn:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

.form-item {
  display: flex;
  flex-direction: column;
  gap: var(--admin-space-1);
}

.form-label {
  color: var(--text-secondary);
  font-size: 13px;
  font-weight: 500;
}

.form-input {
  width: 100%;
  min-height: var(--admin-control-height);
  padding: 8px 10px;
  color: var(--text-primary);
  background-color: var(--bg-input);
  border: 1px solid var(--border-color);
  border-radius: var(--admin-radius-sm);
  font-size: 13px;
  outline: none;
  transition:
    border-color var(--admin-transition),
    box-shadow var(--admin-transition);
}

.form-input:hover {
  border-color: var(--border-hover);
}

.form-input:focus {
  border-color: var(--primary-color);
  box-shadow: var(--admin-focus-ring);
}

textarea.form-input {
  min-height: 96px;
  resize: vertical;
}

@media (max-width: 640px) {
  .modal-overlay {
    padding-left: 0;
  }

  .modal {
    max-width: none;
    padding: 20px 16px 24px;
    border-left: none;
  }

  .modal-actions {
    position: sticky;
    bottom: 0;
    padding-top: var(--admin-space-3);
    background-color: var(--bg-card);
  }

  .modal-actions .modal-btn {
    min-height: var(--admin-control-height-lg);
    flex: 1;
  }
}

@keyframes drawer-in {
  from {
    opacity: 0;
    transform: translateX(24px);
  }

  to {
    opacity: 1;
    transform: translateX(0);
  }
}
</style>
