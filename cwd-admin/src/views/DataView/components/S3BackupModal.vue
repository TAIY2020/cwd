<template>
  <div
    v-if="visible"
    class="modal-overlay s3-backup-overlay"
    @click.self="handleClose"
  >
    <div class="modal s3-backup-modal">
      <div class="modal-header">
        <h3 class="modal-title">{{ t("data.sections.s3.backupListTitle") }}</h3>
        <button
          type="button"
          class="modal-close"
          :aria-label="t('data.sections.s3.close')"
          @click="handleClose"
        >
          <PhX :size="18" />
        </button>
      </div>
      <div class="modal-content">
        <div v-if="loading" class="loading">
          <div class="loading-spinner"></div>
          <div class="loading-text">{{ t("data.sections.s3.loadingBackups") }}</div>
        </div>
        <div v-else-if="backups.length === 0" class="empty-backup-list">
          {{ t("data.sections.s3.emptyBackupList") }}
        </div>
        <div v-else class="backup-list">
          <div v-for="item in backups" :key="item.key" class="backup-item">
            <div class="backup-info">
              <div class="backup-name" :title="item.key">{{ item.key }}</div>
              <div class="backup-meta">
                <span class="backup-size">{{ formatFileSize(item.size) }}</span>
                <span class="backup-date">{{ new Date(item.lastModified).toLocaleString() }}</span>
              </div>
            </div>
            <div class="backup-actions">
              <button
                type="button"
                class="backup-btn download"
                :aria-label="t('data.sections.s3.download')"
                @click="handleDownload(item.key)"
                :title="t('data.sections.s3.download')"
              >
                <PhDownloadSimple :size="16" />
              </button>
              <button
                type="button"
                class="backup-btn delete"
                :aria-label="t('data.sections.s3.delete')"
                @click="handleDelete(item.key)"
                :disabled="deletingKey === item.key"
                :title="t('data.sections.s3.delete')"
              >
                <PhTrash v-if="deletingKey !== item.key" :size="16" />
                <span v-else class="loading-spinner small"></span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from "vue";
import { useI18n } from "vue-i18n";
import {
  fetchS3BackupList,
  deleteS3Backup,
  S3BackupItem,
} from "../../../api/admin";
import { getApiBaseUrl } from "../../../api/http";

const props = defineProps<{
  visible: boolean;
  onClose?: () => void;
}>();

const emit = defineEmits<{
  close: [];
}>();

const { t } = useI18n();

const loading = ref(false);
const backups = ref<S3BackupItem[]>([]);
const deletingKey = ref<string | null>(null);

const handleClose = () => {
  if (loading.value || deletingKey.value) {
    return;
  }
  emit("close");
};

const handleDownload = async (key: string) => {
  try {
    const apiBaseUrl = getApiBaseUrl();
    const token = localStorage.getItem('cwd_admin_token');
    const url = `${apiBaseUrl}/admin/backup/s3/download?key=${encodeURIComponent(key)}`;

    const res = await fetch(url, {
      method: 'GET',
      headers: token ? { 'Authorization': `Bearer ${token}` } : {},
    });

    if (!res.ok) {
      throw new Error(`下载失败: ${res.status} ${res.statusText}`);
    }

    const blob = await res.blob();
    const fileName = key;

    const a = document.createElement('a');
    a.href = window.URL.createObjectURL(blob);
    a.download = fileName;
    document.body.appendChild(a);
    a.click();
    window.URL.revokeObjectURL(a.href);
    document.body.removeChild(a);
  } catch (e: any) {
    console.error('Download error:', e);
  }
};

const handleDelete = async (key: string) => {
  if (!confirm(t("data.sections.s3.confirmDelete", { file: key }))) {
    return;
  }

  deletingKey.value = key;
  try {
    await deleteS3Backup(key);
    backups.value = backups.value.filter((item) => item.key !== key);
  } catch (e: any) {
    console.error("Delete error:", e);
  } finally {
    deletingKey.value = null;
  }
};

const fetchBackups = async () => {
  if (!props.visible) return;

  loading.value = true;
  try {
    const res = await fetchS3BackupList();
    backups.value = res.files;
  } catch (e: any) {
    console.error("Fetch backups error:", e);
  } finally {
    loading.value = false;
  }
};

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return "0 B";
  const k = 1024;
  const sizes = ["B", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i];
};

