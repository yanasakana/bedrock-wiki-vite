<template>
	<NavBar />
	<Sidebar />

	<main
		:class="{
			'm-8': true,
			'md:ml-80': isVisible,
			'mt-0': true,
			'min-h-screen': true,
		}"
	>
		<h1 v-if="page.title != ''" id="top">{{ page.title }}</h1>
		<TOC v-if="showToc" />
		<Content
			:class="{
				'toc-visible': !isReducedScreen && showToc,
			}"
		/>
		<div v-if="showContributors">
			<h2>Contributors</h2>
			<Suspense>
				<template #default>
					<Contributors />
				</template>
				<template #fallback>
					<div>
						<span> Loading contributors... </span>
					</div>
				</template>
			</Suspense>
		</div>
	</main>
</template>

<script setup lang="ts">
import { computed } from 'vue'
const route = useRoute()
import TOC from './Content/TOC.vue'
import Sidebar from './Sidebar/Sidebar.vue'
import NavBar from './Navigation/NavBar.vue'
import { useSidebarState } from '../Composables/sidebar'
import Contributors from './Content/Contributors.vue'
const { isVisible } = useSidebarState()
import { useIsReducedScreen } from '../Composables/isReducedScreen'
const { isReducedScreen } = useIsReducedScreen()
import { usePageData, useRoute } from 'vitepress'
const page = usePageData()

// Default toc to true
const showToc = computed(() =>
	route.data.frontmatter.show_toc == null
		? true
		: !!route.data.frontmatter.show_toc
)

const showContributors = computed(() =>
	route.data.frontmatter.show_contributors == null
		? true
		: !!route.data.frontmatter.show_contributors
)
</script>

<style scoped>
.toc-visible {
	padding-right: 300px;
}
details summary::-webkit-details-marker {
	display: none;
}
</style>
