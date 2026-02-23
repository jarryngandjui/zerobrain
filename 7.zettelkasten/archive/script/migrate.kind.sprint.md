<%*
const files = app.vault.getMarkdownFiles();
let updatedCount = 0;

for (const file of files) {
    const cache = app.metadataCache.getFileCache(file);
    const frontmatter = cache?.frontmatter;

    if (frontmatter && frontmatter.kind && frontmatter.kind.includes("big7")) {
        await app.fileManager.processFrontMatter(file, (fm) => {
            if (Array.isArray(fm["kind"])) {
                // Find the index of 'big7' and replace it with 'sprint'
                const index = fm["kind"].indexOf("big7");
                if (index !== -1) {
                    fm["kind"][index] = "sprint";
                }
            } else if (fm["kind"] === "big7") {
                // Fallback for single-string values
                fm["kind"] = ["sprint"];
            }
        });
        updatedCount++;
    }
}

new Notice(`Vault Update Complete: Changed 'big7' to 'sprint' in ${updatedCount} files.`);
%>