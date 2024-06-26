<script lang="ts">
	import type { PageData } from './$types';
	import { RadioGroup, RadioItem } from '@skeletonlabs/skeleton';
	import * as m from '$paraglide/messages';
	import { breadcrumbObject } from '$lib/utils/stores';
	import { COMPLIANCE_COLOR_MAP } from '$lib/utils/constants';
	import { getRequirementTitle } from '$lib/utils/helpers';

	export let data: PageData;

	breadcrumbObject.set(data.compliance_assessment);

	let possible_options = [
		{ id: 'to_do', label: m.toDo() },
		{ id: 'in_progress', label: m.inProgress() },
		{ id: 'non_compliant', label: m.nonCompliant() },
		{ id: 'partially_compliant', label: m.partiallyCompliant() },
		{ id: 'compliant', label: m.compliant() },
		{ id: 'not_applicable', label: m.notApplicable() }
	];

	// Reactive variable to keep track of the current item index
	let currentIndex = 0;

	$: color = COMPLIANCE_COLOR_MAP[data.requirement_assessments[currentIndex].status];

	$: requirement = data.requirements.find(
		(req) => req.id === data.requirement_assessments[currentIndex].requirement
	);
	$: parent = data.requirements.find((req) => req.urn === requirement.parent_urn);

	$: title = requirement.display_short
		? requirement.display_short
		: parent.display_short
		? parent.display_short
		: parent.description;

	// Function to handle the "Next" button click
	function nextItem() {
		if (currentIndex < data.requirement_assessments.length - 1) {
			currentIndex += 1;
		} else {
			currentIndex = 0;
		}
	}

	// Function to handle the "Back" button click
	function previousItem() {
		if (currentIndex > 0) {
			currentIndex -= 1;
		} else {
			currentIndex = data.requirement_assessments.length - 1;
		}
	}

	// Function to update the status of the current item
	function updateStatus(event) {
		data.requirement_assessments[currentIndex].status = event.target.value;
		const form = document.getElementById('flashModeForm');
		const formData = new FormData(form);
		fetch(form.action, {
			method: 'POST',
			body: formData
		});
	}
</script>

<div class="flex flex-col h-full justify-center items-center">
	<div
		style="border-color: {color}"
		class="flex flex-col bg-white w-3/4 h-3/4 rounded-xl shadow-xl p-4 border-4"
	>
		{#if data.requirement_assessments[currentIndex]}
			<div class="flex flex-col w-full h-full space-y-4">
				<div class="flex justify-between h-1/6">
					<div class="">
						<a
							href="/compliance-assessments/{data.compliance_assessment.id}"
							class="flex items-center space-x-2 text-primary-800 hover:text-primary-600"
						>
							<i class="fa-solid fa-arrow-left" />
							<p class="">{m.goBackToAudit()}</p>
						</a>
					</div>
					<div class="font-semibold">{currentIndex + 1}/{data.requirement_assessments.length}</div>
				</div>
				<div class="flex flex-col h-1/2 items-center text-center justify-center">
					<p class="font-semibold">{title}</p>
					{#if data.requirement_assessments[currentIndex].description}
						{data.requirement_assessments[currentIndex].description}
					{/if}
				</div>
				<div class="items-center">
					<div class="">
						<h3 class="mb-4 font-semibold text-gray-900 dark:text-white">{m.status()}</h3>
						<form id="flashModeForm" action="?/updateRequirementAssessment" method="post">
							<ul
								class="items-center w-full text-sm font-medium text-gray-900 bg-white border border-gray-200 rounded-lg sm:flex dark:bg-gray-700 dark:border-gray-600 dark:text-white"
							>
								<input hidden name="id" value={data.requirement_assessments[currentIndex].id} />
								{#each possible_options as option}
									<li
										class="w-full border-b border-gray-200 sm:border-b-0 sm:border-r dark:border-gray-600"
									>
										<div class="flex items-center ps-3">
											<input
												id={option.id}
												type="radio"
												value={option.id}
												name="status"
												checked={option.id === data.requirement_assessments[currentIndex].status}
												on:change={updateStatus}
												class="w-4 h-4 text-primary-500 bg-gray-100 border-gray-300 focus:ring-primary-500 dark:focus:ring-primary-600 dark:ring-offset-gray-700 dark:focus:ring-offset-gray-700 focus:ring-2 dark:bg-gray-600 dark:border-gray-500"
											/>
											<label
												for={option.id}
												class="w-full py-3 ms-2 text-sm font-medium text-gray-900 dark:text-gray-300"
												>{option.label}
											</label>
										</div>
									</li>
								{/each}
							</ul>
						</form>
					</div>
				</div>
			</div>
			<div class="flex justify-between">
				<button class="bg-gray-400 text-white px-4 py-2 rounded" on:click={previousItem}>
					{m.previous()}
				</button>
				<button class="variant-filled-primary px-4 py-2 rounded" on:click={nextItem}>
					{m.next()}
				</button>
			</div>
		{/if}
	</div>
</div>
