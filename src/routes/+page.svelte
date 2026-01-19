<script lang="ts">
  import { invoke } from "@tauri-apps/api/core";
  import { onMount } from "svelte";

  interface FileEntry {
    name: string;
    path: string;
    is_dir: boolean;
    size: number;
  }

  let files: FileEntry[] = $state([]);
  let currentPath: string = $state("");
  let loading: boolean = $state(false);
  let error: string = $state("");

  async function loadFiles(path?: string) {
    loading = true;
    error = "";
    try {
      files = await invoke<FileEntry[]>("list_files", { path });
      if (path) {
        currentPath = path;
      }
    } catch (e) {
      error = String(e);
      files = [];
    } finally {
      loading = false;
    }
  }

  async function navigateTo(entry: FileEntry) {
    if (entry.is_dir) {
      await loadFiles(entry.path);
    }
  }

  async function goUp() {
    if (!currentPath) return;
    const parent = await invoke<string | null>("get_parent_path", { path: currentPath });
    if (parent) {
      await loadFiles(parent);
    }
  }

  function formatSize(bytes: number): string {
    if (bytes === 0) return "—";
    const units = ["B", "KB", "MB", "GB"];
    const i = Math.floor(Math.log(bytes) / Math.log(1024));
    return `${(bytes / Math.pow(1024, i)).toFixed(i > 0 ? 1 : 0)} ${units[i]}`;
  }

  onMount(() => {
    loadFiles();
  });
</script>

<main>
  <header>
    <h1>File Browser</h1>
    <div class="path-bar">
      <button class="up-btn" onclick={goUp} disabled={!currentPath}>
        ↑
      </button>
      <span class="current-path">{currentPath || "Home"}</span>
    </div>
  </header>

  {#if loading}
    <div class="loading">
      <div class="spinner"></div>
      <span>Loading...</span>
    </div>
  {:else if error}
    <div class="error">
      <span class="error-icon">⚠</span>
      <span>{error}</span>
    </div>
  {:else if files.length === 0}
    <div class="empty">This folder is empty</div>
  {:else}
    <ul class="file-list">
      {#each files as entry}
        <li>
          <button
            class="file-entry"
            class:is-dir={entry.is_dir}
            onclick={() => navigateTo(entry)}
            disabled={!entry.is_dir}
          >
            <span class="icon">{entry.is_dir ? "📁" : "📄"}</span>
            <span class="name">{entry.name}</span>
            <span class="size">{entry.is_dir ? "" : formatSize(entry.size)}</span>
          </button>
        </li>
      {/each}
    </ul>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    background: #0d1117;
    color: #c9d1d9;
  }

  main {
    max-width: 720px;
    margin: 0 auto;
    padding: 1.5rem;
    font-family: "JetBrains Mono", "Fira Code", monospace;
  }

  header {
    margin-bottom: 1.5rem;
  }

  h1 {
    font-size: 1.5rem;
    font-weight: 600;
    color: #58a6ff;
    margin: 0 0 1rem 0;
    letter-spacing: -0.5px;
  }

  .path-bar {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    background: #161b22;
    border: 1px solid #30363d;
    border-radius: 6px;
    padding: 0.5rem 0.75rem;
  }

  .up-btn {
    background: #21262d;
    border: 1px solid #30363d;
    color: #c9d1d9;
    width: 32px;
    height: 32px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.15s ease;
  }

  .up-btn:hover:not(:disabled) {
    background: #30363d;
    border-color: #58a6ff;
  }

  .up-btn:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  .current-path {
    font-size: 0.85rem;
    color: #8b949e;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .loading {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.75rem;
    padding: 3rem;
    color: #8b949e;
  }

  .spinner {
    width: 20px;
    height: 20px;
    border: 2px solid #30363d;
    border-top-color: #58a6ff;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  .error {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 1rem;
    background: #3d1a1a;
    border: 1px solid #f85149;
    border-radius: 6px;
    color: #f85149;
  }

  .error-icon {
    font-size: 1.25rem;
  }

  .empty {
    text-align: center;
    padding: 3rem;
    color: #8b949e;
    font-style: italic;
  }

  .file-list {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 2px;
  }

  .file-entry {
    display: flex;
    align-items: center;
    width: 100%;
    padding: 0.65rem 0.75rem;
    background: #161b22;
    border: 1px solid transparent;
    border-radius: 6px;
    cursor: default;
    font-family: inherit;
    font-size: 0.9rem;
    color: #c9d1d9;
    text-align: left;
    transition: all 0.1s ease;
  }

  .file-entry.is-dir {
    cursor: pointer;
  }

  .file-entry.is-dir:hover {
    background: #1f2937;
    border-color: #30363d;
  }

  .file-entry:disabled:not(.is-dir) {
    opacity: 1;
  }

  .icon {
    margin-right: 0.75rem;
    font-size: 1.1rem;
  }

  .name {
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .is-dir .name {
    color: #58a6ff;
  }

  .size {
    font-size: 0.8rem;
    color: #6e7681;
    margin-left: 1rem;
    min-width: 70px;
    text-align: right;
  }
</style>
