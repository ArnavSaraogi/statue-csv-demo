<script lang="ts">
	import Papa from 'papaparse';

	// Props interface
	interface CsvEditorProps {
		filename?: string;
	}

	let { filename = 'edited_data.csv' }: CsvEditorProps = $props();

	// State using Svelte 5 runes
	let headers = $state<string[]>([]);
	let data = $state<string[][]>([]);
	let fileInputElement = $state<HTMLInputElement | null>(null);
	let isFullscreen = $state(false);
	let filenameValue = $state(filename);

	// Derived state
	let isEmpty = $derived(headers.length === 0);
	let rowCount = $derived(data.length);
	let colCount = $derived(headers.length);

	// Sync filename state with prop
	$effect(() => {
		filenameValue = filename;
	});

	// File loading
	function loadCsvFile(event: Event) {
		const target = event.target as HTMLInputElement;
		const file = target.files?.[0];
		if (!file) return;

		Papa.parse(file, {
			complete: (results) => {
				const allData = results.data as string[][];
				if (allData.length > 0) {
					headers = allData[0] || [];
					data = allData.slice(1).filter(row => row.some(cell => cell !== ''));
				}
			},
			skipEmptyLines: true
		});
	}

	// Cell editing
	function updateCell(rowIdx: number, colIdx: number, event: Event) {
		const target = event.target as HTMLDivElement;
		const value = target.textContent || '';
		data[rowIdx][colIdx] = value;
	}

	// Header editing
	function updateHeader(colIdx: number, event: Event) {
		const target = event.target as HTMLDivElement;
		const value = target.textContent || `Column ${colIdx + 1}`;
		headers[colIdx] = value;
	}

	// Svelte action to manage cell content without reactivity conflicts
	function initCellContent(node: HTMLElement, text: string) {
		// Set initial content
		node.textContent = text;

		return {
			update(newText: string) {
				// Only update if cell is not currently focused (being edited)
				if (document.activeElement !== node) {
					node.textContent = newText;
				}
			}
		};
	}

	// Helper function to get base filename without extension
	function getBaseFilename(name: string): string {
		const lastDot = name.lastIndexOf('.');
		return lastDot > 0 ? name.slice(0, lastDot) : name;
	}

	// Row operations
	function addRow() {
		data = [...data, new Array(headers.length).fill('')];
	}

	function deleteRow(index: number) {
		if (confirm('Delete this row?')) {
			data = data.filter((_, i) => i !== index);
		}
	}

	// Column operations
	function addColumn() {
		const name = prompt('Column name:');
		if (name) {
			headers = [...headers, name];
			data = data.map(row => [...row, '']);
		}
	}

	function deleteColumn(index: number) {
		if (confirm('Delete this column?')) {
			headers = headers.filter((_, i) => i !== index);
			data = data.map(row => row.filter((_, i) => i !== index));
		}
	}

	// Download CSV
	function downloadCsv() {
		const csv = Papa.unparse({ fields: headers, data });
		const blob = new Blob([csv], { type: 'text/csv' });
		const link = document.createElement('a');
		link.href = URL.createObjectURL(blob);
		// Ensure .csv extension
		const base = getBaseFilename(filenameValue);
		link.download = base + '.csv';
		link.click();
		URL.revokeObjectURL(link.href);
	}

	// Download JSON
	function downloadJson() {
		const jsonData = data.map(row => {
			const obj: Record<string, string> = {};
			headers.forEach((header, i) => {
				obj[header] = row[i] || '';
			});
			return obj;
		});
		const blob = new Blob([JSON.stringify(jsonData, null, 2)], { type: 'application/json' });
		const link = document.createElement('a');
		link.href = URL.createObjectURL(blob);
		// Use base filename with .json extension
		const base = getBaseFilename(filenameValue);
		link.download = base + '.json';
		link.click();
		URL.revokeObjectURL(link.href);
	}

	// Trigger file input
	function triggerFileInput() {
		fileInputElement?.click();
	}

	// Create blank CSV
	function createBlankCsv() {
		headers = ['Column 1', 'Column 2', 'Column 3'];
		data = [
			['', '', ''],
			['', '', ''],
			['', '', '']
		];
	}

	// Reset editor
	function resetEditor() {
		if (confirm('Reset editor and clear all data? This cannot be undone.')) {
			headers = [];
			data = [];
		}
	}

	// Handle keyboard events
	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Escape' && isFullscreen) {
			isFullscreen = false;
		}
	}
</script>

<svelte:window on:keydown={handleKeydown} />

