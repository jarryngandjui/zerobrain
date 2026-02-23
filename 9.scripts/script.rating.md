<%-*
const file = tp.config.target_file;
const content = await app.vault.read(file);

// Filters specifically for lines containing the "#task" tag
const taskRegex = /^[ \t]*[-*+] \[.\].*#task.*$/gm;
const completedRegex = /^[ \t]*[-*+] \[[xX-]\].*#task.*$/gm;

// Calculates as decimal (e.g., 0.93) rounded to 2 places
const allMatches = content.match(taskRegex) || [];
const completedMatches = content.match(completedRegex) || [];
const total = allMatches.length;
const completed = completedMatches.length;

const rating = total > 0 ? parseFloat((completed / total).toFixed(2)) : 0.00;

await app.fileManager.processFrontMatter(file, (frontmatter) => {
    frontmatter["rating"] = rating;
});

new Notice(`Rating updated: ${rating} (${completed}/${total} #tasks)`);
-%>