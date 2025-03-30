<script setup lang="ts">
import '@/polyfills';

import { ref, computed, watch, onMounted, onBeforeUnmount, nextTick } from 'vue';
import { useRoute } from 'vue-router';
import LoadingView from '@/views/LoadingView.vue';
import BannerStack from '@/components/banners/BannerStack.vue';
import AskAssistantChat from '@/components/AskAssistant/AskAssistantChat.vue';
import Modals from '@/components/Modals.vue';
import Telemetry from '@/components/Telemetry.vue';
import AskAssistantFloatingButton from '@/components/AskAssistant/AskAssistantFloatingButton.vue';
import { loadLanguage } from '@/plugins/i18n';
import { APP_MODALS_ELEMENT_ID, HIRING_BANNER, VIEWS } from '@/constants';
import { useRootStore } from '@/stores/root.store';
import { useAssistantStore } from '@/stores/assistant.store';
import { useUIStore } from '@/stores/ui.store';
import { useUsersStore } from '@/stores/users.store';
import { useSettingsStore } from '@/stores/settings.store';
import { useHistoryHelper } from '@/composables/useHistoryHelper';
import { useStyles } from './composables/useStyles';

const route = useRoute();
const rootStore = useRootStore();
const assistantStore = useAssistantStore();
const uiStore = useUIStore();
const usersStore = useUsersStore();
const settingsStore = useSettingsStore();

const { setAppZIndexes } = useStyles();

// Initialize undo/redo
useHistoryHelper(route);

const loading = ref(true);
const defaultLocale = computed(() => rootStore.defaultLocale);
const isDemoMode = computed(() => route.name === VIEWS.DEMO);
const showAssistantButton = computed(() => assistantStore.canShowAssistantButtonsOnCanvas);
const hasContentFooter = ref(false);
const appGrid = ref<Element | null>(null);

const assistantSidebarWidth = computed(() => assistantStore.chatWidth);

onMounted(async () => {
	setAppZIndexes();
	logHiringBanner();
	loading.value = false;
	window.addEventListener('resize', updateGridWidth);
	await updateGridWidth();
});

onBeforeUnmount(() => {
	window.removeEventListener('resize', updateGridWidth);
});

const logHiringBanner = () => {
	if (settingsStore.isHiringBannerEnabled && !isDemoMode.value) {
		console.log(HIRING_BANNER);
	}
};

const updateGridWidth = async () => {
	await nextTick();
	if (appGrid.value) {
		const { width, height } = appGrid.value.getBoundingClientRect();
		uiStore.appGridDimensions = { width, height };
	}
};

// As assistant sidebar width changes, recalculate the total width regularly
watch(assistantSidebarWidth, async () => {
	await updateGridWidth();
});

watch(route, (r) => {
	hasContentFooter.value = r.matched.some(
		(matchedRoute) => matchedRoute.components?.footer !== undefined,
	);
});

watch(defaultLocale, (newLocale) => {
	void loadLanguage(newLocale);
});
</script>

<template>
	<LoadingView v-if="loading" />
	<div
		v-else
		id="n8n-app"
		:class="{
			[$style.container]: true,
			[$style.sidebarCollapsed]: uiStore.sidebarMenuCollapsed,
		}"
	>
		<!-- 🚀 Custom Header Start -->
		<div class="custom-header">
			🚀 Leading Edge AI – Workflow Editor
		</div>
		<!-- 🚀 Custom Header End -->
