<template>
	<div class="mx-auto h-full max-w-full">
		<div v-if="props.title" class="h-5 text-base font-semibold text-gray-600">
			{{ props.title }}
		</div>
		<div
			class="relative mt-2 flex w-full flex-col overflow-scroll text-base scrollbar-hide"
			:class="[props.title ? 'h-[calc(100%-1.75rem)]' : 'h-[calc(100%-1rem)]']"
		>
			<table v-if="props.columns">
				<thead class="sticky top-0">
					<tr>
						<td
							v-for="column in props.columns"
							class="whitespace-nowrap bg-gray-100 px-2.5 py-1.5 font-medium text-gray-600 first:rounded-l-md last:rounded-r-md"
							scope="col"
						>
							{{ column }}
						</td>
					</tr>
				</thead>
				<tbody>
					<tr v-for="row in props.rows" class="border-b">
						<td
							v-for="cell in row"
							class="whitespace-nowrap bg-white px-2.5 py-2 font-light text-gray-600"
						>
							{{ ellipsis(cell, 100) }}
						</td>
					</tr>
				</tbody>
			</table>
			<div
				v-if="props.rows.length == 0"
				class="mt-2 flex h-[calc(100%-1.5rem)] w-full items-center justify-center text-lg font-light text-gray-500"
			>
				No Data
			</div>
		</div>
	</div>
</template>

<script setup>
import { ellipsis } from '@/utils'

const props = defineProps({
	title: {
		type: String,
		default: '',
	},
	columns: {
		type: Array,
	},
	rows: {
		type: Array,
	},
})
</script>