{#snippet csvEditorContent()}
	{#if isEmpty}
		<div class="empty-state">
			<div class="empty-icon">
				<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
					<polyline points="14 2 14 8 20 8"></polyline>
					<line x1="12" y1="18" x2="12" y2="12"></line>
					<line x1="9" y1="15" x2="15" y2="15"></line>
				</svg>
			</div>
			<h3>No CSV Loaded</h3>
			<p>Click a button below to load a file or create a new CSV</p>
			<div class="empty-state-actions">
				<button class="btn-primary" onclick={triggerFileInput}>
					Load CSV File
				</button>
				<button class="btn" onclick={createBlankCsv}>
					Create Blank CSV
				</button>
			</div>
			<input
				bind:this={fileInputElement}
				type="file"
				accept=".csv"
				onchange={loadCsvFile}
				style="display: none;"
			/>
		</div>
	{:else}
		<div class="toolbar">
			<div class="toolbar-section">
				<button class="btn" onclick={triggerFileInput} title="Load new CSV file">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
						<polyline points="17 8 12 3 7 8"></polyline>
						<line x1="12" y1="3" x2="12" y2="15"></line>
					</svg>
					Load CSV
				</button>
				<button class="btn btn-reset" onclick={resetEditor} title="Reset editor and clear all data">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<polyline points="1 4 1 10 7 10"></polyline>
						<path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"></path>
					</svg>
					Reset
				</button>
				<input
					bind:this={fileInputElement}
					type="file"
					accept=".csv"
					onchange={loadCsvFile}
					style="display: none;"
				/>
			</div>

			<div class="toolbar-section">
				<button class="btn" onclick={addRow} title="Add new row">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<line x1="12" y1="5" x2="12" y2="19"></line>
						<line x1="5" y1="12" x2="19" y2="12"></line>
					</svg>
					Add Row
				</button>
				<button class="btn" onclick={addColumn} title="Add new column">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<line x1="12" y1="5" x2="12" y2="19"></line>
						<line x1="5" y1="12" x2="19" y2="12"></line>
					</svg>
					Add Column
				</button>
			</div>

			<div class="toolbar-section">
				<div class="filename-input-wrapper">
					<label for="filename-input" class="filename-label">Filename:</label>
					<input
						id="filename-input"
						type="text"
						bind:value={filenameValue}
						class="filename-input"
						placeholder="filename"
						title="Edit filename for downloads"
					/>
				</div>

				<button class="btn-primary" onclick={downloadCsv} title="Download as CSV">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
						<polyline points="7 10 12 15 17 10"></polyline>
						<line x1="12" y1="15" x2="12" y2="3"></line>
					</svg>
					Download CSV
				</button>
				<button class="btn" onclick={downloadJson} title="Download as JSON">
					<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
						<polyline points="7 10 12 15 17 10"></polyline>
						<line x1="12" y1="15" x2="12" y2="3"></line>
					</svg>
					Export JSON
				</button>
			</div>

			<div class="info-section">
				<span class="info-badge">{rowCount} rows × {colCount} columns</span>
			</div>
		</div>

		<div class="table-container" class:fullscreen={isFullscreen}>
			<table class="data-table">
				<thead>
					<tr>
						<th class="row-number-header">#</th>
						{#each headers as header, colIdx}
							<th>
								<div class="header-cell">
									<div
										class="header-text editable-header"
										contenteditable="true"
										onblur={(e) => updateHeader(colIdx, e)}
										use:initCellContent={header}
									></div>
									<button
										class="delete-btn"
										onclick={() => deleteColumn(colIdx)}
										title="Delete column"
									>
										<svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
											<line x1="18" y1="6" x2="6" y2="18"></line>
											<line x1="6" y1="6" x2="18" y2="18"></line>
										</svg>
									</button>
								</div>
							</th>
						{/each}
					</tr>
				</thead>
				<tbody>
					{#each data as row, rowIdx}
						<tr>
							<td class="row-number">{rowIdx + 1}</td>
							{#each row as cell, colIdx}
								<td>
									<div
										class="editable-cell"
										contenteditable="true"
										onblur={(e) => updateCell(rowIdx, colIdx, e)}
										use:initCellContent={cell}
									></div>
								</td>
							{/each}
							<td class="row-actions">
								<button
									class="delete-btn"
									onclick={() => deleteRow(rowIdx)}
									title="Delete row"
								>
									<svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
										<line x1="18" y1="6" x2="6" y2="18"></line>
										<line x1="6" y1="6" x2="18" y2="18"></line>
									</svg>
								</button>
							</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	{/if}
{/snippet}

<!-- Normal mode -->
{#if !isFullscreen}
	<div class="csv-editor">
		<!-- Full-screen button in top-right corner -->
		<button
			class="fullscreen-btn"
			onclick={() => isFullscreen = true}
			title="Open in full screen"
			aria-label="Open in full screen"
		>
			<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
				<path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"/>
			</svg>
		</button>

		{@render csvEditorContent()}
	</div>
{/if}

<!-- Full-screen modal -->
{#if isFullscreen}
	<!-- Backdrop -->
	<!-- svelte-ignore a11y_click_events_have_key_events -->
	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<div
		class="modal-backdrop"
		onclick={() => isFullscreen = false}
	></div>

	<!-- Modal content -->
	<div class="modal-container" role="dialog" aria-modal="true" aria-labelledby="modal-title">
		<!-- Header with close button -->
		<div class="modal-header">
			<h2 id="modal-title" class="modal-title">CSV Editor</h2>
			<button
				class="close-modal-btn"
				onclick={() => isFullscreen = false}
				title="Close full screen"
				aria-label="Close full screen"
			>
				<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<line x1="18" y1="6" x2="6" y2="18"></line>
					<line x1="6" y1="6" x2="18" y2="18"></line>
				</svg>
			</button>
		</div>

		<!-- CSV Editor content -->
		<div class="modal-content">
			{@render csvEditorContent()}
		</div>
	</div>
{/if}

<style>
	.csv-editor {
		position: relative;
		width: 100%;
		background: var(--color-card);
		border-radius: 8px;
		border: 1px solid var(--color-border);
		overflow: hidden;
	}

	/* Full-screen button (top-right corner) */
	.fullscreen-btn {
		position: absolute;
		top: 0.5rem;
		right: 0.5rem;
		z-index: 10;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 0.5rem;
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 6px;
		color: var(--color-foreground);
		cursor: pointer;
		transition: all 0.2s ease;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}

	.fullscreen-btn:hover {
		background: var(--color-muted);
		border-color: var(--color-primary);
		transform: scale(1.05);
	}

	/* Empty state */
	.empty-state {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 4rem 2rem;
		text-align: center;
		gap: 1rem;
	}

	.empty-icon {
		color: var(--color-muted);
		opacity: 0.5;
	}

	.empty-state h3 {
		font-size: 1.25rem;
		font-weight: 600;
		color: var(--color-foreground);
		margin: 0;
	}

	.empty-state p {
		color: var(--color-muted);
		margin: 0;
	}

	.empty-state-actions {
		display: flex;
		gap: 0.75rem;
		align-items: center;
		flex-wrap: wrap;
		justify-content: center;
	}

	/* Toolbar */
	.toolbar {
		display: flex;
		align-items: center;
		gap: 1rem;
		padding: 1rem;
		border-bottom: 1px solid var(--color-border);
		background: var(--color-background);
		flex-wrap: wrap;
	}

	.toolbar-section {
		display: flex;
		gap: 0.5rem;
		align-items: center;
	}

	.info-section {
		margin-left: auto;
	}

	.info-badge {
		font-size: 0.875rem;
		color: var(--color-muted);
		padding: 0.25rem 0.75rem;
		background: var(--color-card);
		border-radius: 4px;
		border: 1px solid var(--color-border);
	}

	/* Buttons */
	.btn,
	.btn-primary {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.5rem 1rem;
		font-size: 0.875rem;
		font-weight: 500;
		border-radius: 6px;
		border: 1px solid var(--color-border);
		background: var(--color-card);
		color: var(--color-foreground);
		cursor: pointer;
		transition: all 0.2s ease;
	}

	.btn:hover {
		background: var(--color-muted);
		border-color: var(--color-primary);
	}

	.btn-primary {
		background: var(--color-primary);
		color: var(--color-on-primary);
		border-color: var(--color-primary);
	}

	.btn-primary:hover {
		opacity: 0.9;
		transform: translateY(-1px);
	}

	.btn-reset:hover {
		background: rgba(255, 0, 0, 0.1);
		border-color: #ef4444;
		color: #ef4444;
	}

	.filename-input-wrapper {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.25rem 0;
	}

	.filename-label {
		font-size: 0.8125rem;
		font-weight: 500;
		color: var(--color-foreground);
		white-space: nowrap;
	}

	.filename-input {
		padding: 0.375rem 0.625rem;
		font-size: 0.8125rem;
		border-radius: 4px;
		border: 1px solid var(--color-border);
		background: var(--color-background);
		color: var(--color-foreground);
		transition: all 0.2s ease;
		min-width: 150px;
		max-width: 200px;
	}

	.filename-input:focus {
		outline: none;
		border-color: var(--color-primary);
		box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
	}

	.filename-input::placeholder {
		color: var(--color-muted);
	}

	/* Table container */
	.table-container {
		overflow-x: auto;
		max-height: 600px;
		overflow-y: auto;
	}

	.table-container.fullscreen {
		max-height: none;
		flex: 1;
	}

	.data-table {
		width: 100%;
		border-collapse: collapse;
		font-size: 0.875rem;
	}

	/* Table header */
	thead {
		position: sticky;
		top: 0;
		background: var(--color-muted);
		z-index: 10;
	}

	th {
		padding: 0.75rem 1rem;
		text-align: left;
		font-weight: 600;
		color: var(--color-foreground);
		border-bottom: 2px solid var(--color-border);
		border-right: 1px solid var(--color-border);
		white-space: nowrap;
	}

	.row-number-header {
		width: 50px;
		text-align: center;
		background: var(--color-background);
		position: sticky;
		left: 0;
		z-index: 11;
	}

	.header-cell {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 0.5rem;
		min-width: 120px;
	}

	.editable-header {
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		padding: 0.25rem;
		margin: -0.25rem;
		border-radius: 4px;
		cursor: text;
		outline: none;
		transition: background 0.2s ease, box-shadow 0.2s ease;
	}

	.editable-header:hover {
		background: rgba(0, 0, 0, 0.05);
	}

	.editable-header:focus {
		background: var(--color-card);
		box-shadow: inset 0 0 0 2px var(--color-primary);
		cursor: text;
	}

	/* Table body */
	tbody tr:hover {
		background: var(--color-hover, rgba(0, 0, 0, 0.02));
	}

	td {
		padding: 0;
		border-bottom: 1px solid var(--color-border);
		border-right: 1px solid var(--color-border);
	}

	.row-number {
		width: 50px;
		text-align: center;
		font-weight: 500;
		color: var(--color-muted);
		background: var(--color-background);
		position: sticky;
		left: 0;
		padding: 0.5rem;
		z-index: 1;
	}

	.editable-cell {
		padding: 0.75rem 1rem;
		min-width: 120px;
		min-height: 40px;
		outline: none;
		cursor: text;
	}

	.editable-cell:focus {
		background: var(--color-card);
		box-shadow: inset 0 0 0 2px var(--color-primary);
	}

	.row-actions {
		width: 50px;
		text-align: center;
		padding: 0.5rem;
		position: sticky;
		right: 0;
		background: var(--color-background);
		z-index: 1;
	}

	/* Delete buttons */
	.delete-btn {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 0.25rem;
		background: rgba(255, 255, 255, 0.1);
		border: 1px solid var(--color-border);
		color: var(--color-foreground);
		cursor: pointer;
		border-radius: 4px;
		transition: all 0.2s ease;
		opacity: 0.7;
	}

	.delete-btn:hover {
		opacity: 1;
		background: rgba(255, 0, 0, 0.15);
		border-color: #ef4444;
		color: #ef4444;
	}

	/* Modal backdrop */
	.modal-backdrop {
		position: fixed;
		inset: 0;
		z-index: 40;
		background: rgba(0, 0, 0, 0.5);
		backdrop-filter: blur(4px);
	}

	/* Modal container */
	.modal-container {
		position: fixed;
		inset: 1rem;
		z-index: 50;
		display: flex;
		flex-direction: column;
		background: var(--color-background);
		border-radius: 8px;
		box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.3);
		overflow: hidden;
	}

	/* Modal header */
	.modal-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 1rem;
		border-bottom: 1px solid var(--color-border);
		background: var(--color-card);
	}

	.modal-title {
		font-size: 1.25rem;
		font-weight: 600;
		color: var(--color-foreground);
		margin: 0;
	}

	/* Close button */
	.close-modal-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 0.5rem;
		background: transparent;
		border: 1px solid var(--color-border);
		border-radius: 4px;
		color: var(--color-foreground);
		cursor: pointer;
		transition: all 0.2s ease;
	}

	.close-modal-btn:hover {
		background: var(--color-muted);
		border-color: var(--color-primary);
	}

	/* Modal content */
	.modal-content {
		flex: 1;
		overflow: hidden;
		display: flex;
		flex-direction: column;
	}

	.modal-content .toolbar {
		flex-shrink: 0;
	}

	.modal-content .table-container {
		flex: 1;
		max-height: none;
	}

	.modal-content .empty-state {
		flex: 1;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}

	/* Responsive */
	@media (max-width: 768px) {
		.toolbar {
			flex-direction: column;
			align-items: stretch;
		}

		.toolbar-section {
			width: 100%;
			justify-content: space-between;
		}

		.info-section {
			margin-left: 0;
			width: 100%;
		}

		.info-badge {
			display: block;
			text-align: center;
		}

		.btn,
		.btn-primary {
			flex: 1;
			justify-content: center;
		}

		.filename-input-wrapper {
			width: 100%;
			flex-direction: column;
			align-items: stretch;
			gap: 0.25rem;
		}

		.filename-input {
			width: 100%;
			max-width: none;
		}

		.modal-container {
			inset: 0.5rem;
		}
	}
</style>
