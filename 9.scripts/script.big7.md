<%*
const quickAddApi = app.plugins.plugins.quickadd.api;

// --- CONSTANTS ---

const SPRINT_DIR = "7.zettelkasten/sprint";

const CATEGORIES = ["Passion", "Health", "Social", "Finances"];

const PRIORITY_MAP = {
    "Highest": "🔺",
    "High": "⏫",
    "Medium": "🔼",
    "Low": "🔽",
    "Lowest": "⏬",
};

const PRIORITIES = Object.keys(PRIORITY_MAP);

// --- SPRINT HELPERS ---

function getSprintOptions() {
    const thisMonday = moment().startOf("isoweek");
    const nextMonday = thisMonday.clone().add(7, "days");

    return [
        {
            label: `This Week (${thisMonday.format("YYYY-MM-DD")})`,
            date: thisMonday,
        },
        {
            label: `Next Week (${nextMonday.format("YYYY-MM-DD")})`,
            date: nextMonday,
        },
    ];
}

function resolveSprintFile(sprintMoment) {
    const fileName = `${sprintMoment.format("YYYY-MM-DD")} Big 7 Check-in.md`;
    const path = `${SPRINT_DIR}/${fileName}`;
    return app.vault.getAbstractFileByPath(path);
}

// --- PROMPTS ---

async function selectSprint() {
    const options = getSprintOptions();
    const labels = options.map(o => o.label);

    const choice = await quickAddApi.suggester(labels, labels);
    if (!choice) return null;

    return options.find(o => o.label === choice);
}

async function promptTaskName() {
    return await quickAddApi.inputPrompt("Task name");
}

async function promptCategory() {
    return await quickAddApi.suggester(CATEGORIES, CATEGORIES);
}

async function promptPriority() {
    return await quickAddApi.suggester(PRIORITIES, PRIORITIES);
}

async function promptDate() {
    return await quickAddApi.inputPrompt("Due date (YYYY-MM-DD)");
}

// --- TASK HELPERS ---

function buildTask(name, priority, date) {
    let task = `- [ ] #task ${name}`;

    if (priority) task += ` ${PRIORITY_MAP[priority]}`;
    if (date) task += ` 📅 ${date}`;

    return task;
}

async function insertTask(file, task, category) {
    const content = await app.vault.read(file);
    const lines = content.split("\n");

    const heading = `#### ${category}`;
    let index = -1;

    for (let i = 0; i < lines.length; i++) {
        if (lines[i].trim() === heading) {
            index = i + 1;
            break;
        }
    }

    if (index === -1) {
        new Notice(`Category not found: ${category}`);
        return;
    }

    while (index < lines.length && lines[index].startsWith("- [")) {
        index++;
    }

    lines.splice(index, 0, task);
    await app.vault.modify(file, lines.join("\n"));
}

// --- MAIN EXECUTION ---

const sprint = await selectSprint();
if (!sprint) return;

const sprintFile = resolveSprintFile(sprint.date);
if (!sprintFile) {
    new Notice(`Sprint file not found for ${sprint.date.format("YYYY-MM-DD")}`);
    return;
}

const name = await promptTaskName();
if (!name) return;

const category = await promptCategory();
if (!category) return;

const priority = await promptPriority();
const date = await promptDate();

const task = buildTask(name, priority, date);
await insertTask(sprintFile, task, category);

new Notice(`Task added to ${sprint.date.format("YYYY-MM-DD")} – ${category}`);
%>
