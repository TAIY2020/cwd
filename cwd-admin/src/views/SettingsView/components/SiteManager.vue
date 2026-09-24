<template>
  <div class="domain-settings">
    <div class="domain-settings-desc">
      配置后台可见的站点。左侧为后台下拉框中显示的站点，右侧为数据库中发现的所有站点。支持拖拽或点击按钮移动。
    </div>

    <div class="domain-transfer">
      <!-- Visible Domains (Left) -->
      <div class="transfer-panel">
        <div class="transfer-header">后台显示站点 ({{ visibleList.length }})</div>
        <div class="transfer-body" @dragover.prevent @drop="onDrop($event, 'visible')">
          <div v-if="visibleList.length === 0" class="transfer-empty">
            无站点 (默认显示全部)
          </div>
          <div
            v-for="domain in visibleList"
            :key="domain"
            class="transfer-item"
            draggable="true"
            @dragstart="onDragStart($event, domain, 'visible')"
            @dblclick="moveToHidden(domain)"
          >
            <span class="domain-text">{{ getSiteLabel(domain) }}</span>
            <button
              type="button"
              class="move-btn"
              title="移出"
              @click="moveToHidden(domain)"
            >
              <PhArrowRight :size="16" />
            </button>
          </div>
        </div>
      </div>

      <!-- Actions -->
      <div class="transfer-actions">
        <button
          type="button"
          class="action-btn"
          title="全部左移"
          @click="moveAllToVisible"
        >
          <PhCaretDoubleLeft />
        </button>
        <button
          type="button"
          class="action-btn"
          title="全部右移"
          @click="moveAllToHidden"
        >
          <PhCaretDoubleRight />
        </button>
      </div>

      <!-- Hidden/All Domains (Right) -->
      <div class="transfer-panel">
        <div class="transfer-header">其他站点 ({{ hiddenList.length }})</div>
        <div class="transfer-body" @dragover.prevent @drop="onDrop($event, 'hidden')">
          <div v-if="hiddenList.length === 0" class="transfer-empty">无更多站点</div>
          <div
            v-for="domain in hiddenList"
            :key="domain"
            class="transfer-item"
            draggable="true"
            @dragstart="onDragStart($event, domain, 'hidden')"
            @dblclick="moveToVisible(domain)"
          >
            <button
              type="button"
              class="move-btn"
              title="移入"
              @click="moveToVisible(domain)"
            >
              <PhArrowLeft :size="16" />
            </button>
            <span class="domain-text">{{ getSiteLabel(domain) }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="form-actions">
      <button class="card-button" :disabled="loading" @click="handleSave">
        {{ loading ? "保存中..." : "保存" }}
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import {
  fetchSiteList,
  fetchFeatureSettings,
  saveFeatureSettings,
} from "../../../api/admin";

const loading = ref(false);
const allDomains = ref<string[]>([]);
const visibleList = ref<string[]>([]);

const hiddenList = computed(() => {
  const visibleSet = new Set(visibleList.value);
  return allDomains.value.filter((d) => !visibleSet.has(d));
});

function getSiteLabel(value: string) {
  if (!value) {
    return "默认站点 (Default)";
  }
  return value;
}

async function loadData() {
  try {
    const [siteRes, settingsRes] = await Promise.all([
      fetchSiteList(),
      fetchFeatureSettings(),
    ]);

    allDomains.value = siteRes.sites || [];
    if (settingsRes.visibleDomains) {
      visibleList.value = settingsRes.visibleDomains;
    } else {
      visibleList.value = [];
    }
  } catch (e) {
    console.error(e);
  }
}

function moveToHidden(domain: string) {
  visibleList.value = visibleList.value.filter((d) => d !== domain);
}

function moveToVisible(domain: string) {
  if (!visibleList.value.includes(domain)) {
    visibleList.value.push(domain);
  }
}

function moveAllToVisible() {
  const current = new Set(visibleList.value);
  hiddenList.value.forEach((d) => current.add(d));
  visibleList.value = Array.from(current);
}

function moveAllToHidden() {
  visibleList.value = [];
}

function onDragStart(event: DragEvent, domain: string, source: "visible" | "hidden") {
  if (event.dataTransfer) {
    event.dataTransfer.setData("text/plain", JSON.stringify({ domain, source }));
    event.dataTransfer.effectAllowed = "move";
  }
}

function onDrop(event: DragEvent, target: "visible" | "hidden") {
  const data = event.dataTransfer?.getData("text/plain");
  if (data) {
    try {
      const { domain, source } = JSON.parse(data);
      if (source !== target) {
        if (target === "visible") {
          moveToVisible(domain);
        } else {
          moveToHidden(domain);
        }
      }
    } catch (e) {
      console.error(e);
    }
  }
}

async function handleSave() {
  loading.value = true;
  try {
    await saveFeatureSettings({
      visibleDomains: visibleList.value,
    });
    // 保存成功后刷新页面以应用更改（LayoutView 重新加载）
    window.location.reload();
  } catch (e) {
    alert("保存失败");
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  loadData();
});
</script>

<style lang="less" scoped>
.domain-settings {
  display: flex;
  flex-direction: column;
  gap: var(--admin-space-5);
}

.domain-settings-desc {
  color: var(--text-secondary);
  font-size: 13px;
  line-height: 1.6;
}

.domain-transfer {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto minmax(0, 1fr);
  gap: var(--admin-space-5);
  height: 400px;
}

.transfer-panel {
  display: flex;
  min-width: 0;
  height: 100%;
  min-height: 300px;
  flex-direction: column;
  overflow: hidden;
  background-color: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: var(--admin-radius-md);
}

.transfer-header {
  padding: var(--admin-space-3) var(--admin-space-4);
  color: var(--text-primary);
  background-color: var(--bg-sider);
  border-bottom: 1px solid var(--border-color);
  font-weight: 600;
}

.transfer-body {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: var(--admin-space-1);
  padding: var(--admin-space-2);
  overflow-y: auto;
  overscroll-behavior: contain;
}

.transfer-empty {
  margin-top: 40px;
  color: var(--text-secondary);
  font-size: 13px;
  text-align: center;
}

.transfer-item {
  display: flex;
  min-height: 40px;
  align-items: center;
  justify-content: space-between;
  gap: var(--admin-space-2);
  padding: var(--admin-space-2) var(--admin-space-3);
  color: var(--text-primary);
  background-color: var(--bg-sider);
  border: 1px solid transparent;
  border-radius: var(--admin-radius-sm);
  cursor: grab;
  user-select: none;
  transition:
    background-color var(--admin-transition),
    border-color var(--admin-transition);

  &:hover {
    background-color: var(--bg-hover);
    border-color: var(--border-color);
  }

  &:active {
    cursor: grabbing;
  }

  .domain-text {
    flex: 1;
    min-width: 0;
    overflow-wrap: anywhere;
    word-break: break-all;
  }
}

.move-btn {
  display: inline-flex;
  width: 28px;
  height: 28px;
  flex: 0 0 auto;
  align-items: center;
  justify-content: center;
  padding: 0;
  color: var(--text-secondary);
  background: transparent;
  border: none;
  border-radius: var(--admin-radius-sm);
  cursor: pointer;
  transition:
    color var(--admin-transition),
    background-color var(--admin-transition);

  &:hover {
    color: var(--primary-color);
    background-color: var(--bg-active);
  }
}

.transfer-actions {
  display: flex;
  flex-direction: column;
  gap: var(--admin-space-2);
  align-self: center;
}

.action-btn {
  display: inline-flex;
  width: 36px;
  height: 36px;
  align-items: center;
  justify-content: center;
  padding: 0;
  color: var(--text-primary);
  background-color: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: 999px;
  cursor: pointer;
  transition:
    color var(--admin-transition),
    border-color var(--admin-transition),
    background-color var(--admin-transition);

  &:hover {
    color: var(--primary-color);
    background-color: var(--bg-hover);
    border-color: var(--primary-color);
  }
}

.form-actions {
  display: flex;
  justify-content: flex-end;
}

@media (max-width: 768px) {
  .domain-transfer {
    grid-template-columns: minmax(0, 1fr);
    gap: var(--admin-space-3);
    height: auto;
  }

  .transfer-panel {
    height: auto;
    min-height: 240px;
  }

  .transfer-actions {
    flex-direction: row;
    justify-content: center;
    align-self: auto;
  }

  .form-actions .card-button {
    width: 100%;
  }
}
</style>
