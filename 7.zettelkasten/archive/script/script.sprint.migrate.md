<%-*
/**
 * MIGRATION SCRIPT: SPRINT ARCHIVE
 * ------------------------------------------------
 * 1. Confirms target directory with user.
 * 2. Extracts start date from filename (YYYY-MM-DD).
 * 3. Calculates rating (Legacy vs. #task logic).
 * 4. Sets Sprint properties and completion dates.
 */

// --- 1. CONFIGURATION & CONSTANTS ---
const MIGRATION_PATH = "7.zettelkasten/archive/sprint/2025";
const LEGACY_TASK_CUTOFF = moment("2025-12-15");
const DATE_TIME_FORMAT = "YYYY-MM-DDTHH:mm";
const quickAddApi = app.plugins.plugins.quickadd.api;

// --- 2. MODULAR FUNCTIONS ---

/**
 * Determines task completion rating.
 * If file date < 2025-12-15: Counts all checkboxes.
 * If file date >= 2025-12-15: Requires #task tag.
 */
function calculateRating(content, fileDate) {
    const isPostCutoff = fileDate.isSameOrAfter(LEGACY_TASK_CUTOFF);
    const taskRegex = isPostCutoff 
        ? /^[ \t]*[-*+] \[.\].*#task.*$/gm 
        : /^[ \t]*[-*+] \[.\].*$/gm;
    const completedRegex = isPostCutoff 
        ? /^[ \t]*[-*+] \[[xX-]\].*#task.*$/gm 
        : /^[ \t]*[-*+] \[[xX-]\].*$/gm;

    const allMatches = content.match(taskRegex) || [];
    const completedMatches = content.match(completedRegex) || [];
    const total = allMatches.length;
    const completed = completedMatches.length;

    return total > 0 ? parseFloat((completed / total).toFixed(2)) : 0.00;
}


async function applyProperties(frontmatter, fileDate, rating) {
	const startDateStr = fileDate.format(DATE_TIME_FORMAT);
	const endDateStr = fileDate.clone().add(7, 'days').format(DATE_TIME_FORMAT);

    frontmatter["kind"] = ["sprint"];
    frontmatter["tags"] = ["big7"];
    frontmatter["category"] = ["planning"];
    frontmatter["rating"] = rating;
    frontmatter["start date"] = startDateStr;
    frontmatter["completed date"] = endDateStr;
    frontmatter["status"] = ["✅ done"];
    frontmatter["complete"] = true;
}

const confirm = await quickAddApi.suggester(
    ["Continue Migration", "Cancel"], 
    [true, false], 
    `Run migration on: ${MIGRATION_PATH}?`
);

if (!confirm) {
    new Notice("Migration aborted.");
    return;
}

const folder = app.vault.getAbstractFileByPath(MIGRATION_PATH);
if (!folder || !folder.children) {
    new Notice("Error: Target folder not found.");
    return;
}

const files = folder.children.filter(f => f.extension === "md");
let count = 0;

for (const file of files) {
    // 1. Extract Date from filename prefix (YYYY-MM-DD)
    const dateMatch = file.name.match(/^\d{4}-\d{2}-\d{2}/);
    if (!dateMatch) continue; 
    
    const fileDate = moment(dateMatch[0], "YYYY-MM-DD");
    const content = await app.vault.read(file);
    const rating = calculateRating(content, fileDate);

    await app.fileManager.processFrontMatter(file, (fm) => {
        applyProperties(fm, fileDate, rating);
    });
    
    count++;
}

new Notice(`Migration Success: ${count} files updated.`);
-%>