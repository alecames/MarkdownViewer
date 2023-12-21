<script lang="ts">
	import { invoke } from "@tauri-apps/api/tauri";
	import { onMount } from "svelte";
	import { open } from "@tauri-apps/api/shell";

	let editPath: string = "";
	let htmlContent: string = "";

	async function loadMarkdown(filePath: string) {
		try {
			const html = await invoke("open_markdown", { path: filePath });
			htmlContent = html as string;
		} catch (error) {
			console.error("Error loading Markdown file:", error);
		}
	}

	async function openInEditor() {
		if (editPath) {
			await open(editPath, "code");
		} else {
			console.error("No file path available to open in editor.");
		}
	}

	onMount(() => {
		invoke("send_markdown_path")
			.then((path: string) => {
				editPath = path;
				loadMarkdown(path);
			})
			.catch((error) => {
				console.error("Error receiving Markdown file path:", error);
			});
	});

	onMount(async () => {
		setTimeout(() => {
			setupWindow();
			interceptLinks();
		}, 300);
	});

	// https://github.com/tauri-apps/tauri/issues/5170
	async function setupWindow() {
		const appWindow = (await import("@tauri-apps/api/window")).appWindow;
		appWindow.show();
	}

	function interceptLinks() {
		document.addEventListener("click", async (event) => {
			let target = event.target as HTMLElement;

			while (target && target.tagName !== "A" && target !== document.body) {
				target = target.parentElement;
			}

			if (
				target &&
				target.tagName === "A" &&
				(target as HTMLAnchorElement).href
			) {
				const anchor = target as HTMLAnchorElement;

				if (!anchor.href.startsWith("#")) {
					event.preventDefault();
					await open(anchor.href);
				}
			}
		});
	}
</script>

{#if !htmlContent}
	<div class="message">
		<p>
			Open a Markdown file by right-clicking and selecting 'Open with...'
			&rightarrow; 'Markdown Viewer'
		</p>
	</div>
{:else}
	<article
		contenteditable="false"
		class="markdown-body"
		bind:innerHTML={htmlContent}>
	</article>
{/if}

<style>
	:root {
		--animation: cubic-bezier(0.05, 0.95, 0.05, 0.95);
	}

	.markdown-body {
		box-sizing: border-box;
		min-width: 200px;
		max-width: 980px;
		margin: 0 auto;
		padding: 45px;
	}

	.message {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		user-select: none;
		font-family:
			Segoe UI,
			Helvetica Neue,
			Helvetica,
			Arial,
			sans-serif;
		height: 90vh;
	}

	@media (prefers-color-scheme: dark) {
		.message {
			color: #ffffffaa;
		}
	}

	@media (prefers-color-scheme: light) {
		.message {
			color: #000000aa;
		}
	}
</style>
