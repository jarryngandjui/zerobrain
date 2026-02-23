<%*
const quickAddApi = app.plugins.plugins.quickadd.api;
const file = tp.config.target_file;
const now = moment().format("YYYY-MM-DDTHH:mm");

// --- MODULAR FUNCTIONS ---

async function setInProgress(frontmatter) {
    frontmatter["start date"] = now;
    frontmatter["status"] = ["🟨 progress"];
    frontmatter["complete"] = false;
}

async function setCompleted(frontmatter) {
    frontmatter["completed date"] = now;
    frontmatter["status"] = ["✅ done"];
    frontmatter["complete"] = true;
}

async function setCancelled(frontmatter) {
    frontmatter["cancelled date"] = now;
    frontmatter["status"] = ["🛑 blocked"];
    frontmatter["complete"] = false;
}

// --- MAIN EXECUTION ---

const options = ["🟨 progress", "✅ done", "🛑 blocked", "☑︎ productivity"];
const choice = await quickAddApi.suggester(options, options);

if (!choice) return;

await app.fileManager.processFrontMatter(file, (frontmatter) => {
    switch (choice) {
        case "🟨 progress":
            setInProgress(frontmatter);
            break;
        case "✅ done":
            setCompleted(frontmatter);
            break;
        case "🛑 blocked":
            setCancelled(frontmatter);
            break;
    }
});

new Notice(`Status updated to: ${choice}`);
%>