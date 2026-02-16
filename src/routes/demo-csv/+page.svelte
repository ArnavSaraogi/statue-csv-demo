<script lang="ts">
	import CsvEditor from '$lib/components/CsvEditor.svelte';
</script>

<svelte:head>
	<title>CSV Editor Demo - Statue SSG</title>
	<meta
		name="description"
		content="Interactive CSV editor component with upload, edit, and download capabilities"
	/>
</svelte:head>

<div class="demo-container">
	<header class="demo-header">
		<h1>CSV Editor Component</h1>
		<p class="subtitle">
			Features include creating CSVs, loading and editing CSVs, downloading data as CSV or JSON,
			adding and deleting rows/columns, and fullscreen mode for ease of use.
		</p>
	</header>

	<section class="demo-section">
		<div class="section-header">
			<h2>Live Demo</h2>
			<p>
				Try uploading a CSV file, editing cells, adding rows/columns, and downloading your changes.
			</p>
		</div>

		<div class="component-wrapper">
			<CsvEditor filename="sample_data.csv" />
		</div>
	</section>

	<section class="demo-section">
		<div class="section-header">
			<h2>How to Use</h2>
		</div>

		<div class="instructions-grid">
			<div class="instruction-card">
				<div class="instruction-number">1</div>
				<h3>Load or Create CSV</h3>
				<p>
					Click "Load CSV File" to upload an existing file, or click "Create Blank CSV" to start
					with an empty 3x3 grid that you can immediately edit and expand.
				</p>
			</div>

			<div class="instruction-card">
				<div class="instruction-number">2</div>
				<h3>Edit Cells</h3>
				<p>
					Click any cell to edit its content. Changes are saved automatically when you click outside
					the cell (on blur).
				</p>
			</div>

			<div class="instruction-card">
				<div class="instruction-number">3</div>
				<h3>Add/Delete Rows</h3>
				<p>
					Use "Add Row" to append a new empty row. Click the × button on any row to delete it
					(confirmation required).
				</p>
			</div>

			<div class="instruction-card">
				<div class="instruction-number">4</div>
				<h3>Add/Delete Columns</h3>
				<p>
					Use "Add Column" to create a new column (you'll be prompted for a name). Click × in the
					header to delete a column.
				</p>
			</div>

			<div class="instruction-card">
				<div class="instruction-number">5</div>
				<h3>Download CSV</h3>
				<p>
					Click "Download CSV" to export your edited data as a CSV file. The file will use the
					filename prop (default: edited_data.csv).
				</p>
			</div>

			<div class="instruction-card">
				<div class="instruction-number">6</div>
				<h3>Export JSON</h3>
				<p>
					Click "Export JSON" to download your data as a JSON file. Each row becomes an object with
					column headers as keys.
				</p>
			</div>

			<div class="instruction-card">
				<div class="instruction-number">7</div>
				<h3>Full-Screen Mode</h3>
				<p>
					Click the expand icon next to the row/column count to open the editor in full-screen mode.
					Press ESC or click outside to exit.
				</p>
			</div>
		</div>
	</section>

	<section class="demo-section">
		<div class="section-header">
			<h2>Features</h2>
		</div>

		<div class="features-grid">
			<div class="feature-card">
				<div class="feature-icon">📁</div>
				<h3>CSV File Upload</h3>
				<p>
					Built-in file input with CSV parsing powered by PapaParse. Handles various CSV formats and
					encodings.
				</p>
			</div>

			<div class="feature-card">
				<div class="feature-icon">📄</div>
				<h3>Create from Scratch</h3>
				<p>
					Start with a blank 3x3 CSV grid without needing an existing file. Perfect for creating new
					datasets from scratch.
				</p>
			</div>

			<div class="feature-card">
				<div class="feature-icon">✏️</div>
				<h3>Inline Editing</h3>
				<p>
					Click any cell to edit content with contenteditable. Changes save on blur for a seamless
					editing experience.
				</p>
			</div>

			<div class="feature-card">
				<div class="feature-icon">➕</div>
				<h3>Dynamic Rows/Columns</h3>
				<p>
					Add or remove rows and columns on the fly. Confirmation dialogs prevent accidental
					deletions.
				</p>
			</div>

			<div class="feature-card">
				<div class="feature-icon">⬇️</div>
				<h3>Export Options</h3>
				<p>
					Download your data as CSV or JSON. Perfect for data manipulation and format conversion.
				</p>
			</div>

			<div class="feature-card">
				<div class="feature-icon">🎨</div>
				<h3>Theme Support</h3>
				<p>Uses CSS variables for theming. Automatically adapts to your selected Statue theme.</p>
			</div>

			<div class="feature-card">
				<div class="feature-icon">📱</div>
				<h3>Responsive Design</h3>
				<p>
					Fully responsive with sticky headers and row numbers. Works on mobile, tablet, and
					desktop.
				</p>
			</div>

			<div class="feature-card">
				<div class="feature-icon">🖥️</div>
				<h3>Full-Screen Mode</h3>
				<p>
					Expand the editor to full-screen for easier viewing and editing of large datasets. Exit
					with ESC or backdrop click.
				</p>
			</div>
		</div>
	</section>

	<section class="demo-section">
		<div class="section-header">
			<h2>Sample CSV Data</h2>
			<p>
				Need sample data to test? Copy the CSV below and save it as a .csv file, then upload it.
			</p>
		</div>

		<div class="code-block">
			<pre><code
					>Name,Email,Role,Department,Salary
John Doe,john@example.com,Software Engineer,Engineering,95000
Jane Smith,jane@example.com,Product Manager,Product,105000
Bob Johnson,bob@example.com,Designer,Design,85000
Alice Williams,alice@example.com,Data Scientist,Engineering,98000
Charlie Brown,charlie@example.com,Marketing Manager,Marketing,92000</code
				></pre>
		</div>

		<div class="sample-description">
			<p>
				<strong>Alternative:</strong> You can also create a CSV file with any spreadsheet software (Excel,
				Google Sheets, LibreOffice Calc) and export it as CSV.
			</p>
		</div>
	</section>

	<section class="demo-section">
		<div class="section-header">
			<h2>Installation</h2>
		</div>

		<div class="code-block">
			<pre><code
					>npm install statue-ssg papaparse
npm install -D @types/papaparse</code
				></pre>
		</div>
	</section>

	<section class="demo-section">
		<div class="section-header">
			<h2>Usage in Your Project</h2>
		</div>

		<div class="code-block">
			<pre><code
					>&lt;script&gt;
  import &#123; CsvEditor &#125; from 'statue-ssg';
&lt;/script&gt;

&lt;CsvEditor filename="my_data.csv" /&gt;</code
				></pre>
		</div>

		<div class="props-table">
			<h3>Props</h3>
			<table>
				<thead>
					<tr>
						<th>Prop</th>
						<th>Type</th>
						<th>Default</th>
						<th>Description</th>
					</tr>
				</thead>
				<tbody>
					<tr>
						<td><code>filename</code></td>
						<td>string</td>
						<td>"edited_data.csv"</td>
						<td>The filename used when downloading CSV or JSON files</td>
					</tr>
				</tbody>
			</table>
		</div>
	</section>

	<section class="demo-section">
		<div class="section-header">
			<h2>Technical Details</h2>
		</div>

		<div class="tech-details">
			<ul>
				<li><strong>Svelte Version:</strong> Svelte 5 (uses runes: $state, $derived, $props)</li>
				<li><strong>TypeScript:</strong> Full TypeScript support with typed props and events</li>
				<li><strong>CSV Parsing:</strong> PapaParse library for robust CSV handling</li>
				<li><strong>State Management:</strong> Reactive state using Svelte 5 runes</li>
				<li><strong>Styling:</strong> Theme-aware CSS variables for seamless integration</li>
				<li><strong>Accessibility:</strong> Semantic HTML with ARIA labels and keyboard support</li>
			</ul>
		</div>
	</section>
</div>

<style>
	.demo-container {
		max-width: 1200px;
		margin: 0 auto;
		padding: 2rem 1rem;
	}

	.demo-header {
		text-align: center;
		margin-bottom: 3rem;
		padding-bottom: 2rem;
		border-bottom: 2px solid var(--color-border);
	}

	.demo-header h1 {
		font-size: 2.5rem;
		font-weight: 700;
		color: var(--color-foreground);
		margin: 0 0 1rem 0;
	}

	.subtitle {
		font-size: 1.125rem;
		color: var(--color-muted);
		max-width: 600px;
		margin: 0 auto;
	}

	.demo-section {
		margin-bottom: 3rem;
	}

	.section-header {
		margin-bottom: 1.5rem;
	}

	.section-header h2 {
		font-size: 1.875rem;
		font-weight: 600;
		color: var(--color-foreground);
		margin: 0 0 0.5rem 0;
	}

	.section-header p {
		color: var(--color-muted);
		margin: 0;
	}

	.component-wrapper {
		margin: 2rem 0;
	}

	/* Instructions Grid */
	.instructions-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
		gap: 1.5rem;
		margin-top: 1.5rem;
	}

	.instruction-card {
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 8px;
		padding: 1.5rem;
		transition:
			transform 0.2s ease,
			box-shadow 0.2s ease;
	}

	.instruction-card:hover {
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
	}

	.instruction-number {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		width: 32px;
		height: 32px;
		background: var(--color-primary);
		color: var(--color-on-primary);
		border-radius: 50%;
		font-weight: 700;
		font-size: 1rem;
		margin-bottom: 1rem;
	}

	.instruction-card h3 {
		font-size: 1.125rem;
		font-weight: 600;
		color: var(--color-foreground);
		margin: 0 0 0.5rem 0;
	}

	.instruction-card p {
		color: var(--color-muted);
		margin: 0;
		line-height: 1.6;
	}

	/* Features Grid */
	.features-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
		gap: 1.5rem;
		margin-top: 1.5rem;
	}

	.feature-card {
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 8px;
		padding: 1.5rem;
		text-align: center;
	}

	.feature-icon {
		font-size: 2.5rem;
		margin-bottom: 1rem;
	}

	.feature-card h3 {
		font-size: 1.125rem;
		font-weight: 600;
		color: var(--color-foreground);
		margin: 0 0 0.5rem 0;
	}

	.feature-card p {
		color: var(--color-muted);
		margin: 0;
		line-height: 1.6;
		font-size: 0.9rem;
	}

	/* Code Block */
	.code-block {
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 8px;
		padding: 1.5rem;
		overflow-x: auto;
		margin: 1rem 0;
	}

	.code-block pre {
		margin: 0;
		font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
		font-size: 0.875rem;
		line-height: 1.6;
	}

	.code-block code {
		color: var(--color-foreground);
	}

	.sample-description {
		margin-top: 1rem;
		padding: 1rem;
		background: var(--color-background);
		border-left: 4px solid var(--color-primary);
		border-radius: 4px;
	}

	.sample-description p {
		margin: 0;
		color: var(--color-muted);
	}

	/* Props Table */
	.props-table {
		margin-top: 1.5rem;
	}

	.props-table h3 {
		font-size: 1.25rem;
		font-weight: 600;
		color: var(--color-foreground);
		margin: 0 0 1rem 0;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 8px;
		overflow: hidden;
	}

	thead {
		background: var(--color-muted);
	}

	th {
		padding: 0.75rem 1rem;
		text-align: left;
		font-weight: 600;
		color: var(--color-foreground);
		border-bottom: 2px solid var(--color-border);
	}

	td {
		padding: 0.75rem 1rem;
		color: var(--color-foreground);
		border-bottom: 1px solid var(--color-border);
	}

	tbody tr:last-child td {
		border-bottom: none;
	}

	code {
		background: var(--color-background);
		padding: 0.2rem 0.4rem;
		border-radius: 4px;
		font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
		font-size: 0.875rem;
		color: var(--color-primary);
	}

	/* Technical Details */
	.tech-details ul {
		list-style: none;
		padding: 0;
		margin: 1rem 0;
	}

	.tech-details li {
		padding: 0.75rem 1rem;
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 6px;
		margin-bottom: 0.5rem;
		color: var(--color-foreground);
	}

	.tech-details li strong {
		color: var(--color-primary);
	}

	/* Responsive */
	@media (max-width: 768px) {
		.demo-header h1 {
			font-size: 2rem;
		}

		.section-header h2 {
			font-size: 1.5rem;
		}

		.instructions-grid,
		.features-grid {
			grid-template-columns: 1fr;
		}

		table {
			font-size: 0.875rem;
		}

		th,
		td {
			padding: 0.5rem;
		}
	}
</style>