watch(
  () => props.visible,
  (newVal) => {
    if (newVal) {
      fetchBackups();
    } else {
      backups.value = [];
    }
  }
);
</script>

<style scoped lang="less">
.modal-overlay.s3-backup-overlay {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--admin-space-4);
  background-color: var(--admin-overlay);
}

.s3-backup-modal {
  display: flex;
  width: min(600px, 100%);
  max-height: calc(100dvh - 32px);
  flex-direction: column;
  gap: var(--admin-space-4);
  padding: var(--admin-space-5);
  overflow: hidden;
  background-color: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: var(--admin-radius-md);
  box-shadow: var(--admin-shadow-md);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--admin-space-3);
  padding-bottom: var(--admin-space-3);
  border-bottom: 1px solid var(--border-color);
}

.modal-title {
  margin: 0;
  color: var(--text-primary);
  font-size: 16px;
  font-weight: 600;
  line-height: 1.4;
}

.modal-close {
  display: inline-flex;
  width: 34px;
  height: 34px;
  flex: 0 0 auto;
  align-items: center;
  justify-content: center;
  padding: 0;
  color: var(--text-secondary);
  background: transparent;
  border: 1px solid transparent;
  border-radius: var(--admin-radius-sm);
  cursor: pointer;
  transition:
    color var(--admin-transition),
    background-color var(--admin-transition),
    border-color var(--admin-transition);
}

.modal-close:hover {
  color: var(--text-primary);
  background-color: var(--bg-hover);
  border-color: var(--border-color);
}

.modal-content {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  overscroll-behavior: contain;
}

.loading {
  display: flex;
  min-height: 180px;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: var(--admin-space-2);
  padding: var(--admin-space-8);
}

.loading-spinner {
  width: 24px;
  height: 24px;
  border: 2px solid var(--border-color);
  border-top-color: var(--primary-color);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.loading-spinner.small {
  width: 14px;
  height: 14px;
}

.loading-text {
  color: var(--text-secondary);
  font-size: 13px;
}

.empty-backup-list {
  padding: var(--admin-space-8) var(--admin-space-4);
  color: var(--text-secondary);
  font-size: 13px;
  text-align: center;
}

.backup-list {
  display: flex;
  flex-direction: column;
  gap: var(--admin-space-2);
}

.backup-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: var(--admin-space-3);
  padding: var(--admin-space-3);
  background-color: var(--bg-sider);
  border: 1px solid var(--border-color);
  border-radius: var(--admin-radius-sm);
}

.backup-info {
  flex: 1;
  min-width: 0;
}

.backup-name {
  margin-bottom: var(--admin-space-1);
  overflow: hidden;
  color: var(--text-primary);
  font-size: 13px;
  font-weight: 500;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.backup-meta {
  display: flex;
  flex-wrap: wrap;
  gap: var(--admin-space-2);
  align-items: center;
  font-size: 12px;
  color: var(--text-secondary);
}

.backup-size {
  padding: 2px 8px;
  background-color: var(--bg-hover);
  border-radius: var(--admin-radius-sm);
}

.backup-actions {
  display: flex;
  gap: var(--admin-space-1);
  flex-shrink: 0;
}

.backup-btn {
  display: inline-flex;
  width: 34px;
  height: 34px;
  align-items: center;
  justify-content: center;
  padding: 0;
  color: var(--text-secondary);
  background-color: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: var(--admin-radius-sm);
  cursor: pointer;
  transition:
    color var(--admin-transition),
    background-color var(--admin-transition),
    border-color var(--admin-transition);
}

.backup-btn:hover:not(:disabled) {
  color: var(--text-primary);
  background-color: var(--bg-hover);
  border-color: var(--border-hover);
}

.backup-btn.download:hover {
  color: var(--primary-color);
}

.backup-btn.delete:hover {
  color: var(--color-danger);
  background-color: var(--admin-danger-soft);
  border-color: var(--color-danger);
}

.backup-btn:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

@media (max-width: 640px) {
  .modal-overlay.s3-backup-overlay {
    align-items: flex-end;
    padding: 0;
  }

  .s3-backup-modal {
    width: 100%;
    max-height: 88dvh;
    padding: 20px 16px 24px;
    border-right: none;
    border-bottom: none;
    border-left: none;
    border-radius: var(--admin-radius-md) var(--admin-radius-md) 0 0;
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
