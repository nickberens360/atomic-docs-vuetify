<template>
  <header class="docs-app-bar">
    <div class="docs-app-bar-content">
      <div class="docs-app-bar-title">
        <button 
          class="docs-nav-icon-button"
          @click="toggleDrawer"
        >
          <span class="docs-nav-icon">☰</span>
        </button>
        <span class="docs-title-text">Welcome</span>
      </div>

      <div class="docs-app-bar-actions">
        <div
          v-if="!isComponentDocsRoute"
          class="docs-search-container"
        >
          <div class="docs-text-field">
            <div class="docs-input-wrapper">
              <span class="docs-prepend-icon">🔍</span>
              <input
                v-model="filterText"
                name="filter-list"
                placeholder="Search Components"
                class="docs-input"
                autocomplete="one-time-code"
              >
              <span 
                v-if="filterText" 
                class="docs-append-icon" 
                @click="filterText = ''"
              >
                ✕
              </span>
            </div>
          </div>

          <div class="docs-menu-container">
            <div 
              v-show="isMenuOpen"
              class="docs-menu"
              @mouseleave="isMenuOpen = false"
            >
              <DocsComponentNavigation
                :filter-text="filterText"
                :on-nav-click="handleNavClick"
                bg-color="surface"
              />
            </div>
          </div>
        </div>

        <div class="docs-theme-toggle">
          <span class="docs-theme-icon">
            {{ appStore.theme.isDark ? '🌙' : '☀️' }}
          </span>
          <label class="docs-switch">
            <input 
              type="checkbox"
              :checked="appStore.theme.isDark"
              @change="toggleTheme(($event.target as HTMLInputElement).checked)"
            >
            <span class="docs-slider" />
          </label>
        </div>
      </div>
    </div>
  </header>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useAppStore } from '@/stores/app'
import { useRouter, useRoute } from "vue-router";
import DocsComponentNavigation from "./DocsComponentNavigation.vue";

// Import the ComponentItem interface from the types used in ComponentNavigation
interface ComponentItem {
  type: 'component';
  label: string;
  relativePath: string;
  exampleComponent: string;
}

const router = useRouter();
const route = useRoute();
const filterText = ref('');
const isMenuOpen = ref(false);

// Computed property to check if the current route is 'componentDocs'
const isComponentDocsRoute = computed(() => {
  return (route.name as any) === 'componentDocs';
});

function handleNavClick(arg: ComponentItem): void {
  router.push({
    name: 'componentDoc' as any,
    params: { componentName: arg.exampleComponent },
    query: { relativePath: arg.relativePath }
  });
  isMenuOpen.value = false;
}

const appStore = useAppStore()

// Function to toggle the drawer and rail
function toggleDrawer() {
  appStore.isAppRailOpen = !appStore.isAppRailOpen
  appStore.isAppNavDrawerOpen = !appStore.isAppNavDrawerOpen
}

// Function to toggle the theme
function toggleTheme(value: boolean | null) {
  // If value is null, default to false or keep the current value
  const isDark = value !== null ? value : false;

  // Update the theme in the store
  appStore.theme.isDark = isDark

  // Apply theme class to document body
  if (isDark) {
    document.body.classList.add('docs-app-theme--dark')
    document.body.classList.remove('docs-app-theme--light')
  } else {
    document.body.classList.add('docs-app-theme--light')
    document.body.classList.remove('docs-app-theme--dark')
  }
}

// Add event listeners for menu
const handleMouseEnter = () => {
  isMenuOpen.value = true;
};

const handleMouseLeave = () => {
  isMenuOpen.value = false;
};

// Add event listeners when component is mounted
onMounted(() => {
  const searchContainer = document.querySelector('.docs-search-container');
  if (searchContainer) {
    searchContainer.addEventListener('mouseenter', handleMouseEnter);
    searchContainer.addEventListener('mouseleave', handleMouseLeave);
  }

  // Initialize theme class based on current theme setting
  if (appStore.theme.isDark) {
    document.body.classList.add('docs-app-theme--dark');
    document.body.classList.remove('docs-app-theme--light');
  } else {
    document.body.classList.add('docs-app-theme--light');
    document.body.classList.remove('docs-app-theme--dark');
  }
});

// Remove event listeners when component is unmounted
onUnmounted(() => {
  const searchContainer = document.querySelector('.docs-search-container');
  if (searchContainer) {
    searchContainer.removeEventListener('mouseenter', handleMouseEnter);
    searchContainer.removeEventListener('mouseleave', handleMouseLeave);
  }
});
</script>

<style scoped lang="scss">
.docs-app-bar {
  position: fixed; /* Fixed positioning on mobile */
  top: 0;
  left: 0;
  right: 0;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  height: 64px;
  width: 100%;
  background-color: var(--docs-background-color, #f5f5f5);
  box-shadow: none;
  padding: 0 16px;
  z-index: 100;
}

.docs-app-bar-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.docs-app-bar-title {
  display: flex;
  align-items: center;
}

.docs-nav-icon-button {
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  margin-right: 8px;
  border-radius: 4px;

  &:hover {
    background-color: rgba(0, 0, 0, 0.04);
  }
}

.docs-nav-icon {
  font-size: 24px;
}

.docs-title-text {
  font-size: 20px;
  font-weight: 500;
}

.docs-app-bar-actions {
  display: flex;
  align-items: center;
}

.docs-search-container {
  position: relative;
  margin-right: 16px;
}

.docs-text-field {
  width: 300px;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.docs-input-wrapper {
  display: flex;
  align-items: center;
  padding: 8px 12px;
}

.docs-prepend-icon, .docs-append-icon {
  font-size: 18px;
  color: rgba(0, 0, 0, 0.54);
}

.docs-append-icon {
  cursor: pointer;
}

.docs-input {
  flex: 1;
  border: none;
  outline: none;
  padding: 0 8px;
  font-size: 14px;
}

.docs-menu-container {
  position: relative;
}

.docs-menu {
  position: absolute;
  top: 100%;
  left: 0;
  width: 300px;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  z-index: 100;
  margin-top: 4px;
}

.docs-theme-toggle {
  display: flex;
  align-items: center;
  margin-right: 16px;
}

.docs-theme-icon {
  margin-right: 8px;
  font-size: 20px;
}

/* Custom switch styling */
.docs-switch {
  position: relative;
  display: inline-block;
  width: 40px;
  height: 20px;
}

.docs-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.docs-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 20px;
}

.docs-slider:before {
  position: absolute;
  content: "";
  height: 16px;
  width: 16px;
  left: 2px;
  bottom: 2px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .docs-slider {
  background-color: #2196F3;
}

input:checked + .docs-slider:before {
  transform: translateX(20px);
}

</style>
