<template>
  <div class="docs-custom-nav-list" :class="bgColor">
    <template
      v-for="(item, key) in finalStructure"
      :key="key"
    >
      <DocsRecursiveNavItem
        :nav-items="item"
        @nav-click="handleNavClick"
      />
    </template>
  </div>
</template>

<script setup lang="ts">
import { computed, inject } from 'vue';
import { useRouter } from 'vue-router';
import DocsRecursiveNavItem from './DocsRecursiveNavItem.vue';

interface ComponentItem {
  type: 'component';
  label: string;
  relativePath: string;
  exampleComponent: string;
}

interface DirectoryItem {
  type: 'directory';
  label: string;
  relativePath: string;
  children: Record<string, ComponentItem | DirectoryItem>;
}

type NavigationItem = ComponentItem | DirectoryItem;

interface Props {
  filterText?: string;
  onNavClick?: ((arg: ComponentItem) => void) | null;
  bgColor?: string;
}

const props = withDefaults(defineProps<Props>(), {
  filterText: '',
  onNavClick: null,
  bgColor: 'background'
});

interface ComponentDocPlugin {
  convertPathToExampleName: (path: string) => string;
}

const componentDocPlugin = inject('componentDocPlugin') as ComponentDocPlugin;
const router = useRouter();

const directoryStructure = computed<Record<string, NavigationItem>>(() => {
  return Object.keys(import.meta.glob('@/components/**/*.vue')).reduce<Record<string, NavigationItem>>((accumulator, filePath) => {
    const relativePath = filePath.split('components/').slice(1).join('');
    const exampleComponent = componentDocPlugin.convertPathToExampleName(relativePath);
    const pathSegments = relativePath.split('/');
    let lastRef = accumulator;
    pathSegments.forEach((pathSegment) => {
      if (pathSegment.endsWith('.vue')) {
        lastRef[pathSegment] = { type: 'component', label: pathSegment, relativePath, exampleComponent };
      } else if (!lastRef[pathSegment]) {
        lastRef[pathSegment] = { type: 'directory', label: pathSegment, relativePath, children: {} };
        lastRef = (lastRef[pathSegment] as DirectoryItem).children;
      } else if (lastRef[pathSegment].type === 'directory') {
        lastRef = (lastRef[pathSegment] as DirectoryItem).children;
      }
    });
    return accumulator;
  }, {});
});

function filterNestedStructure(
  structure: Record<string, NavigationItem>,
  filterText: string
): Record<string, NavigationItem> {
  return Object.entries(structure).reduce<Record<string, NavigationItem>>((accumulator, [key, value]) => {
    if (value.type === 'directory' && Object.keys(value.children).length > 0) {
      const filteredChildren = filterNestedStructure(value.children, filterText);

      if (Object.keys(filteredChildren).length > 0) {
        accumulator[key] = { ...value, children: filteredChildren };
      }
    } else if (value.type === 'component' && value.label.toLowerCase().includes(filterText.toLowerCase())) {
      accumulator[key] = value;
    }

    return accumulator;
  }, {});
}

const finalStructure = computed<Record<string, NavigationItem>>(() => {
  return filterNestedStructure(directoryStructure.value, props.filterText);
});

// Add these interfaces to match RecursiveNavItem.vue
interface ComponentNavItem {
  type: 'component';
  label: string;
  relativePath?: string;
  exampleComponent?: string;
}

// Forward declaration of NavItem
type NavItem = ComponentNavItem | DirectoryNavItem;

interface DirectoryNavItem {
  type: 'directory';
  label: string;
  relativePath?: string;
  children: Record<string, NavItem>;
}

function handleNavClick(arg: NavItem): void {
  // Only process component items for navigation
  if (arg.type !== 'component') return;

  if (props.onNavClick) {
    // Make sure relativePath and exampleComponent are defined before passing to onNavClick
    if (arg.relativePath && arg.exampleComponent) {
      props.onNavClick({
        type: 'component',
        label: arg.label,
        relativePath: arg.relativePath,
        exampleComponent: arg.exampleComponent
      });
    }
    return;
  }

  // Make sure relativePath and exampleComponent are defined before using them
  if (arg.relativePath && arg.exampleComponent) {
    router.push({
      name: 'componentDoc' as any,
      params: { componentName: arg.exampleComponent },
      query: { relativePath: arg.relativePath }
    });
  }
}
</script>

<style lang="scss" scoped>
.docs-custom-nav-list {
  padding: 8px;
  border-radius: 4px;
  background-color: white;

  &.background {
    background-color: #f5f5f5;
  }
}
</style>