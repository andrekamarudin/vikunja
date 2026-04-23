<template>
	<div class="content has-text-centered">
		<h1 v-if="salutation">
			{{ salutation }}
		</h1>

		<Message
			v-if="deletionScheduledAt !== null"
			variant="danger"
			class="mbe-4"
		>
			{{
				$t('user.deletion.scheduled', {
					date: formatDisplayDate(deletionScheduledAt),
					dateSince: formatDateSince(deletionScheduledAt),
				})
			}}
			<RouterLink :to="{name: 'user.settings.deletion'}">
				{{ $t('user.deletion.scheduledCancel') }}
			</RouterLink>
		</Message>
		<AddTask
			class="is-max-width-desktop"
			@taskAdded="updateTaskKey"
		/>
		<div class="home-shortcuts is-max-width-desktop">
			<BaseButton
				href="/llm"
				:open-external-in-new-tab="false"
				class="wrapper-link"
			>
				<span>🤖 {{ $t('home.openAssistant') }}</span>
			</BaseButton>
			<BaseButton
				href="/calendar"
				:open-external-in-new-tab="false"
				class="wrapper-link"
			>
				<span>🗓️ {{ $t('home.openCalendarView') }}</span>
			</BaseButton>
			<BaseButton
				href="/ical"
				:open-external-in-new-tab="false"
				class="wrapper-link"
			>
				<span>🗓️ {{ $t('home.openCalendarFeed') }}</span>
			</BaseButton>
			<BaseButton
				href="/gantt"
				:open-external-in-new-tab="false"
				class="wrapper-link"
			>
				<span>📊 {{ $t('home.openGanttView') }}</span>
			</BaseButton>
			<BaseButton
				href="/feedback"
				:open-external-in-new-tab="false"
				class="wrapper-link"
			>
				<span>💬 {{ $t('home.openFeedback') }}</span>
			</BaseButton>
		</div>
		<ImportHint v-if="tasksLoaded" />
		<div
			v-if="authStore.settings.frontendSettings.showLastViewed !== false && projectHistory.length > 0"
			class="is-max-width-desktop has-text-start mbs-4"
		>
			<h3>{{ $t('home.lastViewed') }}</h3>
			<ProjectCardGrid
				v-cy="'projectCardGrid'"
				:projects="projectHistory"
				:show-even-number-of-projects="true"
			/>
		</div>
		<ShowTasks
			v-if="projectStore.hasProjects"
			:key="showTasksKey"
			:label-ids="labelIds"
			class="show-tasks"
			@tasksLoaded="tasksLoaded = true"
			@clearLabelFilter="handleClearLabelFilter"
		/>
	</div>
</template>

<script lang="ts" setup>
import {ref, computed} from 'vue'
import {useRoute, useRouter} from 'vue-router'

import Message from '@/components/misc/Message.vue'
import ShowTasks from '@/views/tasks/ShowTasks.vue'
import ProjectCardGrid from '@/components/project/partials/ProjectCardGrid.vue'
import AddTask from '@/components/tasks/AddTask.vue'
import ImportHint from '@/components/home/ImportHint.vue'
import BaseButton from '@/components/base/BaseButton.vue'

import {getHistory} from '@/modules/projectHistory'
import {parseDateOrNull} from '@/helpers/parseDateOrNull'
import {formatDateSince, formatDisplayDate} from '@/helpers/time/formatDate'
import {useDaytimeSalutation} from '@/composables/useDaytimeSalutation'

import {useProjectStore} from '@/stores/projects'
import {useAuthStore} from '@/stores/auth'

const salutation = useDaytimeSalutation()

const authStore = useAuthStore()
const projectStore = useProjectStore()
const route = useRoute()
const router = useRouter()

const projectHistory = computed(() => {
	// If we don't check this, it tries to load the project background right after logging out	
	if(!authStore.authenticated) {
		return []
	}
	
	return getHistory()
		.map(l => projectStore.projects[l.id])
		.filter(l => Boolean(l))
})

const tasksLoaded = ref(false)

const deletionScheduledAt = computed(() => parseDateOrNull(authStore.info?.deletionScheduledAt))

// Extract label IDs from query parameter
const labelIds = computed(() => {
	const labelsParam = route.query.labels
	if (!labelsParam) {
		return undefined
	}
	return Array.isArray(labelsParam) ? labelsParam : [labelsParam]
})

// This is to reload the tasks list after adding a new task through the global task add.
// FIXME: Should use pinia (somehow?)
const showTasksKey = ref(0)

function updateTaskKey() {
	showTasksKey.value++
}

function handleClearLabelFilter() {
	const query = {...route.query}
	delete query.labels
	router.push({
		name: route.name as string,
		query,
	})
}
</script>

<style scoped lang="scss">
.home-shortcuts {
	display: inline-flex;
	gap: .5rem;
	justify-content: center;
	margin-block-start: 1rem;
	flex-wrap: wrap;
	padding: .5rem;
	border-radius: $radius;
	background: var(--white);
	box-shadow: var(--shadow-sm);

	.wrapper-link {
		display: inline-flex;
		align-items: center;
		gap: .5rem;
		border-radius: $radius;
		padding: .4rem .7rem;
		font-size: .85rem;
		color: var(--text);
		text-decoration: none;
		transition: all 100ms;

		&:hover {
			color: var(--switch-view-color);
			background: var(--primary);
		}
	}
}

.show-tasks {
	margin-block-start: 2rem;
}
</style>
