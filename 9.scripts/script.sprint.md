<%*
const quickAddApi = app.plugins.plugins.quickadd.api;
const dateFormat = "YYYY-MM-DD";
const currentTitle = tp.file.title;

const dateRegex = /^\d{4}-\d{2}-\d{2}/;
if (dateRegex.test(currentTitle)) {
    new Notice("Aborting: Date already exists in title.");
    return; 
}

const choice = await quickAddApi.suggester(
    ["This Week (Monday)", "Next Week (Monday)"],
    ["this", "next"]
);

if (!choice) return;

let targetMoment = moment().startOf('isoweek');
if (choice === "next") targetMoment.add(7, 'days');

const targetDate = targetMoment.format(dateFormat);
const newName = `${targetDate} ${currentTitle}`;
await tp.file.rename(newName);
new Notice(`Named to: ${newName}`);
%>