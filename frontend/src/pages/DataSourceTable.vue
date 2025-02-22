<template>
	<BasePage>
		<template #header>
			<div v-if="dataSourceTable.doc" class="flex flex-1 items-center space-x-4">
				<div class="flex items-start space-x-4">
					<h1 class="text-3xl font-medium text-gray-900">
						{{ dataSourceTable.doc.label }}
					</h1>
					<div class="flex h-8 items-center">
						<Badge :color="hidden ? 'yellow' : 'green'" class="h-fit">
							{{ hidden ? 'Disabled' : 'Enabled' }}
						</Badge>
					</div>
				</div>
				<div class="space-x-2">
					<Dropdown
						placement="left"
						:button="{
							icon: 'more-horizontal',
							appearance: 'minimal',
						}"
						:options="[
							{
								label: hidden ? 'Enable' : 'Disable',
								icon: hidden ? 'eye' : 'eye-off',
								handler: () => (hidden = !hidden),
							},
							{
								label: 'Add Link',
								icon: 'link',
								handler: () => (addLinkDialog = true),
							},
						]"
					/>
				</div>
			</div>
		</template>
		<template #main>
			<div v-if="dataSourceTable.columns" class="flex h-full w-full flex-col pt-1">
				<div class="flex h-6 space-x-1 text-sm font-light text-gray-600">
					{{ dataSourceTable.columns.length }} Columns -
					{{ dataSourceTable.no_of_rows }} Rows
				</div>
				<div class="h-[calc(100%-1.5rem)] w-full">
					<Table
						:columns="
							dataSourceTable.columns.map((c) => `${c[0]} (${formatType(c[1])})`)
						"
						:rows="dataSourceTable.rows"
					/>
				</div>
			</div>
		</template>
	</BasePage>

	<Dialog :options="{ title: 'Create a Link' }" v-model="addLinkDialog">
		<template #body-content>
			<div class="space-y-4">
				<div>
					<div class="mb-2 block text-sm leading-4 text-gray-700">Table</div>
					<Autocomplete
						v-model="newLink.table"
						:options="tableOptions"
						placeholder="Select a table..."
					/>
				</div>
				<Input
					type="select"
					v-model="newLink.primaryKey"
					:options="dataSourceTable.columns.map((c) => c[0])"
					:label="`Select a column from ${dataSourceTable.doc.label}`"
				/>
				<Input
					v-if="newLink.table?.value"
					type="select"
					v-model="newLink.foreignKey"
					:options="foreignKeyOptions"
					:label="`Select a column from ${newLink.table.label}`"
				/>
			</div>
		</template>
		<template #actions>
			<Button
				appearance="primary"
				@click="createLink"
				:loading="creatingLink"
				:disabled="createLinkDisabled"
			>
				Create
			</Button>
		</template>
	</Dialog>
</template>

<script setup>
import BasePage from '@/components/BasePage.vue'
import Table from '@/components/Query/Visualize/Table.vue'
import Autocomplete from '@/components/Controls/Autocomplete.vue'
import { useDataSourceTable } from '@/utils/datasource'
import { Dropdown, Badge, createResource } from 'frappe-ui'
import { computed, ref, reactive, watch, inject } from 'vue'

const props = defineProps({
	name: {
		type: String,
		required: true,
	},
	table: {
		type: String,
		required: true,
	},
})

const MAX_COLS = 8
const addLinkDialog = ref(false)
const newLink = reactive({
	table: '',
	primaryKey: '',
	foreignKey: '',
})

const dataSourceTable = useDataSourceTable(props.name, props.table)
const hidden = computed({
	get() {
		return dataSourceTable.doc.hidden
	},
	set(value) {
		if (value !== dataSourceTable.doc.hidden) {
			dataSourceTable.doc.hidden = value
			dataSourceTable.updateVisibility(value)
		}
	},
})

const getTableOptions = createResource({
	method: 'insights.api.get_tables',
	initialData: [],
})
const tableOptions = computed(() =>
	getTableOptions.data.map((table) => ({
		...table,
		value: table.table,
	}))
)
getTableOptions.submit({ data_source: props.name })

const getForeignKeyOptions = createResource({
	method: 'insights.api.get_data_source_table',
	initialData: [],
})
watch(
	() => newLink.table,
	(table) => {
		if (table) {
			getForeignKeyOptions.submit({
				name: props.name,
				table: table.table,
			})
		} else {
			foreignKeyOptions.value = []
		}
	}
)
const foreignKeyOptions = computed({
	get() {
		return getForeignKeyOptions.data?.columns?.map((c) => c[0]) || []
	},
	set(value) {
		getForeignKeyOptions.data = value
	},
})

const createLinkDisabled = computed(() => {
	return !newLink.table || !newLink.primaryKey || !newLink.foreignKey
})

const $notify = inject('$notify')
const createLinkResource = createResource({
	method: 'insights.api.create_table_link',
	onSuccess() {
		newLink.table = ''
		newLink.primaryKey = ''
		newLink.foreignKey = ''
		addLinkDialog.value = false
		$notify({
			appearance: 'success',
			title: 'Success',
			message: 'Link created successfully',
		})
	},
})
const creatingLink = computed(() => createLinkResource.loading)

function createLink() {
	if (createLinkDisabled.value) return
	createLinkResource.submit({
		data_source: props.name,
		primary_table: {
			label: dataSourceTable.doc.label,
			table: dataSourceTable.doc.table,
		},
		foreign_table: {
			label: newLink.table.label,
			table: newLink.table.table,
		},
		primary_key: newLink.primaryKey,
		foreign_key: newLink.foreignKey,
	})
}

function formatType(columnType) {
	return columnType.replace(/\(.*\)/, '').toUpperCase()
}
</script>
